<template>
  <div class="roles">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-button class="addRoles" @click="addRolesDialog" plain type="primary">添加角色</el-button>
    <template>
      <el-table :data="roleList" style="width: 100%">
        <el-table-column type="expand">
          <template v-slot:default="{ row }">
            <p v-if="row.children.length === 0">暂无权限</p>
            <el-row v-for="l1 in row.children" :key="l1.id">
              <el-col :span="4">
                <el-tag @close="delRights(row,l1.id)" closable>{{l1.authName}}</el-tag>
                <i class="el-icon-arrow-right"></i>
              </el-col>
              <el-col :span="20">
                <el-row v-for="l2 in l1.children" :key="l2.id">
                  <el-col :span="4">
                    <el-tag
                      @close="delRights(row,l2.id)"
                      class="l2"
                      closable
                      type="success"
                    >{{l2.authName}}</el-tag>
                    <i class="el-icon-arrow-right"></i>
                  </el-col>
                  <el-col :span="20">
                    <el-tag
                      @close="delRights(row,l3.id)"
                      class="l3"
                      closable
                      type="warning"
                      v-for="l3 in l2.children"
                      :key="l3.id"
                    >{{l3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="描述"></el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="{row}">
            <el-button
              @click="editRolesDialog(row)"
              plain
              size="small"
              icon="el-icon-edit"
              type="primary"
            ></el-button>
            <el-button
              @click="delRoles(row.id)"
              plain
              size="small"
              icon="el-icon-delete"
              type="danger"
            ></el-button>
            <el-button
              @click="showRightsDialog(row)"
              plain
              size="small"
              icon="el-icon-check"
              type="success"
            >分配角色</el-button>
          </template>
        </el-table-column>
      </el-table>
    </template>
    <!-- 分配角色 -->
    <el-dialog title="提示" :visible.sync="rightsVisible" width="40%">
      <el-tree
        :data="data"
        ref="tree"
        show-checkbox
        default-expand-all
        node-key="id"
        :props="defaultProps"
      ></el-tree>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="rightsVisible = false">取 消</el-button>
          <el-button @click="addRights" type="primary">分 配</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 添加或修改角色 -->
    <el-dialog @close="closeDialog" :title="dialogTitle" :visible.sync="rolesVisible" width="40%">
      <el-form ref="rolesForm" :rules="rules" :model="rolesForm" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="rolesForm.roleName" placeholder="请输入角色名"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="rolesForm.roleDesc" placeholder="请输入角色描述"></el-input>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="rolesVisible = false">取 消</el-button>
          <el-button @click="updateRoles" type="primary">确 定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      roleList: [],
      rightsVisible: false,
      rolesVisible: false,
      data: [],
      dialogTitle: '',
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      roleId: '',
      rolesForm: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      rules: {
        roleName: {
          required: true, message: '请输入角色名', trigger: ['change', 'blur']
        },
        roleDesc: {
          required: true, message: '请输入角色描述', trigger: ['change', 'blur']
        }
      }
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    async getRoleList () {
      const { meta, data } = await this.$axios.get('roles')
      if (meta.status === 200) {
        // console.log(data)
        this.roleList = data
      } else {
        this.$message.error(meta.msg)
      }
    },
    async delRights (row, rightId) {
      const { meta, data } = await this.$axios.delete(
        `roles/${row.id}/rights/${rightId}`
      )
      if (meta.status === 200) {
        console.log(this.roleList)
        console.log(data)
        row.children = data
      } else {
        this.$message.error(meta.msg)
      }
    },
    async delRoles (id) {
      const { meta } = await this.$axios.delete(`roles/${id}`)
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        this.getRoleList()
      } else {
        this.$message.error(meta.msg)
      }
    },
    async showRightsDialog (row) {
      this.roleId = row.id
      this.rightsVisible = true
      const { meta, data } = await this.$axios.get(`rights/tree`)
      if (meta.status === 200) {
        this.data = data
        const ids = []
        row.children.forEach(l1 => {
          l1.children.forEach(l2 => {
            l2.children.forEach(l3 => {
              ids.push(l3.id)
            })
          })
        })
        this.$refs.tree.setCheckedKeys(ids)
      } else {
        this.$message.error(meta.msg)
      }
    },
    async addRights () {
      const rids = [
        ...this.$refs.tree.getHalfCheckedKeys(),
        ...this.$refs.tree.getCheckedKeys()
      ].join(',')
      const { meta } = await this.$axios.post(`roles/${this.roleId}/rights`, {
        rids
      })
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        this.rightsVisible = false
        this.getRoleList()
      } else {
        this.$message.error(meta.msg)
      }
    },
    addRolesDialog () {
      this.rolesVisible = true
      this.dialogTitle = '添加角色'
    },
    editRolesDialog (row) {
      this.rolesVisible = true
      this.dialogTitle = '修改角色'
      this.$nextTick(() => {
        this.rolesForm.id = row.id
        this.rolesForm.roleName = row.roleName
        this.rolesForm.roleDesc = row.roleDesc
      })
    },
    closeDialog () {
      this.$refs.rolesForm.resetFields()
    },
    async updateRoles () {
      const isAdd = this.dialogTitle === '添加角色'
      const url = isAdd ? 'roles' : `roles/${this.rolesForm.id}`
      const method = isAdd ? 'post' : 'put'
      console.log(isAdd, url, method)

      const { meta } = await this.$axios[method](url, this.rolesForm)
      if (meta.status === 200 || meta.status === 201) {
        this.$message.success('操作成功')
        this.rolesVisible = false
        this.getRoleList()
      }
    }
  }
}
</script>

<style lang="scss">
.roles {
  .addRoles {
    margin-bottom: 10px;
  }
  .l2,
  .l3 {
    margin: 5px 0;
  }
  .l3 {
    margin-right: 5px;
  }
}
</style>
