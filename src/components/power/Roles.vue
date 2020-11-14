<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
      </el-breadcrumb>
      <el-card>
        <el-row>
          <el-col>
            <el-button type="primary">添加角色</el-button>
          </el-col>
        </el-row>
        <el-table :data="rolesList" border stripe style="width: 100%;">
          <el-table-column type="expand" width="100">
            <template slot-scope="scope">
              <el-row :key="item1.id" :class="['bdbottom', i1 === 0 ? 'bdtop': '' ,'vcenter']" v-for="(item1, i1) in scope.row.children" >
                <el-col :span="5">
                  <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="19">
                  <el-row :key="item2.id" :class="[ i2 === 0 ? '' : 'bdtop', 'vcenter' ] "  v-for="(item2, i2) in item1.children">
                    <el-col :span="6">
                      <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="18">
                      <el-tag type="warning" closable @close="removeRightById(scope.row, item3.id)" :key="item3.id" v-for="item3 in item2.children">{{item3.authName}}</el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <el-table-column type="index" label="#" width="100"></el-table-column>
          <el-table-column prop="roleName" label="角色名称"></el-table-column>
          <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
              <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
              <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-card>
      <el-dialog
        @close="closeSetRightDialog"
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%">
        <el-tree
          :data="rightsList"
          ref="treeRef"
          show-checkbox
          node-key="id"
          default-expand-all
          :default-checked-keys="defKeys"
          :props="treeProps">
        </el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
export default {
  data: function () {
    return {
      rolesList: [],
      setRightDialogVisible: false,
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defKeys: [],
      roleId: ''
    }
  },
  created: function () {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      // console.log(res)
      if (res.meta.status !== 200) this.$message.error('获取角色列表失败')
      this.rolesList = res.data
    },
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.error('已取消删除')
      const { data: res } = await this.$http.delete('roles/' + role.id + '/rights/' + rightId)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      // console.log(res)
      // this.getRolesList()
      role.children = res.data
      this.$message.success('已删除权限')
    },
    async showSetRightDialog(role) {
      this.roleId = role.id
      this.defKeys = []
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限失败')
      // console.log(res)
      this.rightsList = res.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 递归函数
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      // var that = this
      // node.children.forEach(function (val) {
      //   that.getLeafKeys(val, arr)
      // })
      node.children.forEach(val => this.getLeafKeys(val, arr))
    },
    // 解决bug  点击下一个权限设置后,defkeys保存了上一个角色的权限数据, 导致下一个角色的权限设置包含上一个角色的数据
    closeSetRightDialog() {
      this.defKeys = []
    },
    async allotRights() {
      var idStr = this.$refs.treeRef.getCheckedKeys().join(',') + ',' + this.$refs.treeRef.getHalfCheckedKeys().join(',')
      const { data: res } = await this.$http.post('roles/' + this.roleId + '/rights', { rids: idStr })
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style scoped>
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
