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
        dataPath: 'data.datas',
        totalPath: 'data.total',
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
            label: '组件名称',
            $el: {
              placeholder: '请输入'
            }
          },
          {
            $type: 'select',
            $id: 'type',
            label: '分类',
            $options: [
              '前端组件',
              '测试子组件',
              '分布式工具',
              '应用服务',
              '数据存储'
            ].map(f => ({label: f, value: f})),
            $el: {
              placeholder: '请选择'
            }
          },
          {
            $type: 'select',
            $id: 'status',
            label: '状态',
            $options: ['上架', '下架'].map(f => ({label: f, value: f})),
            $el: {
              placeholder: '请选择'
            }
          }
        ],
        form: [
          {
            $type: 'input',
            $id: 'name',
            label: '组件名称',
            $el: {
              placeholder: '请输入'
            },
            rules: [
              {
                required: true,
                message: '请输入组件名称',
                trigger: 'blur'
              }
            ]
          },
          {
            $type: 'select',
            $id: 'type',
            label: '分类',
            $options: [
              '前端组件',
              '测试子组件',
              '分布式工具',
              '应用服务',
              '数据存储'
            ].map(f => ({label: f, value: f})),
            $el: {
              placeholder: '请选择'
            },
            rules: [
              {
                required: true,
                message: '请选择分类',
                trigger: 'blur'
              }
            ]
          },
          {
            $type: 'input',
            $id: 'version',
            label: '版本',
            $el: {
              placeholder: '请输入'
            },
            rules: [
              {
                required: true,
                message: '请输入版本号',
                trigger: 'blur'
              }
            ]
          },
          {
            $type: 'select',
            $id: 'lang',
            label: '开发语言',
            $options: ['javascript', 'nodejs', 'java', 'PHP'].map(f => ({
              label: f,
              value: f
            })),
            $el: {
              placeholder: '请选择'
            },
            rules: [
              {
                required: true,
                message: '请选择开发语言',
                trigger: 'blur'
              }
            ]
          },
          {
            $type: 'select',
            $id: 'status',
            label: '状态',
            $options: ['上架', '下架'].map(f => ({label: f, value: f})),
            $el: {
              placeholder: '请选择'
            },
            rules: [
              {
                required: true,
                message: '请选择状态',
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
