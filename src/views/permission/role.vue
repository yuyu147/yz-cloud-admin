<template>
  <div class="app-container">
    <el-button type="primary" @click="handleAddRole">{{ $t('permission.addRole') }}</el-button>

    <el-table :data="rolesList" style="width: 100%;margin-top:30px;" border>
      <el-table-column align="center" type="index" label="角色ID" width="220">
        <!-- <template slot-scope="scope">{{ scope.row.id }}</template> -->
      </el-table-column>
      <el-table-column align="center" label="角色名字">
        <template slot-scope="scope">{{ scope.row.title }}</template>
      </el-table-column>
      <!-- <el-table-column align="header-center" label="角色描述">
        <template slot-scope="scope">{{ scope.row.content }}</template>
      </el-table-column>-->
      <el-table-column width="200" align="center" label="操作">
        <template slot-scope="scope">
          <el-button
            type="primary"
            size="small"
            @click="handleEdit(scope)"
          >{{ $t('permission.editPermission') }}</el-button>
          <el-button
            type="danger"
            size="small"
            @click="handleDelete(scope)"
          >{{ $t('permission.delete') }}</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog :visible.sync="dialogVisible" :title="dialogType==='edit'?'Edit Role':'New Role'">
      <el-form :model="role" label-width="80px" label-position="left">
        <el-form-item label="名字">
          <el-input v-model="role.title" placeholder="角色名字" />
        </el-form-item>
        <!-- <el-form-item label="角色简述">
          <el-input
            v-model="role.content"
            :autosize="{ minRows: 2, maxRows: 4}"
            type="textarea"
            placeholder="在此输入角色简述"
          />
        </el-form-item>-->
        <el-form-item label="角色权限">
          <el-tree
            ref="tree"
            :check-strictly="checkStrictly"
            :data="routesData"
            :props="defaultProps"
            show-checkbox
            node-key="path"
            class="permission-tree"
          />
        </el-form-item>
      </el-form>
      <div style="text-align:right;">
        <el-button type="danger" @click="dialogVisible=false">{{ $t('permission.cancel') }}</el-button>
        <el-button type="primary" @click="confirmRole">{{ $t('permission.confirm') }}</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import path from 'path'
import { deepClone } from '@/utils'
/* 此处 getRoutes 就为正常路由 */
import { getRoutes, addRole, deleteRole, updateRole } from '@/api/role'
import { roleIndex, edit_role, del_role } from "@/api/yunzhijia";
import { asyncRoutes, constantRoutes } from '@/router'
import i18n from '@/lang'

const defaultRole = {
  key: '',
  name: '',
  description: '',
  routes: []
}

