/*
 * @Author: Chenxu 
 * @Date: 2019-07-04 13:59:59 
 * @Last Modified by: Chenxu
 * @Last Modified time: 2019-07-26 09:50:21
 */
<template>
  <div class="app-container">
    <!-- 模态窗 -->
    <el-form ref="dataForm" label-position="left" style=" margin-left:50px;">
      <el-form-item v-for="(item,index) in list" :key="item.id">
        <el-input
          @keydown.native="$set(list[index], 'show', true)"
          v-model="item.type_name"
          style="width: 300px;"
        />
        <el-button
          v-if="item.show"
          type="primary"
          @click="updateData({type_name:item.type_name,id:item.id})"
        >修改</el-button>
        <el-button type="danger" @click="delData(item.id)">删除</el-button>
      </el-form-item>

      <el-form-item>
        <el-input v-model="temp_name" style="width: 300px;" />
        <el-button type="success" @click="addData">添加</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { need_type, edit_need_type, del_need_type } from '@/api/yunzhijia'
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
  name: 'demand',
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
      temp_name: '',
      rules: {
      }
    }
  },
  created () {
    this.getList()
  },
  methods: {
    /* 删除 */
    delData (id) {
      del_need_type({ id: id }).then(res => {
        this.getList()
        this.$notify({
          title: '成功',
          message: '删除成功',
          type: 'success',
          duration: 2000
        })
      })
    },
    getList () {
      this.listLoading = true
      need_type().then(response => {
        this.list = response.result
        this.listLoading = false
      })
    },
    addData () {
      let params = {
        type_name: this.temp_name
      }
      edit_need_type(params).then(() => {
        this.dialogFormVisible = false
        this.temp_name = ''
        this.getList()
        this.$notify({
          title: '成功',
          message: '添加成功',
          type: 'success',
          duration: 2000
        })
      })
    },
    updateData (obj) {
      edit_need_type(obj).then(() => {
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
