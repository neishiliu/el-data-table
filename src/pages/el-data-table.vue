<template>
  <nelson-data-table v-bind="tableConfig"></nelson-data-table>
</template>

<script>
import NelsonDataTable from '../components/NelsonDataTable.vue'
import dayjs from 'dayjs'
export default {
  components: {
    NelsonDataTable
  },
  data() {
    return {
      tableConfig: {
        url: '/security/api/v1/example/users',
        showCheck: true,
        tableConfig: {
          'cell-class-name': data => {
            const {row, columnIndex} = data
            if (columnIndex === 6) {
              return row.status === '上架' ? 'green' : ''
            }
          }
        },
        itemEdit: row => row.status,
        columns: [
          {
            prop: 'name',
            label: '组件名称'
          },
          {
            prop: 'type',
            label: '分类'
          },
          {
            prop: 'version',
            label: '版本',
            width: '100px'
          },
          {
            prop: 'lang',
            label: '开发语言'
          },
          {
            prop: 'date',
            label: '最后更新时间',
            formatter: (row, column, cellValue, index) => {
              return dayjs(cellValue).format('YYYY-MM-DD')
            }
          },
          {
            prop: 'status',
            label: '状态',
            width: '100px'
          }
        ],
        searchForm: [
          {
            $type: 'input',
            $id: 'name',
            label: '用户名',
            $el: {
              placeholder: '请输入'
            }
          }
        ],
        form: [
          {
            $type: 'input',
            $id: 'name',
            label: '用户名',
            $el: {
              placeholder: '请输入'
            },
            rules: [
              {
                required: true,
                message: '请输入用户名',
                trigger: 'blur'
              }
            ]
          }
        ]
      }
    }
  },
  methods: {}
}
</script>
<style lang="stylus">
.green {
  color: #06cc06;
}
</style>
