/*
 * @Author: Chenxu
 * @Date: 2019-07-04 13:59:59
 * @Last Modified by: Chenxu
 * @Last Modified time: 2019-08-05 10:40:18
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
      <el-select v-model="listQuery.is_top" class="filter-item" placeholder="置顶状态">
        <el-option v-for="item in tops" :key="item.value" :label="item.label" :value="item.value" />
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
      fit
      highlight-current-row
      style="width: 100%;"
      @sort-change="sortChange"
    >
      <el-table-column
        :label="$t('table.id')"
        type="index"
        sortable="custom"
        align="center"
        width="80"
      ></el-table-column>

      <el-table-column label="名字" min-width="150px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="图片" width="300px" align="center">
        <template slot-scope="scope">
          <img style="width:50px;height:50px" :src="scope.row.image.preview_image" alt />
        </template>
      </el-table-column>
      <el-table-column label="是否置顶" width="200px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.is_top }}</span>
        </template>
      </el-table-column>
      <!-- <el-table-column label="排序" width="200px" align="center">
        <template slot-scope="scope">
          <span>{{scope.row.sort}}</span>
        </template>
      </el-table-column>-->
      <!-- <el-table-column label="内容" min-width="100px" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.content }}</span>
        </template>
      </el-table-column>-->

      <!-- 操作 -->

      <el-table-column
        :label="$t('table.actions')"
        align="center"
        width="220"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="showDetail(row)">查看详情</el-button>
          <el-button size="mini" @click="handleUpdate(row)">编辑</el-button>
          <el-button type="danger" size="mini" @click="delUser(row)">移除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="listQuery.p"
      :limit.sync="listQuery.row"
      @pagination="getList"
    />

    <!-- 模态窗 -->
    <el-dialog
      top="3vh"
      :before-close="close"
      class="model"
      :title="textMap[dialogStatus]"
      :visible.sync="dialogFormVisible"
    >
      <el-form
        ref="dataForm"
        :model="temp"
        label-position="right"
        label-width="120px"
        style="margin:0 50px;"
      >
        <el-form-item label="标题" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>

        <el-form-item label="所属区域">
          <el-cascader
            :props="{ checkStrictly: true }"
            clearable
            v-model="temp.area_id"
            :options="areaList"
          />
        </el-form-item>

        <el-form-item label="内容" prop="title">
          <quill-editor
            class="editor"
            v-model="temp.content"
            :options="editorOption"
            ref="myQuillEditor"
          ></quill-editor>
        </el-form-item>
        <el-form-item label="图片" prop="title">
          <el-upload
            class="avatar-uploader"
            action="http://up.qiniup.com/"
            :show-file-list="false"
            :data="upData"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="images" :src="images" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon" />
          </el-upload>
        </el-form-item>

        <el-form-item label="是否置顶" prop="title">
          <el-switch
            v-model="temp.is_top"
            :active-value="1"
            :inactive-value="0"
            active-color="#13ce66"
            inactive-color="#DCDFE6"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">{{ $t('table.cancel') }}</el-button>
        <el-button
          type="primary"
          v-if="!isNobtn"
          @click="dialogStatus==='create'?createData():updateData()"
        >{{ $t('table.confirm') }}</el-button>
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
import { news_index, add_news, del_news, get_area, cat, token } from '@/api/yunzhijia'
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
  name: 'Url',
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
  watch: {
    'listQuery': {
      handler (val) {
        console.log(val);
        this.getList()
      },
      deep: true
    }
  },
  data () {
    return {
      props: { multiple: true },
      /* 富文本配置 */
      editorOption: {
        modules: {
          toolbar: [
            ['bold', 'italic', 'underline', 'strike'],    //加粗，斜体，下划线，删除线
            [{ 'header': 1 }, { 'header': 2 }],        // 标题，键值对的形式；1、2表示字体大小
            [{ 'list': 'ordered' }, { 'list': 'bullet' }],     //列表
            [{ 'color': [] }, { 'background': [] }],     // 字体颜色，字体背景颜色
            [{ 'align': [] }],    //对齐方式
            ['clean'],    //清除字体样式
            ['image', 'video']    //上传图片、上传视频
          ]
        }
      },
      tops: [{
        value: '',
        label: '全部'
      }, {
        value: 1,
        label: '置顶'
      },
      {
        value: 0,
        label: '未置顶'
      }],
      areaList: [],
      images: '',
      upData: {
        token: ''
      },
      tableKey: 0,
      list: null,
      total: 0,
      cats: [],
      listLoading: true,
      listQuery: {
        p: 1,
        row: 20,
        order: 'url.id desc',
        is_top: ''
      },
      calendarTypeOptions,
      statusOptions: ['published', 'draft', 'deleted'],
      temp: {
        area_id: '',
        id: null,
        title: '',
        content: '',
        is_top: 0,
        image: ''
      },
      isNobtn: false,
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑',
        create: '创建'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [{ required: true, message: 'type is required', trigger: 'change' }],
        timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
        title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      },
      downloadLoading: false
    }
  },
  created () {
    this.getList()
    this.getCat()
    this.getToken()
    this.getArea()
  },
  methods: {
    handleAvatarSuccess (res, file) {
      this.images = URL.createObjectURL(file.raw)
      this.temp.image = res.key
    },
    beforeAvatarUpload (file) {
      const isLt2M = file.size / 1024 / 1024 < 2
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isLt2M
    },
    /* 获取地理位置 */
    getArea () {
      get_area().then(response => {
        // this.areaList = response.result
        this.areaList.push(response.result)
        // console.log();
      })
    },
    /* 获取 七牛云token */
    getToken () {
      token().then(res => {
        this.upData.token = res.result
      })
    },
    delUser (row) {
      del_news({ id: row.id }).then(res => {
        this.$notify({
          title: '成功',
          message: '删除成功',
          type: 'success',
          duration: 2000
        })
        this.getList()
      })
    },
    getCat () {
      const params = {
        p: 1,
        row: 20,
        order: 'url.id desc'
      }
      cat(params).then(res => {
        this.cats = res.result
        console.log(res)
      })
    },
    getList () {
      this.listLoading = true
      news_index(this.listQuery).then(response => {
        this.list = response.result.list
        this.total = response.result.count
        this.listLoading = false
      }).catch(err => {
        this.listLoading = false
        this.list = []
      })
    },
    handleFilter () {
      this.listQuery.p = 1
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
    resetTemp () {
      this.temp = {
        id: null,
        area_id: '',
        title: '',
        content: '',
        is_top: 0,
        image: ''
      }
    },
    handleCreate () {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData () {
      delete this.temp.id
      // const tempObj = { area_id: this.temp.area_id[this.temp.area_id.length - 1], ...this.temp }
      const tempObj = { area_id: this.temp.area_id, ...this.temp }

      add_news(tempObj).then((res) => {
        this.getList()
        this.dialogFormVisible = false
        this.$notify({
          title: '成功',
          message: '创建成功',
          type: 'success',
          duration: 2000
        })
      })
    },
    close () {
      this.dialogFormVisible = false
      this.isNobtn = false
    },
    showDetail (row) {
      this.isNobtn = true
      this.handleUpdate(row)
    },
    handleUpdate (row) {
      this.resetTemp()
      this.temp = { ...row } // copy obj
      /* 单独处理temp内容 */
      this.images = row.image.preview_image
      if (row.is_top == '置顶') {
        this.temp.is_top = 1
      } else {
        this.temp.is_top = 0
      }
      this.temp.area_id = []

      this.dialogStatus = 'update'
      this.dialogFormVisible = true
    },
    updateData () {
      add_news(this.temp).then(() => {
        for (const v of this.list) {
          if (v.id === this.temp.id) {
            const index = this.list.indexOf(v)
            this.list.splice(index, 1, this.temp)
            break
          }
        }
        this.dialogFormVisible = false
        this.$notify({
          title: '成功',
          message: '更新成功',
          type: 'success',
          duration: 2000
        })
      })
    },
    handleDelete (row) {
      this.$notify({
        title: '成功',
        message: '删除成功',
        type: 'success',
        duration: 2000
      })
      const index = this.list.indexOf(row)
      this.list.splice(index, 1)
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

/deep/.model .el-input__inner,
/deep/.model .el-textarea__inner,
/deep/.model .el-select .el-input__inner,
/deep/.model .el-cascader {
  // width: 600px;
  width: 100%;
}
.editor {
  width: 100%;
  height: 300px;
  margin-bottom: 50px;
}

.fixed-width .el-button--mini {
  width: auto;
}
</style>
