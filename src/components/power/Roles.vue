<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-eleme">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand"></el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="400px">
          <template v-slot:default="scope">
            <!-- {{scope.row}} -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            >编辑</el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRoleById(scope.row.id)"
            >删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 修改用户权限的对话框 -->
      <el-dialog
        title="修改用户权限"
        :visible.sync="editDialogVisible"
        width="50%"
        :close-on-click-modal="false"
      >
        <el-form :model="editForm" ref="editFormRef" label-width="70px">
          <el-form-item label="角色名称">
            <el-input v-model="editForm.roleName" disabled></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRoleInfo">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      roleList: [],
      editForm: {},
      editDialogVisible: false
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.roleList = res.data
      console.log(this.roleList)
    },
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }
      console.log(res)
      this.editForm = res.data
    },
    async removeRoleById(id) {
      console.log(id)
      // 询问用户是否删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => {
        return err
      })
      // 如果用户点击确认,返回confirm字符串
      // 如果用户点击取消，返回cancel字符串
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败')
      }
      this.getRoleList()
      return this.$message.success('删除角色成功')
    },
    async editRoleInfo() {
      const { data: res } = await this.$http.put(
        'roles/' + this.editForm.roleId,
        {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        }
      )
      console.log(res.meta.status)
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色信息失败')
      }
      this.$message.success('修改角色信息成功！')
      // 隐藏添加用户的对话框
      this.editDialogVisible = false
      // 重新获取用户列表
      this.getRoleList()
    }
  }
}
</script>

<style lang="less" scoped>
</style>
