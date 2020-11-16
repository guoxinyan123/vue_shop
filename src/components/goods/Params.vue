<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>参数列表</el-breadcrumb-item>
      </el-breadcrumb>
      <el-card>
        <el-alert title="注意：只允许为第三级分类设置相关参数" type="warning" show-icon :closable="false"></el-alert>
        <el-row>
          <el-col>
            <span>选择商品分类：</span>
            <el-cascader
              v-model="selectedCateKeys"
              :options="cateList"
              :props="{ expandTrigger: 'hover', label: 'cat_name', value: 'cat_id', children: 'children' }"
              @change="handleChange"></el-cascader>
          </el-col>
        </el-row>
        <el-tabs v-model="activeName" @tab-click="getParamsList">
          <el-tab-pane label="动态参数" name="many">
            <el-button type="primary" size="mini" :disabled="flag" @click="addParamsDialogVisible">添加参数</el-button>
            <el-table
              border
              stripe
              :data="manyParamsList"
              style="width: 100%">
              <el-table-column type="expand" width="180">
                <template slot-scope="scope">
                  <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i, scope.row)">{{item}}</el-tag>
                  <el-input
                    class="input-new-tag"
                    ref="saveTagInput"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope.row)"
                    @blur="handleInputConfirm(scope.row)">
                  </el-input>
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index" label="#" width="180"></el-table-column>
              <el-table-column prop="attr_name" label="参数名称"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button size="mini" type="primary" icon="el-icon-edit" @click="editParamsDialogVisible(scope.row)">修改</el-button>
                  <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteParams(scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="静态属性" name="only">
            <el-button type="primary" size="mini" :disabled="flag" @click="addParamsDialogVisible">添加属性</el-button>
            <el-table
              :data="onlyParamsList"
              style="width: 100%">
              <el-table-column type="expand" width="180">
                <template slot-scope="scope">
                  <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i, scope.row)">{{item}}</el-tag>
                  <el-input
                    class="input-new-tag"
                    ref="saveTagInput"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope.row)"
                    @blur="handleInputConfirm(scope.row)">
                  </el-input>
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index" label="#" width="180"></el-table-column>
              <el-table-column prop="attr_name" label="属性名称"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button type="primary" icon="el-icon-edit" size="mini" @click="editParamsDialogVisible(scope.row)">修改</el-button>
                  <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
      </el-card>
      <el-dialog
        @close="closeAddParamsDialog"
        :title="activeName === 'many' ? '添加动态参数' : '添加静态属性' "
        :visible.sync="addParamsDialog"
        width="50%">
        <el-form ref="addParamsFormRef" :model="addParamsForm" :rules="addParamsFormRules" label-width="80px">
          <el-form-item :label="activeName === 'many' ? '动态参数' : '静态属性' " prop="attr_name">
            <el-input v-model="addParamsForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addParamsDialog = false">取 消</el-button>
          <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
      </el-dialog>
      <el-dialog
        @close="closeEditParamsDialog"
        :title="activeName === 'many' ? '修改动态参数' : '修改静态属性' "
        :visible.sync="editParamsDialog"
        width="50%">
        <el-form ref="editParamsFormRef" :model="editParamsForm" :rules="editParamsFormRules" label-width="80px">
          <el-form-item :label="activeName === 'many' ? '动态参数' : '静态属性' " prop="attr_name">
            <el-input v-model="editParamsForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editParamsDialog = false">取 消</el-button>
          <el-button type="primary" @click="editParams">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
export default {
  data: function () {
    return {
      cateList: [],
      selectedCateKeys: [],
      activeName: 'many',
      flag: true,
      manyParamsList: [],
      onlyParamsList: [],
      addParamsDialog: false,
      addParamsForm: {
        attr_name: ''
      },
      addParamsFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }]
      },
      editParamsDialog: false,
      editParamsForm: {
        attr_name: '',
        attr_id: 0
      },
      editParamsFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }]
      }
    }
  },
  created: function () {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      // console.log(res)
      this.cateList = res.data
    },
    async handleChange() {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.flag = true
        this.manyParamsList = []
        this.onlyParamsList = []
        return
      }
      this.flag = false
      this.getParamsList()
    },
    async getParamsList() {
      const { data: res } = await this.$http.get('categories/' + this.selectedCateKeys[2] + '/attributes', { params: { sel: this.activeName } })
      if (res.meta.status !== 200) return this.$message.error('获取参数列表失败')
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals = item.attr_vals.split(',') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyParamsList = res.data
      } else {
        this.onlyParamsList = res.data
      }
    },
    addParamsDialogVisible() {
      this.addParamsDialog = true
    },
    addParams() {
      var that = this
      this.$refs.addParamsFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories/' + this.selectedCateKeys[2] + '/attributes', {
          attr_name: this.addParamsForm.attr_name,
          attr_sel: this.activeName,
          attr_vals: ''
        })
        // console.log(res)
        if (res.meta.status !== 201) return this.$message.error('添加参数失败')
        this.$message.success('添加参数成功')
        that.getParamsList()
        this.addParamsDialog = false
      })
    },
    closeAddParamsDialog() {
      this.$refs.addParamsFormRef.resetFields()
    },
    async editParamsDialogVisible(params) {
      const { data: res } = await this.$http.get('categories/' + params.cat_id + '/attributes/' + params.attr_id, {
        attr_sel: this.activeName
      })
      if (res.meta.status !== 200) return this.$message.error('获取参数失败')
      this.editParamsForm.attr_name = res.data
      this.editParamsDialog = true
    },
    async editParams() {
      const { data: res } = await this.$http.put('categories/' + this.selectedCateKeys[2] + '/attributes/' + this.editParamsForm.attr_id, {
        attr_name: this.editParamsForm.attr_name,
        attr_sel: this.activeName
      })
      if (res.meta.status !== 200) return this.$message.error('更新参数失败')
      this.getParamsList()
      this.$message.success('更新参数成功')
      this.editParamsDialog = false
    },
    closeEditParamsDialog() {
      this.$refs.editParamsFormRef.resetFields()
    },
    async deleteParams(params) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete('categories/' + params.cat_id + '/attributes/' + params.attr_id)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getParamsList()
    },
    showInput(row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    async handleInputConfirm(row) {
      // console.log(row)
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put('categories/' + row.cat_id + '/attributes/' + row.attr_id, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) return this.$message.error('修改参数失败')
      this.$message.success('修改参数成功')
    },
    async handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  }
}
</script>

<style lang="less" scoped>
  .el-row {
    margin: 15px;
  }
  .el-cascader {
    width: 300px;
  }
  .el-tag {
    margin: 10px;
  }
  .input-new-tag {
    width: 150px;
  }
</style>
