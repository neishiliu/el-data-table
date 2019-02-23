<template>
  <div class="el-data-table">
    <!--搜索框-->
    <section>
      <el-form-renderer
        v-if="searchForm.length > 0 || !!$slots.search"
        inline
        :content="searchForm"
        ref="searchForm"
      >
        <!--@slot 额外的搜索内容, 当searchForm不满足需求时可以使用-->
        <slot name="search"></slot>
        <el-form-item>
          <!--https://github.com/ElemeFE/element/pull/5920-->
          <el-button
            native-type="submit"
            type="primary"
            @click="search"
            size="small"
            >查询</el-button
          >
          <el-button @click="resetSearch" size="small">重置</el-button>
        </el-form-item>
      </el-form-renderer>
    </section>

    <!--主操作按钮-->
    <section>
      <el-button type="primary" size="small" v-if="showAdd">新增</el-button>
      <el-button size="small" type="danger" v-if="showCheck">删除</el-button>
    </section>

    <!--表格-->
    <section>
      <el-table
        ref="multipleTable"
        :data="tableData"
        tooltip-effect="dark"
        style="width: 100%;"
        @selection-change="handleSelectionChange"
        v-bind="tableConfig"
        :loading="loading"
      >
        <el-table-column v-if="showCheck" type="selection" width="55">
        </el-table-column>
        <el-table-column
          v-for="(col, index) in columns"
          :key="index"
          v-bind="col"
        >
        </el-table-column>
        <el-table-column label="操作" v-if="showActions">
          <template slot-scope="data">
            <el-button
              size="small"
              v-if="itemView(data.row)"
              @click="onDefaultShow(data.row)"
            >
              查看
            </el-button>
            <el-button
              size="small"
              v-if="itemEdit(data.row)"
              @click="onDefaultEdit(data.row)"
            >
              編輯
            </el-button>
            <el-button
              size="small"
              v-if="itemDelete(data.row)"
              @click="onDefaultDelete(data.row)"
            >
              刪除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </section>

    <!--分页器-->
    <section class="pagination">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        v-bind="paginationConfig"
        :current-page="currentPage"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </section>
  </div>
</template>

<script>
export default {
  name: 'nelson-data-table',
  props: {
    // 數據源url
    url: {
      type: String
    },
    // 是否显示新增按钮
    showAdd: {
      type: Boolean,
      default: true
    },
    // 是否显示多选框（同时控制主删除按钮的显示）
    showCheck: {
      type: Boolean,
      default: false
    },
    // 是否顯示列表內操作按鈕
    showActions: {
      type: Boolean,
      default: true
    },
    // 列表項是否可查看
    itemView: {
      type: Function,
      default: () => true
    },
    // 列表項是否可編輯
    itemEdit: {
      type: Function,
      default: () => true
    },
    // 列表項是否可刪除
    itemDelete: {
      type: Function,
      default: () => true
    },
    tableConfig: {
      type: Object
    },
    // 表格数据列配置
    columns: {
      type: Array,
      default: []
    },
    // 分頁器配置
    paginationConfig: {
      type: Object,
      default: function() {
        return {
          'page-sizes': [10, 20, 50, 100],
          'page-size': 10
        }
      }
    },
    // 搜索框配置
    searchForm: {
      type: Array
    }
  },
  data: function() {
    return {
      currentPage: 1,
      count:
        this.paginationConfig['page-size'] ||
        this.paginationConfig['page-sizes'][0],
      total: 100,
      loading: false,
      formInline: {
        user: '',
        region: ''
      },
      tableData: [],
      multipleSelection: []
    }
  },
  created() {
    if (this.url) {
      console.log(this.url)
      this.load()
    }
  },
  methods: {
    load() {
      const query = this.$refs.searchForm
        ? this.$refs.searchForm.getFormValue()
        : {}
      let params = '?'
      for (let key in query) {
        console.log(key)
        params += `&${key}=${query[key].trim()}`
      }
      params += `&page=${this.currentPage}&count=${this.count}`

      this.loading = true
      this.$axios.$get(this.url + params).then(
        res => {
          this.loading = false
          this.tableData = res.datas
          this.total = res.total
        },
        err => {
          this.loading = false
          console.log(err)
        }
      )
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    handleSizeChange(val) {
      this.count = val
      this.currentPage = 1
      this.load()
      console.log(`每页 ${val} 条`)
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.load()
      console.log(`当前页: ${val}`)
    },
    search() {
      this.page = 1
      this.load()
    },
    resetSearch() {
      this.$refs.searchForm.resetFields()
      this.currentPage = 1
    },
    onDefaultShow(row) {
      console.log(row)
    },
    onDefaultEdit(row) {
      console.log(row)
    },
    onDefaultDelete(row) {
      console.log(row)
    }
  }
}
</script>
<style lang="stylus">
.pagination {
  text-align: right;
  margin-top: 10px;
}
</style>
