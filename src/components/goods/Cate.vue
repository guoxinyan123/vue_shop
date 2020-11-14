<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
      </el-breadcrumb>
      <el-card>
        <el-row>
          <el-col><el-button type="primary" @click="ShowAddCateDialog">添加分类</el-button></el-col>
        </el-row>
        <tree-table
          class="tree-table"
          :data="cateList"
          :columns="columns"
          border
          :selection-type="false"
          :expand-type="false"
          show-index
          index-text="#"
          :show-row-hover="false">
          <template slot="isOk" slot-scope="scope">
            <i v-if="scope.row.cat_deleted == false" class="el-icon-success" style="color: green"></i>
            <i v-else class="el-icon-error" style="color: red"></i>
          </template>
          <template slot="order" slot-scope="scope">
            <el-tag size="mini" v-if="scope.row.cat_level == 0">一级</el-tag>
            <el-tag size="mini" v-else-if="scope.row.cat_level == 1" type="success">二级</el-tag>
            <el-tag size="mini" v-else type="warning">三级</el-tag>
          </template>
          <template slot="opt" slot-scope="scope">
            <el-button size="mini" icon="el-icon-edit" type="primary">编辑</el-button>
            <el-button size="mini" icon="el-icon-delete" type="danger" @click="deleteCate(scope.row)">删除</el-button>
          </template>
        </tree-table>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="queryInfo.pagenum"
          :page-size="queryInfo.pagesize"
          :page-sizes="[5, 10, 15, 20]"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </el-card>
      <el-dialog
        @close="closeAddCateDialog"
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%">
        <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" >
          <el-form-item label="分类名称: " prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类: ">
            <el-cascader
              v-model="selectedKeys"
              :options="parentCateList"
              :props="cascaderProps"
              clearable
              @change="parentCateChanged"></el-cascader>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
export default {
  data: function () {
    return {
      queryInfo: {
        type: [],
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      total: 0,
      columns: [
        { label: '分类名称', prop: 'cat_name' },
        { label: '是否有效', type: 'template', template: 'isOk' },
        { label: '排序', type: 'template', template: 'order' },
        { label: '操作', type: 'template', template: 'opt' }
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
        checkStrictly: 'true'
      },
      selectedKeys: []
    }
  },
  created: function () {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      // console.log(res)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getCateList()
    },
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getCateList()
    },
    ShowAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error('获取父级分类数据失败')
      // console.log(res)
      this.parentCateList = res.data
    },
    parentCateChanged() {
      // console.log(this.selectedKeys)
      if (this.selectedKeys.length === 0) {
        this.addCateForm.cat_level = 0
        this.addCateForm.cat_pid = 0
      } else if (this.selectedKeys.length === 1) {
        this.addCateForm.cat_level = 1
        this.addCateForm.cat_pid = this.selectedKeys[0]
      } else {
        this.addCateForm.cat_level = 2
        this.addCateForm.cat_pid = this.selectedKeys[1]
      }
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$message.error('添加分类失败')
        // console.log(res)
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    closeAddCateDialog() {
      this.$refs.addCatFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    async deleteCate(category) {
      const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.error('已取消删除')
      const { data: res } = await this.$http.delete('categories/' + category.cat_id)
      if (res.meta.status !== 200) return this.$message.error('删除分类数据失败')
      this.getCateList()
      this.$message.success('删除分类数据成功')
    }
  }
}
</script>

<style scoped>
  .tree-table {
    margin-top: 15px;
    margin-bottom: 15px;
  }
  .el-cascader {
    width: 100%;
  }
</style>
