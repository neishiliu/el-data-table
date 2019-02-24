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
      <el-button type="primary" size="small" v-if="showAdd" @click="edit()"
        >新增</el-button
      >
      <el-button size="small" type="danger" v-if="showCheck" @click="mainDelete"
        >删除</el-button
      >
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

    <!--表单-->
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogFormVisible"
      @close="cancel"
    >
      <el-form-renderer
        :content="form"
        ref="dialogForm"
        v-bind="formAttrs"
        :disabled="isView"
      >
        <!--@slot 额外的弹窗表单内容, 当form不满足需求时可以使用 -->
        <slot name="form"></slot>
      </el-form-renderer>

      <div slot="footer" v-show="!isView">
        <el-button @click="cancel" size="small">取 消</el-button>
        <el-button
          type="primary"
          @click="confirm"
          :loading="confirmLoading"
          size="small"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
import _get from 'lodash.get'
export default {
  name: 'nelson-data-table',
  props: {
    // 數據源url
    url: {
      type: String
    },
    dataPath: {
      type: String,
      default: 'data.datas'
    },
    totalPath: {
      type: String,
      default: 'data.total'
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
      default() {
        return {
          'page-sizes': [10, 20, 50, 100],
          'page-size': 10
        }
      }
    },
    // 搜索框配置
    searchForm: {
      type: Array,
      default() {
        return []
      }
    },
    /**
     * 弹窗表单
     */
    form: {
      type: Array,
      default() {
        return []
      }
    },
    // 弹窗表单设置
    formAttrs: {
      type: Object,
      default() {
        return {}
      }
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
      tableData: [],
      multipleSelection: [],
      dialogFormVisible: false,
      dialogTitle: '新增',
      confirmLoading: false,
      isView: false,
      editRow: {}
    }
  },
  created() {
    if (this.url) {
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
        if (query[key]) {
          params += `&${key}=${query[key].trim()}`
        }
      }
      params += `&page=${this.currentPage}&count=${this.count}`

      this.loading = true
      this.$axios.$get(this.url + params).then(
        res => {
          this.loading = false
          this.tableData = _get(res, this.dataPath)
          this.total = _get(res, this.totalPath)
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
      this.load()
    },
    mainDelete() {
      if (this.multipleSelection.length <= 0) {
        this.$message({
          message: '请先选择需要删除的数据！',
          type: 'warning'
        })
      } else {
        this.$confirm('确认删除？')
          .then(res => {
            const idsArr = this.multipleSelection.map(item => {
              return item.id
            })
            this.$axios.$delete(this.url + '/' + idsArr.toString()).then(
              res => {
                this.$message({
                  message: `成功删除id为‘${idsArr.toString()}’的数据`,
                  type: 'success'
                })
                this.load()
              },
              err => {
                this.$message({
                  message: `删除失败`,
                  type: 'error'
                })
              }
            )
          })
          .catch(_ => {})
      }
    },
    onDefaultShow(row) {
      this.isView = true
      this.dialogTitle = '查看'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs.dialogForm.updateForm(row)
      })
    },
    onDefaultEdit(row) {
      this.edit(row)
    },
    onDefaultDelete(row) {
      this.$confirm('确认删除？')
        .then(res => {
          this.$axios.$delete(this.url + '/' + row.id).then(
            res => {
              this.$message({
                message: `成功删除id为‘${idsArr.toString()}’的数据`,
                type: 'success'
              })
              this.load()
            },
            err => {
              this.$message({
                message: `删除失败`,
                type: 'error'
              })
            }
          )
        })
        .catch(_ => {})
    },
    edit(row = null) {
      // row存在即为编辑，不存在则为新增
      if (row) {
        this.isView = false
        this.dialogTitle = '编辑'
        this.dialogFormVisible = true
        this.editRow = row
        this.$nextTick(() => {
          this.$refs.dialogForm.updateForm(row)
        })
      } else {
        this.isView = false
        this.dialogTitle = '新增'
        this.dialogFormVisible = true
      }
    },
    cancel() {
      this.editRow = {}
      this.$refs.dialogForm.resetFields()
      this.dialogFormVisible = false
    },
    confirm() {
      this.$refs.dialogForm.validate(valid => {
        if (!valid) {
          this.$message({
            message: '输入不合法，请检查表单',
            type: 'warning'
          })
          return false
        } else {
          const val = this.$refs.dialogForm.getFormValue()
          const id = this.editRow.id ? this.editRow.id : ''
          // 有id则为编辑用put，没有则为新增用post
          if (id) {
            this.confirmLoading = true
            this.$axios.$put(this.url + '/' + id, val).then(
              res => {
                this.$message({
                  message: '提交成功',
                  type: 'success'
                })
                this.confirmLoading = false
                this.cancel()
                this.load()
              },
              err => {
                this.confirmLoading = false
                this.$message({
                  message: '提交失败',
                  type: 'error'
                })
              }
            )
          } else {
            this.confirmLoading = true
            this.$axios.$post(this.url, val).then(
              res => {
                this.$message({
                  message: '提交成功',
                  type: 'success'
                })
                this.confirmLoading = false
                this.cancel()
                this.load()
              },
              err => {
                this.confirmLoading = false
                this.$message({
                  message: '提交失败',
                  type: 'error'
                })
              }
            )
          }
        }
      })
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
