<template>
  <div>
    <!--面包屑导航区-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图区域-->
    <el-card>
      <!--添加角色按钮-->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>

      <!--展示角色列表-->
      <el-table :data="roleList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column type="角色名称" prop="authName"></el-table-column>
        <el-table-column type="路径" prop="path"></el-table-column>
        <el-table-column type="角色名称" prop="authName"></el-table-column>
        <el-table-column type="路径" prop="path"></el-table-column>

      </el-table>

    </el-card>
  </div>
</template>

<script>
  export default {
    name: 'Roles',
    data() {
      return {
        //所有角色列表数据
        roleList: []
      }
    },
    created() {
      this.getRolesList()
    },
    methods: {
      //获取所有角色列表
      async getRolesList() {
        const {data:res} = await this.$http.post('roles/list')
        if(res.code !== '200') return this.$message.error('获取角色列表失败！')
        this.roleList = res.data
        this.$message.success('获取角色列表成功！')
      }
    }
  }
</script>

<style scoped>

</style>
