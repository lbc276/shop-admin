<template>
  <div class="rights">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <template>
      <el-table :data="rightsList" style="width: 100%">
        <el-table-column prop="authName" label="权限名称"></el-table-column>
        <el-table-column prop="path" label="路径"></el-table-column>
        <el-table-column label="层级">
          <template v-slot:default="obj">
            <p v-if="obj.row.level==='0'">一级</p>
            <p v-if="obj.row.level==='1'">二级</p>
            <p v-if="obj.row.level==='2'">三级</p>
          </template>
        </el-table-column>
      </el-table>
    </template>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rightsList: []
    }
  },
  async created () {
    const { meta, data } = await this.$axios.get('rights/list')
    if (meta.status === 200) {
      this.rightsList = data
      this.$message.success(meta.msg)
    } else {
      this.$message.error(meta.msg)
    }
  }
}
</script>

<style>
</style>
