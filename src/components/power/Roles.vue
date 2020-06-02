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
          <el-button type="primary" @click="showAddDialog()">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开行 -->
        <el-table-column type="expand">
          <template v-slot:default="scope">
            <el-row
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
              :class="['vcenter','bdbottom',i1===0?'bdtop':'']"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightByid(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二、三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环签到渲染二级权限 -->
                <el-row
                  v-for="(item2,i2) in item1.children"
                  :key="item2.id"
                  :class="['vcenter',i2 !== 0 ? 'bdtop':'']"
                >
                  <!-- 渲染二级权限 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightByid(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3,i3) in item2.children"
                      :key="item3.id"
                      :class="[i3 !== 0 ? 'bdtop':'']"
                      closable
                      @close="removeRightByid(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>

            <!-- <pre>{{scope.row}}</pre> -->
          </template>
        </el-table-column>

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
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 修改角色的对话框 -->
      <el-dialog title="修改角色信息" :visible.sync="editDialogVisible" width="50%">
        <el-form :model="editForm" ref="editFormRef" label-width="70px">
          <el-form-item label="角色名称">
            <el-input v-model="editForm.roleName" disabled></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" clss="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRoleInfo">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改角色权限的对话框 -->
      <el-dialog
        title="修改角色权限"
        :visible.sync="setRightDialogVisible"
        width="50%"
        @close="setRightDialogClosed"
      >
        <!-- node-key绑定的id是rightlist中的id -->
        <el-tree
          :data="rightlist"
          :props="treeProps"
          show-checkbox
          node-key="id"
          default-expand-all
          :default-checked-keys="defaultKeys"
          ref="treeRef"
        ></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights()">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 添加角色
      1.按钮绑定事件
      2.对话框设置为可见
      3.v-model双向绑定信息，data设置一个对象存储表单form
      4.点击确定进行发送请求，点击取消回到界面，清空form-->
      <!-- 添加角色权限的对话框 -->
      <el-dialog
        title="添加角色"
        :visible.sync="addDialogVisible"
        width="50%"
        :close-on-click-modal="false"
      >
        <el-form :model="addForm" ref="addFormRef" label-width="70px">
          <el-form-item label="角色名称">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRoleInfo">确 定</el-button>
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
      rolelist: [],
      editForm: {},
      addForm: {},
      editDialogVisible: false,
      addDialogVisible: false,
      setRightDialogVisible: false,
      rightlist: [],
      // 树形控件属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点的id值
      defaultKeys: [],
      // 即将分配权限的角色的id
      roleId: ''
    }
  },
  created() {
    this.getRolelist()
  },
  methods: {
    async getRolelist() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
    },
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }
      this.editForm = res.data
    },
    async removeRoleById(id) {
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
      this.getRolelist()
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
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色信息失败')
      }
      this.$message.success('修改角色信息成功！')
      // 隐藏添加用户的对话框
      this.editDialogVisible = false
      // 重新获取用户列表
      this.getRolelist()
    },
    async addRoleInfo() {
      const { data: res } = await this.$http.post('roles', {
        roleName: this.addForm.roleName,
        roleDesc: this.addForm.roleDesc
      })
      if (res.meta.status !== 201) {
        return this.$message.error('添加角色信息失败')
      }
      this.$message.success('添加角色信息成功！')
      // 隐藏添加角色的对话框
      this.addDialogVisible = false
      // 重新获取用户列表
      this.getRolelist()
    },
    showAddDialog() {
      this.addDialogVisible = true
    },
    // 根据id删除对应权限
    async removeRightByid(role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => {
        return err
      })
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(
        'roles/' + role.id + '/rights/' + rightId
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      // console.log(res)
      // this.getRolelist()
      role.children = res.data
      return this.$message.success('删除权限成功')
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限的数据
      this.setRightDialogVisible = true
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message('获取权限数据失败')
      }
      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defaultKeys)
      // 把获取到的权限数据保存到data中
      this.rightlist = res.data
      console.log(res)
    },
    // 通过递归的形式获得角色下所有三级权限的id，保存到数组中
    getLeafKeys(node, array) {
      if (!node.children) {
        return array.push(node.id)
      }
      node.children.forEach(element => {
        this.getLeafKeys(element, array)
      })
    },
    setRightDialogClosed() {
      this.defaultKeys = []
    },
    async allotRights() {
      this.setRightDialogVisible = false
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('修改角色权限失败')
      }
      this.$message.success('修改角色权限成功')
      this.getRolelist()
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
