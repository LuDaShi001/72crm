<template>
  <div>
    <c-r-m-list-head title="线索管理"
                     placeholder="请输入线索名称"
                     @on-handle="listHeadHandle"
                     @on-search="crmSearch"
                     main-title="新建线索"
                     @on-export="exportInfos"
                     :crm-type="crmType">
    </c-r-m-list-head>
    <div v-empty="!crm.leads.index"
         xs-empty-icon="nopermission"
         xs-empty-text="暂无权限"
         class="crm-container">
      <c-r-m-table-head ref="crmTableHead"
                        :crm-type="crmType"
                        @filter="handleFilter"
                        :scene_id="scene_id"
                        :scene_name="scene_name"
                        @handle="handleHandle"
                        @scene="handleScene"></c-r-m-table-head>
      <el-table class="n-table--border"
                id="crm-table"
                v-loading="loading"
                :data="list"
                :height="tableHeight"
                stripe
                border
                highlight-current-row
                style="width: 100%"
                :cell-style="cellStyle"
                @row-click="handleRowClick"
                @header-dragend="handleHeaderDragend"
                @selection-change="handleSelectionChange">
        <el-table-column show-overflow-tooltip
                         type="selection"
                         align="center"
                         width="55">
        </el-table-column>
        <el-table-column v-for="(item, index) in fieldList"
                         :key="index"
                         show-overflow-tooltip
                         :fixed="index==0"
                         :prop="item.prop"
                         :label="item.label"
                         :width="item.width"
                         :formatter="fieldFormatter">
          <template slot="header"
                    slot-scope="scope">
            <div class="table-head-name">{{scope.column.label}}</div>
          </template>
        </el-table-column>
        <el-table-column>
        </el-table-column>
        <el-table-column fixed="right"
                         width="36">
          <template slot="header"
                    slot-scope="slot">
            <img src="@/assets/img/t_set.png"
                 @click="handleTableSet"
                 class="table-set" />
          </template>
        </el-table-column>
      </el-table>
      <div class="p-contianer">
        <el-pagination class="p-bar"
                       @size-change="handleSizeChange"
                       @current-change="handleCurrentChange"
                       :current-page="currentPage"
                       :page-sizes="pageSizes"
                       :page-size.sync="pageSize"
                       layout="total, sizes, prev, pager, next, jumper"
                       :total="total">
        </el-pagination>
      </div>
    </div>
    <clue-detail v-if="showDview"
                 :id="rowID"
                 @handle="handleHandle"
                 @hide-view="showDview=false"
                 class="d-view"></clue-detail>
    <fields-set :crmType="crmType"
                @set-success="setSave"
                :dialogVisible.sync="showFieldSet"></fields-set>
  </div>
</template>

<script>
import { crmLeadsExcelExport } from '@/api/customermanagement/clue'
import ClueDetail from './ClueDetail'
import table from '../mixins/table'

export default {
  /** 客户管理 的 线索列表 */
  name: 'clueIndex',
  components: {
    ClueDetail
  },
  mixins: [table],
  data() {
    return {
      crmType: 'leads'
    }
  },
  computed: {},
  mounted() {},
  methods: {
    // 导出操作
    exportInfos() {
      var params = {
        search: this.search
      }
      if (this.scene_id) {
        params.scene_id = this.scene_id
      }
      for (var key in this.filterObj) {
        params[key] = this.filterObj[key]
      }
      crmLeadsExcelExport(params)
        .then(res => {
          var blob = new Blob([res.data], {
            type: 'application/vnd.ms-excel;charset=utf-8'
          })
          var downloadElement = document.createElement('a')
          var href = window.URL.createObjectURL(blob) //创建下载的链接
          downloadElement.href = href
          downloadElement.download =
            decodeURI(
              res.headers['content-disposition'].split('filename=')[1]
            ) || '' //下载后文件名
          document.body.appendChild(downloadElement)
          downloadElement.click() //点击下载
          document.body.removeChild(downloadElement) //下载完成移除元素
          window.URL.revokeObjectURL(href) //释放掉blob对象
        })
        .catch(() => {})
    },
    /** 通过回调控制style */
    cellStyle({ row, column, rowIndex, columnIndex }) {
      if (column.property === 'name') {
        return { color: '#3E84E9', cursor: 'pointer' }
      } else {
        return ''
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../styles/table.scss';
</style>
