/*
 * @Author: Chenxu 
 * @Date: 2019-07-04 13:59:59 
 * @Last Modified by: Chenxu
 * @Last Modified time: 2019-07-30 17:03:13
 */
<template>
  <div class="app-container">
    <div class="filter-container">
      <!-- 条件查询 -->
      <el-input
        v-model="listQuery.name"
        placeholder="请输入查询条件"
        style="width: 300px;"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />

      <el-select v-model="listQuery.type_id" class="filter-item" placeholder="选择服务类型">
        <el-option v-for="item in types" :key="item.id" :label="item.type_name" :value="item.id" />
      </el-select>
      <el-select v-model="listQuery.status" class="filter-item" placeholder="选择审核状态">
        <el-option
          v-for="item in statuss"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        />
      </el-select>

      <!-- 操作按钮 -->
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >{{ $t('table.search') }}</el-button>
      <el-button
        class="filter-item"
        style="margin-left: 10px;"
        type="primary"
        icon="el-icon-edit"
        @click="handleCreate"
      >{{ $t('table.add') }}</el-button>
      <el-button
        v-waves
        :loading="downloadLoading"
        class="filter-item"
        type="primary"
        icon="el-icon-download"
        @click="handleDownload"
      >{{ $t('table.export') }}</el-button>
    </div>
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      size="mini"
      fit
      highlight-current-row
      style="width: 100%;"
      @sort-change="sortChange"
    >
      <el-table-column
        :label="$t('table.id')"
        sortable="custom"
        align="center"
        width="80"
        type="index"
      ></el-table-column>
      <el-table-column label="团队名称" min-width="100px">
        <template slot-scope="{row}">
          <span>{{ row.team_name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="所属类型" min-width="150px" align="center">
        <template slot-scope="scope">
          <span>{{scope.row.type_id.toString()}}</span>
        </template>
      </el-table-column>

      <el-table-column label="团队简介" min-width="150px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.description }}</span>
        </template>
      </el-table-column>
      <el-table-column label="团队长姓名和电话" min-width="100px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.team_leader_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="实践中心相册" min-width="100px" align="center">
        <template slot-scope="scope">
          <img
            style="width:50px;height:50px;"
            v-for="(item,index) in scope.row.album"
            :key="index"
            :src="item.preview_image"
            alt
          />
        </template>
      </el-table-column>
      <el-table-column label="创建时间" width="150px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.createtime }}</span>
        </template>
      </el-table-column>
      <el-table-column label="审核状态" width="100px" align="center">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status == 0" type="warning">未审核</el-tag>
          <el-tag v-if="scope.row.status == 1" type="success">已通过</el-tag>
          <el-tag v-if="scope.row.status == 2" type="danger">已拒绝</el-tag>
        </template>
      </el-table-column>

      <!-- 添加 -->

      <!-- 操作 -->
      <el-table-column label="操作" align="center" width="250" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button
            v-if="row.status == 1"
            type="primary"
            size="mini"
            @click="viewNumbers(row)"
          >查看团队成员</el-button>
          <el-button type="primary" size="mini" @click="handleUpdate(row)">编辑</el-button>

          <el-button v-if="!row.status" type="success" size="mini" @click="agree(row)">通过</el-button>
          <el-button v-if="!row.status" type="danger" size="mini" @click="reject(row)">拒绝</el-button>
          <el-button type="danger" size="mini" @click="del_team(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.row"
      @pagination="getList"
    />

    <!-- 添加 -->
    <el-dialog title="添加行政志愿者团队" :visible.sync="dialogFormVisible1">
      <el-form
        ref="dataForm"
        :model="team"
        label-position="right"
        label-width="120px"
        style="width: 400px; margin-left:50px;"
      >
        <el-form-item label="团队名称" prop="title">
          <el-input v-model="team.team_name" />
        </el-form-item>
        <el-form-item label="团徽" prop="title">
          <el-upload
            class="avatar-uploader"
            action="http://up.qiniup.com/"
            :show-file-list="false"
            :data="upData1"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="images" :src="images" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon" />
          </el-upload>
        </el-form-item>
        <el-form-item label="团队简介描述" prop="title">
          <el-input type="textarea" v-model="team.description" />
        </el-form-item>

        <el-form-item label="团队类别" prop="type">
          <el-select v-model="team.type_id" multiple class="filter-item" placeholder="请选择">
            <el-option
              v-for="item in types"
              :key="item.id"
              :label="item.type_name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible1 = false">关闭</el-button>
        <el-button type="primary" @click="add">添加</el-button>
      </div>
    </el-dialog>

    <!-- 修改 -->
    <team-edit :id="rowId" v-show="editShow" @close="editClose" :show="editShow" />

    <!-- 模态窗 -->
    <el-dialog title="团队成员列表" :visible.sync="dialogFormVisible">
      <el-upload
        class="upload-demo"
        :show-file-list="false"
        action="http://yzj.yb.qqiang.net/admin/volunteers/upload_excel"
        :on-preview="handlePreview"
        :on-success="success"
        :on-error="error"
        :limit="1"
        :data="upData"
        :file-list="fileList"
      >
        <el-button size="small" type="primary">导入数据</el-button>
      </el-upload>
      <el-table :data="numbers" style="width: 100%;" max-height="350" v-loading="numberLoading">
        <el-table-column label="序号" type="index" width="150" align="center"></el-table-column>
        <el-table-column prop="user_info" label="用户名：用户手机号" width="auto" align="left"></el-table-column>
        <!-- caozuo  -->
        <el-table-column
          :label="$t('table.actions')"
          align="center"
          width="150"
          class-name="small-padding fixed-width"
        >
          <template slot-scope="{row}">
            <!-- <el-button type="primary" size="mini" @click="handleUpdate(row)">编辑</el-button> -->
            <el-button type="danger" size="mini" @click="delUser(row)">移除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">关闭</el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">{{ $t('table.confirm') }}</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { volunteers_team, remove_xzmember, del_team, userIndex, team_member, token, need_type, cat, volunteers_team_check, addVal, add_xzteam, updateVal, delVal } from '@/api/yunzhijia'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination
import axios from "@/utils/request";
import { getToken, getAdminId } from '@/utils/auth'
import teamEdit from "./team-edit.vue";

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
  name: 'admin-postulant-team',
  components: { Pagination, teamEdit },
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
      team: {
        team_name: '',
        description: '',
        type_id: ''
      },
      rowId: '',
      editShow: false,
      types: [],
      images: '',
      upData1: {
        token: ''
      },

      statuss: [{
        value: '',
        label: '全部'
      }, {
        value: 0,
        label: '审核中'
      },
      {
        value: 1,
        label: '招募中'
      }, {
        value: 2,
        label: '进行中'
      }, {
        value: 3,
        label: '已完成'
      }, {
        value: 4,
        label: '撤销'
      }],
      upData: {
        admin_id: getAdminId()
      },
      cats: [],
      numbers: [],
      dialogFormVisible: false,
      dialogFormVisible1: false,
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      numberLoading: true,
      team_id: '',
      listQuery: {
        p: 1,
        row: 20,
        type: 2,
        status: '',
        type_id: ''
      },
      calendarTypeOptions,
      statusOptions: ['published', 'draft', 'deleted'],

      dialogPvVisible: false,
      pvData: [],
      downloadLoading: false,
      fileList: []
    }
  },
  created () {
    this.getList()
    this.getToken()
    this.getNeedType()
    this.getmemberList()
    this.getToken1()
  },
  watch: {
    'listQuery': {
      handler (val) {
        this.getList()
      },
      deep: true
    }
  },
  methods: {
    del_team (row) {
      del_team({ id: row.id }).then(res => {
        this.$message.success(res.msg)
        this.getList()
      })
    },
    editClose () {
      this.editShow = false
      this.getList()
    },
    handleUpdate (row) {
      this.editShow = true
      /* 获取数据 */
      this.rowId = row.id
    },
    beforeAvatarUpload (file) {
      const isLt2M = file.size / 1024 / 1024 < 2
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isLt2M
    },
    handleAvatarSuccess (res, file) {
      this.images = URL.createObjectURL(file.raw)
      this.team.avatar = res.key
    },
    /* 获取 七牛云token */
    getToken1 () {
      token().then(res => {
        this.upData1.token = res.result
      })
    },
    delUser (row) {
      remove_xzmember({ user_id: row.user_id, team_id: this.team_id }).then(res => {
        this.getNumber(this.team_id)
        this.$notify({
          title: '成功',
          message: '删除成功',
          type: 'success',
          duration: 2000
        })
      })
    },
    /* 添加支援者 */
    add () {
      this.team.type_id = this.team.type_id.toString()
      add_xzteam(this.team).then(res => {
        this.getList()
        this.$message.success(res.msg)
        this.dialogFormVisible1 = false
        this.team = {
          team_name: '',
          description: '',
          type_id: ''
        }
      })
    },
    getmemberList () {
      this.listLoading = true
      let parmas = {
        p: 1,
        row: 20,
        type: 2,
        name: undefined
      }
      userIndex(parmas).then(response => {
        this.userList = response.result.list
        this.total = response.result.count
        this.listLoading = false
      }).catch(err => {
        this.listLoading = false
      })
    },
    getNeedType () {
      need_type().then(res => {
        this.types = res.result
        this.types.unshift({
          type_name: '全部',
          value: ''
        })
      })
    },
    handleCreate () {
      this.dialogFormVisible1 = true
    },
    success (res, file) {
      this.$message(res.msg)
      this.dialogFormVisible = false
    },
    error (res, file) {
      this.$message(res.msg)
      this.dialogFormVisible = false
    },
    /* 获取 七牛云token */
    getToken () {
      token().then(res => {
        this.upData.token = res.result
      })
    },
    handlePreview (file) {
      console.log(file);
    },
    getNumber (id) {
      team_member({ team_id: id, p: 1, row: 99999 }).then(response => {
        this.numbers = response.result.list
        this.numberLoading = false
      }).catch(err => {
        this.numberLoading = false

      })
    },
    agree (row) {
      volunteers_team_check({ team_id: row.id, status: 1 }).then(res => {
        let tempArr = this.list.filter(item => {
          return item.id == row.id
        })
        tempArr[0].status = 1
      })
    },
    reject (row) {
      volunteers_team_check({ team_id: row.id, status: 2 }).then(res => {
        let tempArr = this.list.filter(item => {
          return item.id == row.id
        })
        tempArr[0].status = 2
      })
    },
    viewNumbers (e) {
      this.numbers = []

      this.numberLoading = true
      this.dialogFormVisible = true
      this.getNumber(e.id)
      this.team_id = e.id
    },
    getList () {
      this.listLoading = true
      volunteers_team(this.listQuery).then(response => {
        this.list = response.result.list
        this.total = response.result.count
        this.listLoading = false
      }).catch(err => {
        this.listLoading = false
        this.list = []
      })
    },
    handleFilter () {
      this.listQuery.page = 1
      this.getList()
    },

    sortChange (data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID (order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },

    handleFetchPv (pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    handleDownload () {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['timestamp', 'title', 'type', 'importance', 'status']
        const filterVal = ['timestamp', 'title', 'type', 'importance', 'status']
        const data = this.formatJson(filterVal, this.list)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list'
        })
        this.downloadLoading = false
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

<style scoped lang="less">
.fixed-width .el-button--mini {
  width: auto;
}

/deep/.avatar-uploader .el-upload {
  border: 1px dashed #000000;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
/deep/.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
/deep/.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
/deep/.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
