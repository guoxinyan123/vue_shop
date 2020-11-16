<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>添加商品</el-breadcrumb-item>
      </el-breadcrumb>
      <el-card>
        <el-alert title="添加商品信息" type="info" center show-icon :closable="false"></el-alert>
        <el-steps :active="activeIndex - 0" align-center finish-status="success">
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
          <el-step title="完成"></el-step>
        </el-steps>
        <el-form :model="addGoodsForm" :rules="addGoodsFormRules" ref="addGoodsFormRef" label-width="100px" label-position="top">
        <el-tabs tab-position="left" v-model="activeIndex" :before-leave="beforeTabLeave" @tab-click="tabClick">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addGoodsForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addGoodsForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addGoodsForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addGoodsForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
                <el-cascader
                  v-model="addGoodsForm.goods_cat"
                  :options="cateList"
                  :props="{ expandTrigger: 'hover', label: 'cat_name', value: 'cat_id', children: 'children' }"
                  @change="handleChange">
                </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="cb" border size="mini" v-for="(cb, i) in item.attr_vals" :key="i"></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2" >
            <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
              <el-upload
                :action="uploadURL"
                :headers="requestHeaders"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                list-type="picture"
                :on-success="handleSuccess">
                <el-button size="small" type="primary">点击上传</el-button>
              </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addGoodsForm.goods_introduce"></quill-editor>
            <el-button type="primary" @click="addGoods">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
        </el-form>
      </el-card>
      <el-dialog
        title="图片预览"
        :visible.sync="previewDialogVisible"
        width="50%">
        <img :src="previewPath" alt="" class="previewImg">
      </el-dialog>
    </div>
</template>

<script>
import _ from 'lodash'
export default {
  data: function () {
    return {
      activeIndex: '0',
      addGoodsForm: {
        goods_name: '',
        goods_price: '',
        goods_weight: '',
        goods_number: '',
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addGoodsFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      cateList: [],
      manyTableData: [],
      onlyTableData: [],
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      requestHeaders: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewDialogVisible: false,
      previewPath: ''
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
    handleChange() {
      if (this.addGoodsForm.goods_cat.length !== 3) {
        this.addGoodsForm.goods_cat = []
      }
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addGoodsForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    async tabClick() {
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
        if (res.meta.status !== 200) return this.$message.error('获取商品参数失败')
        // console.log(res.data)
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(',')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
        if (res.meta.status !== 200) return this.$message.error('获取商品属性失败')
        // console.log(res.data)
        this.onlyTableData = res.data
      }
    },
    handleSuccess(response) {
      const picInfo = { pic: response.data.tmp_path }
      this.addGoodsForm.pics.push(picInfo)
    },
    handleRemove(file) {
      const index = this.addGoodsForm.pics.findIndex(value => value.pic === file.response.data.tmp_path)
      this.addGoodsForm.pics.splice(index, 1)
    },
    handlePreview(file) {
      console.log(file)
      this.previewPath = file.response.data.url
      this.previewDialogVisible = true
    },
    addGoods() {
      this.$refs.addGoodsFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写必要的表单项！')
        this.manyTableData.forEach(value => {
          const paramsInfo = { attr_id: value.attr_id, attr_value: value.attr_vals.join(',') }
          this.addGoodsForm.attrs.push(paramsInfo)
        })
        this.onlyTableData.forEach(value => {
          const paramsInfo = { attr_id: value.attr_id, attr_value: value.attr_vals }
          this.addGoodsForm.attrs.push(paramsInfo)
        })
        const form = _.cloneDeep(this.addGoodsForm)
        form.goods_cat = form.goods_cat.join(',')
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) return this.$message.error('添加商品失败')
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId: function () {
      if (this.addGoodsForm.goods_cat.length === 3) {
        return this.addGoodsForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
  .el-checkbox {
    margin-right: 5px;
  }
  .previewImg {
    width: 100%;
  }
  .el-button {
    margin-top: 15px;
  }
</style>
