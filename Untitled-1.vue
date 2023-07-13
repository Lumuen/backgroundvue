<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
 
          <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="信用评价">
                <a-input placeholder="请输入最小值" class="query-group-cust" v-model="queryParam.creditEvaluation_begin"></a-input>
                <span class="query-group-split-cust"></span>
                <a-input placeholder="请输入最大值" class="query-group-cust" v-model="queryParam.creditEvaluation_end"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="经营范围">
                <a-input placeholder="请输入经营范围" v-model="queryParam.businessScope"></a-input>
              </a-form-item>
            </a-col>
          <template v-if="toggleSearchStatus">
      
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('企业库')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        class="j-table-force-nowrap"
        :scroll="{x:true}"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type:'radio'}"
        :customRow="clickThenSelect"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text,record">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" :preview="record.id" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定为这个等级吗?" @confirm="() => handleLevel(record.id)">
                  <a>优秀</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定为这个等级吗?" @confirm="() => handleLevel(record.id)">
                  <a>良好</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定为这个等级吗?" @confirm="() => handleLevel(record.id)">
                  <a>中等</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定为这个等级吗?" @confirm="() => handleLevel(record.id)">
                  <a>较差</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="联系人" key="1" >
        <EnterpriseChildContactsList :mainId="enterpriseChildContactsMainId" />
      </a-tab-pane>
    </a-tabs>

    <enterpriseRoster-modal ref="modalForm" @ok="modalFormOk"></enterpriseRoster-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import EnterpriseRosterModal from './modules/EnterpriseRosterModal'
  import { getAction } from '@/api/manage'
  import EnterpriseChildContactsList from './EnterpriseChildContactsList'
  import {initDictOptions,filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import '@/assets/less/TableExpand.less'

  export default {
    name: "EnterpriseRosterList",
    mixins:[JeecgListMixin],
    components: {
      EnterpriseChildContactsList,
      EnterpriseRosterModal
    },
    data () {
      return {
        description: '企业库管理页面',
        // 表头
        columns: [
          {
            title:'企业名称',
            align:"center",
            sorter: true,
            dataIndex: 'enterpriseName'
          },
          {
            title:'企业法人',
            align:"center",
            sorter: true,
            dataIndex: 'legalPerson'
          },
          {
            title:'企业地址',
            align:"center",
            sorter: true,
            dataIndex: 'address'
          },
          {
            title:'企业类型',
            align:"center",
            sorter: true,
            dataIndex: 'type_dictText',
          },
          {
            title:'企业属性',
            align:"center",
            sorter: true,
            dataIndex: 'attribute_dictText',
          },
          {
            title:'合作关系',
            align:"center",
            sorter: true,
            dataIndex: 'partnership_dictText',
          },
         
          {
            title:'经营范围',
            align:"center",
            sorter: true,
            dataIndex: 'businessScope'
          },
          {
            title:'资质类型',
            align:"center",
            sorter: true,
            dataIndex: 'qualificationType_dictText',
          },
          {
            title:'资质等级',
            align:"center",
            sorter: true,
            dataIndex: 'qualificationRegistration_dictText',
          },
          {
            title:'信用等级',
            align:"center",
            sorter: true,
            dataIndex: 'priority_dictText',
          }, 
          
         
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: "/jeecg-demo/enterpriseRoster/enterpriseRoster/list",
          delete: "/jeecg-demo/enterpriseRoster/enterpriseRoster/delete",
          deleteBatch: "/jeecg-demo/enterpriseRoster/enterpriseRoster/deleteBatch",
          exportXlsUrl: "/jeecg-demo/enterpriseRoster/enterpriseRoster/exportXls",
          importExcelUrl: "jeecg-demo/enterpriseRoster/enterpriseRoster/importExcel",
          edit: "/jeecg-demo/enterpriseRoster/enterpriseRoster/edit",
        },
        dictOptions:{
         type:[],
         attribute:[],
         partnership:[],
         creditEvaluation:[],
         qualificationType:[],
         qualificationRegistration:[],
         state:[],
        },
        /* 分页参数 */
        ipagination:{
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '50'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        selectedMainId:'',
        superFieldList:[],
        enterpriseChildContactsMainId: '',
      }
    },
    created() {
      this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      initDictConfig(){
      },
      clickThenSelect(record) {
        return {
          on: {
            click: () => {
              this.onSelectChange(record.id.split(","), [record]);
            }
          }
        }
      },
      onClearSelected() {
        this.selectedRowKeys = [];
        this.selectionRows = [];
        this.selectedMainId=''
      },
      onSelectChange(selectedRowKeys, selectionRows) {
        this.selectedMainId=selectedRowKeys[0]
        this.selectedRowKeys = selectedRowKeys;
        this.selectionRows = selectionRows;
        this.enterpriseChildContactsMainId = selectionRows[0]['id']
      },
      loadData(arg) {
        if(!this.url.list){
          this.$message.error("请设置url.list属性!")
          return
        }
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        this.onClearSelected()
        var params = this.getQueryParams();//查询条件
        this.loading = true;
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.records;
            this.ipagination.total = res.result.total;
          }
          if(res.code===510){
            this.$message.warning(res.message)
          }
          this.loading = false;
        })
        
      },
      changeStatus(){
        
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'enterpriseName',text:'企业名称',dictCode:''})
        fieldList.push({type:'string',value:'legalPerson',text:'企业法人',dictCode:''})
        fieldList.push({type:'string',value:'address',text:'企业地址',dictCode:''})
        fieldList.push({type:'int',value:'type',text:'企业类型',dictCode:'enterprise_type'})
        fieldList.push({type:'int',value:'attribute',text:'企业属性',dictCode:'enterprise_attributes'})
        fieldList.push({type:'int',value:'partnership',text:'合作关系',dictCode:'partnership'})
        fieldList.push({type:'int',value:'creditEvaluation',text:'信用评价',dictCode:'credit_evaluation'})
        fieldList.push({type:'string',value:'businessScope',text:'经营范围',dictCode:''})
        fieldList.push({type:'int',value:'qualificationType',text:'资质类型',dictCode:'qualification_type'})
        fieldList.push({type:'string',value:'qualificationRegistration',text:'资质等级',dictCode:''})
        fieldList.push({type:'int',value:'state',text:'状态',dictCode:'company_state'})
        fieldList.push({type:'string',value:'remark',text:'备注',dictCode:''})
        fieldList.push({type:'string',value:'annex',text:'附件',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>