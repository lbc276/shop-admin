<template>
  <div class="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索框 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="searchUser"></el-button>
    </el-input>
    <el-button class="addBtn" type="success" @click="openAddUser" plain>添加用户</el-button>
    <!-- 主体显示列表 -->
    <el-table :data="UserList" style="width: 100%">
      <el-table-column prop="username" label="姓名" width="180"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>
      <el-table-column prop="address" label="用户状态">
        <template v-slot:default="obj">
          <el-switch
            @change="changeStatus(obj.row.id,obj.row.mg_state)"
            v-model="obj.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column prop="address" label="操作">
        <template v-slot:default="{row}">
          <el-button
            @click="openUpdateUser(row)"
            plain
            size="small"
            icon="el-icon-edit"
            type="primary"
          ></el-button>
          <el-button
            @click="delUser(row.id)"
            plain
            size="small"
            icon="el-icon-delete"
            type="danger"
          ></el-button>
          <el-button
            @click="addRoles(row)"
            plain
            size="small"
            icon="el-icon-check"
            type="success"
          >分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页
         @size-change="handleSizeChange"
         @current-change="handleCurrentChange"
    -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    />
    <!-- 添加用户 -->
    <el-dialog title="添加用户" @close="closeDialog" :visible.sync="dialogVisible" width="40%">
      <el-form ref="form" :rules="rules" :model="form" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="form.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="form.password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="form.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="form.mobile" placeholder="请输入手机"></el-input>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 修改用户信息 -->
    <el-dialog title="修改用户" :visible.sync="editVisible" width="40%">
      <el-form ref="editForm" :rules="rules" :model="editForm" label-width="80px">
        <el-form-item label="用户名">
          <el-tag type="info">{{editForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="editVisible = false">取 消</el-button>
          <el-button type="primary" @click="updateUser(editForm.id)">确 定</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 分配角色 -->
    <el-dialog title="分配角色" :visible.sync="addRolesVisible" width="40%">
      <el-form ref="assignForm" :model="assignForm" label-width="80px">
        <el-form-item label="用户名">
          <el-tag type="info">{{assignForm.username}}</el-tag>
        </el-form-item>
        <el-select v-model="assignForm.rid" placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.id"
            :label="item.roleName"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="addRolesVisible = false">取 消</el-button>
          <el-button @click="addUserRoles" type="primary">确 定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      UserList: [],
      query: '',
      pagenum: 1,
      pagesize: 4,
      total: 0,
      dialogVisible: false,
      editVisible: false,
      form: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {
        id: '',
        username: '',
        email: '',
        mobile: ''
      },
      rules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: ['change', 'blur']
          },
          {
            min: 3,
            max: 12,
            message: '长度在 3 到 12 个字符',
            trigger: ['change', 'blur']
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: ['change', 'blur']
          },
          {
            min: 3,
            max: 12,
            message: '长度在 3 到 12 个字符',
            trigger: ['change', 'blur']
          }
        ],
        email: [
          {
            type: 'email',
            message: '请输入正确的邮箱',
            trigger: ['change', 'blur']
          }
        ],
        mobile: [{ pattern: /^1[3-9]\d{9}$/, message: '请输入正确的手机号' }]
      },
      assignForm: {
        id: '', // 用户id
        rid: '', // 角色id
        username: '' // 专门用于展示的用户名
      },
      addRolesVisible: false,
      options: []
    }
  },
  created () {
    this.userList()
  },
  methods: {
    async userList () {
      const { meta, data } = await this.$axios.get('users', {
        params: {
          query: this.query,
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      })
      if (meta.status === 200) {
        this.UserList = data.users
        this.total = data.total
      } else {
        this.$message.error(meta.msg)
      }
    },
    handleSizeChange (value) {
      this.pagesize = value
      this.pagenum = 1
      this.userList()
    },
    handleCurrentChange (value) {
      this.pagenum = value
      this.userList()
    },
    async delUser (id) {
      try {
        await this.$confirm('确定要删除吗?', '温馨提示', {
          type: 'warning'
        })
        const { meta } = await this.$axios.delete(`users/${id}`)
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          if (this.UserList.length === 1 && this.pagenum > 1) {
            this.pagenum--
          }
          this.userList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    searchUser () {
      this.pagenum = 1
      this.userList()
    },
    async changeStatus (id, type) {
      const { meta } = await this.$axios.put(`users/${id}/state/${type}`)
      if (meta.status === 200) {
        this.$message.success(meta.msg)
      } else {
        this.$message.error(meta.msg)
      }
    },
    openAddUser () {
      this.dialogVisible = true
    },
    async addUser () {
      try {
        await this.$refs.form.validate()
        const { meta } = await this.$axios.post('users', this.form)

        if (meta.status === 201) {
          this.$message.success(meta.msg)
          this.dialogVisible = false
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    openUpdateUser (row) {
      this.editForm.username = row.username
      this.editForm.email = row.email
      this.editForm.mobile = row.mobile
      this.editForm.id = row.id
      this.editVisible = true
    },
    async updateUser (id) {
      try {
        const { email, mobile } = this.editForm
        await this.$refs.editForm.validate()
        const { meta } = await this.$axios.put(`users/${id}`, {
          email,
          mobile
        })
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          this.editVisible = false
        } else {
          this.$message.success(meta.msg)
        }
      } catch (e) {}
    },
    closeDialog () {
      this.$refs.form.resetFields()
    },
    async addRoles (row) {
      this.assignForm.username = row.username
      this.assignForm.id = row.id
      this.addRolesVisible = true
      const res = await this.$axios.get(`users/${row.id}`)
      if (res.meta.status === 200) {
        const rid = res.data.rid
        this.assignForm.rid = rid === -1 ? '' : rid
      }
      const { meta, data } = await this.$axios.get('roles')
      if (meta.status === 200) {
        this.options = data
      }
    },
    async addUserRoles () {
      const { id, rid } = this.assignForm
      console.log(rid)

      if (rid === '') {
        this.$message.error('请选择角色')
      }
      const { meta } = await this.$axios.put(`users/${id}/role`, { rid })
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        this.addRolesVisible = false
        this.userList()
      } else {
        this.$message.error(meta.msg)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.users {
  .input-with-select {
    width: 400px;
    margin-bottom: 10px;
  }
  .addBtn {
    margin-left: 10px;
  }
}
</style>