export default {
  data () {
    return {
      role: Object.assign({}, defaultRole),
      routes: [],
      rolesList: [],
      dialogVisible: false,
      dialogType: 'new',
      checkStrictly: false,
      defaultProps: {
        children: 'children',
        label: 'title'
      }
    }
  },
  computed: {
    routesData () {
      return this.routes
    }
  },
  created () {
    // Mock: get all routes and roles list from server
    this.getRoutes()
    this.getRoles()
  },
  methods: {
    getRoutes () {
      // const res = await getRoutes()
      // this.serviceRoutes = res.data
      // const routes = this.generateRoutes(res.data)

      this.serviceRoutes = asyncRoutes
      const routes = this.generateRoutes(asyncRoutes)
      this.routes = routes
      console.log(this.routes);
    },
    getRoles () {
      let params = {
        p: 1,
        row: 200
      }
      roleIndex(params).then(res => {
        this.rolesList = res.result.list
      })
    },
    i18n (routes) {
      const app = routes.map(route => {
        route.title = i18n.t(`${route.title}`)
        if (route.children) {
          route.children = this.i18n(route.children)
        }
        return route
      })
      return app
    },
    // Reshape the routes structure so that it looks the same as the sidebar
    /* 路由处理函数 */
    generateRoutes (routes, basePath = '/') {
      const res = []

      for (let route of routes) {
        // skip some route
        // if (route.hidden) { continue }

        const onlyOneShowingChild = this.onlyOneShowingChild(route.children, route)

        if (route.children && onlyOneShowingChild && !route.alwaysShow) {
          route = onlyOneShowingChild
        }

        const data = {
          path: path.resolve(basePath, route.path),
          title: route.meta && route.meta.title
        }

        // recursive child routes
        if (route.children) {
          data.children = this.generateRoutes(route.children, data.path)
        }
        res.push(data)
      }
      return res
    },
    generateArr (routes) {
      let data = []
      routes.forEach(route => {
        data.push(route)
        if (route.children) {
          const temp = this.generateArr(route.children)
          if (temp.length > 0) {
            data = [...data, ...temp]
          }
        }
      })
      return data
    },
    handleAddRole () {
      this.role = Object.assign({}, defaultRole)
      if (this.$refs.tree) {
        this.$refs.tree.setCheckedNodes([])
      }
      this.dialogType = 'new'
      this.dialogVisible = true
    },
    handleEdit (scope) {
      this.dialogType = 'edit'
      this.dialogVisible = true
      this.checkStrictly = false
      this.role = scope.row

      this.$nextTick(() => {
        // const routes = this.generateRoutes(this.role.content, '/permission')
        /* 在此获取的路由 */
        // console.log(routes);
        // console.log(this.generateArr(routes));
        let arr = this.role.content.split(',')
        this.$refs.tree.setCheckedKeys(arr)
        // this.$refs.tree.setCheckedNodes(this.generateArr(routes))
        // // set checked state of a node not affects its father and child nodes
        // this.checkStrictly = false
      })
    },
    handleDelete ({ $index, row }) {
      del_role({ id: row.id }).then(res => {
        this.getRoles()
        this.$message({
          type: 'success',
          message: 'Delete succed!'
        })
      })
    },
    generateTree (routes, basePath = '/postulant', checkedKeys) {
      const res = []

      for (const route of routes) {
        const routePath = path.resolve(basePath, route.path)

        // recursive child routes
        if (route.children) {
          route.children = this.generateTree(route.children, routePath, checkedKeys)
        }

        if (checkedKeys.includes(routePath) || (route.children && route.children.length >= 1)) {
          res.push(route)
        }
      }
      return res
    },
    async confirmRole () {
      const isEdit = this.dialogType === 'edit'

      const checkedKeys = this.$refs.tree.getCheckedKeys()
      this.role.routes = this.generateTree(deepClone(this.serviceRoutes), '/postulant', checkedKeys)


      if (isEdit) {
        await edit_role({ id: this.role.id, content: checkedKeys.toString(), title: this.role.title })
        for (let index = 0; index < this.rolesList.length; index++) {
          if (this.rolesList[index].key === this.role.key) {
            this.rolesList.splice(index, 1, Object.assign({}, this.role))
            break
          }
        }
      } else {
        const { data } = await edit_role({ content: checkedKeys.toString(), title: this.role.title })
        for (let index = 0; index < this.rolesList.length; index++) {
          if (this.rolesList[index].key === this.role.key) {
            this.rolesList.splice(index, 1, Object.assign({}, this.role))
            break
          }
        }
        // this.role.key = data.key
        // this.rolesList.push(this.role)
      }

      this.getRoles()
      const { description, key, name } = this.role
      this.dialogVisible = false
      this.$notify({
        title: 'Success',
        dangerouslyUseHTMLString: true,
        message: `
            
          `,
        type: 'success'
      })
    },
    // reference: src/view/layout/components/Sidebar/SidebarItem.vue
    onlyOneShowingChild (children = [], parent) {
      let onlyOneChild = null
      const showingChildren = children.filter(item => !item.hidden)

      // When there is only one child route, the child route is displayed by default
      if (showingChildren.length === 1) {
        onlyOneChild = showingChildren[0]
        onlyOneChild.path = path.resolve(parent.path, onlyOneChild.path)
        return onlyOneChild
      }

      // Show parent if there are no child route to display
      if (showingChildren.length === 0) {
        onlyOneChild = { ...parent, path: '', noShowingChildren: true }
        return onlyOneChild
      }

      return false
    }
  }
}
</script>

<style lang="scss" scoped>
.app-container {
  .roles-table {
    margin-top: 30px;
  }
  .permission-tree {
    margin-bottom: 30px;
  }
}
</style>
