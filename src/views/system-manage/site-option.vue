/*
 * @Author: Chenxu 
 * @Date: 2019-07-04 13:59:59 
 * @Last Modified by: chenjie
 * @Last Modified time: 2019-07-05 16:49:52
 */
<template>
  <div class="app-container">
    <!-- 模态窗 -->
    <el-form
      ref="dataForm"
      label-position="left"
      label-width="200px"
      style="width: 600px; margin-left:50px;"
    >
      <el-form-item
        v-for="(item,index) in list"
        :key="item.id"
        :label="item.memo"
        :prop="item.name"
      >
        <el-input v-model="list[index].value" />
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="updateData()">{{ $t('table.confirm') }}</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { get_config, update_config } from '@/api/yunzhijia'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const calendarTypeOptions = [
  { key: 'CN', display_name: 'China' },
  { key: 'US', display_name: 'USA' },
  { key: 'JP', display_name: 'Japan' },
  { key: 'EU', display_name: 'Eurozone' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'Site-option',
  components: { Pagination },
  directives: { waves },
  filters: {
    statusFilter (status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter (type) {
      return calendarTypeKeyValue[type]
    }
  },
  data () {
    return {
      tableKey: 0,
      total: 0,
      listLoading: true,
      calendarTypeOptions,
      list: [],
      rules: {
      }
    }
  },
  created () {
    this.getList()
  },
  methods: {
    getList () {
      this.listLoading = true
      get_config().then(response => {
        this.list = response.result

        this.listLoading = false
      })
    },

    updateData () {
      let tempData = {}
      this.list.forEach(item => {
        tempData[item.name] = item.value
      });
      update_config(tempData).then(() => {
        this.dialogFormVisible = false
        this.$notify({
          title: '成功',
          message: '更新成功',
          type: 'success',
          duration: 2000
        })
      })
    },
    formatJson (filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    }
  }
}
</script>

<style scoped>
</style>
