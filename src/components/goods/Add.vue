<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>添加商品</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区 -->
        <el-card>
            <!-- 文本提示栏 -->
            <el-alert
                title="添加商品信息"
                type="info"
                center
                show-icon
                :closable="false">
            </el-alert>
            <!-- 步骤条 -->
            <el-steps :active="active" finish-status="success" align-center>
                <el-step title="基本信息"></el-step>
                <el-step title="商品参数"></el-step>
                <el-step title="商品属性"></el-step>
                <el-step title="商品图片"></el-step>
                <el-step title="商品内容"></el-step>
                <el-step title="完成"></el-step>
            </el-steps>
            <!-- 表单区 -->
            <el-form :model="goodsForm" :rules="goodsRules" ref="goodsRuleFormRef" label-width="100px" class="demo-ruleForm" label-position="top">
                <!-- 分页区 -->
                <el-tabs tab-position="left" :stretch="true" value="0" :before-leave="checkCateList" @tab-click="getTabIndex">
                    <el-tab-pane label="基本信息" name="0">
                        <!-- 基本信息表单区 -->
                        <el-form-item label="商品名称" prop="goods_name">
                            <el-input v-model="goodsForm.goods_name" width="100%"></el-input>
                        </el-form-item>
                        <el-form-item label="商品价格" prop="goods_price">
                            <el-input v-model="goodsForm.goods_price" width="100%" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品重量" prop="goods_weight">
                            <el-input v-model="goodsForm.goods_weight" width="100%" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品数量" prop="goods_number">
                            <el-input v-model="goodsForm.goods_number" width="100%" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品分类" prop="category">
                            <el-cascader
                                expand-trigger='hover'
                                v-model="selectedKeys"
                                :options="cateList"
                                :props="cascaderProps"
                                @change="hasChoosen"
                            >
                            </el-cascader>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品参数" name="1">
                        <!-- 渲染动态参数 -->
                        <el-form-item :label="item.attr_name" v-for="item in manyVals" :key="item.attr_id">
                            <el-checkbox-group v-model="item.attr_vals">
                                <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i"></el-checkbox>
                            </el-checkbox-group>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品属性" name="2">
                        <!-- 渲染静态属性 -->
                        <el-form-item :label="item.attr_name" v-for="item in onlyVals" :key="item.attr_id">
                            <el-input v-model="item.attr_vals"></el-input>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品图片" name="3">
                        <!-- 上传图片按钮 -->
                        <el-upload
                            action="http://127.0.0.1:8888/api/private/v1/upload/"
                            :on-preview="handlePreview"
                            :on-remove="handleRemove"
                            :on-success="handleSuccess"
                            :file-list="fileList"
                            :headers="headerObj"
                            list-type="picture">
                            <el-button size="small" type="primary">点击上传</el-button>
                            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                        </el-upload>
                    </el-tab-pane>
                    <el-tab-pane label="商品内容" name="4">
                        <!-- 富文本编辑器 -->
                        <quill-editor
                            v-model="goodsForm.goods_introduce"
                        />
                        <!-- 添加商品按钮 -->
                        <el-button type="primary" class="addbtn" @click="addGoods">添加商品</el-button>
                    </el-tab-pane>
                </el-tabs>
            </el-form>

        </el-card>
        <!-- 预览图片对话框 -->
        <el-dialog
            title="图片预览"
            :visible.sync="imgPreviewDialogVisible"
            width="80%">
            <img :src="curentImgPath" alt="" width="100%">
        </el-dialog>
    </div>
</template>
<script>
import _ from 'lodash'
export default {

  created () {
    this.getCateList()
  },
  data () {
    var checkGoodsNum = (rule, value, callback) => {
      const regNum = /^\d+$/
      if (regNum.test(value)) {
        callback()
      } else {
        callback(new Error('请输入非负整数'))
      }
    }
    var checkGoodsPrice = (rule, value, callback) => {
      const regNum = /^\d+(\.{0,1}\d+){0,1}$/
      if (regNum.test(value)) {
        callback()
      } else {
        callback(new Error('请输入非负数'))
      }
    }
    return {
      // 当前激活的步骤
      active: 0,
      // 商品信息
      goodsForm: {
        goods_name: '',
        goods_cat: [],
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        attrs: [],
        pics: [
        ],
        goods_introduce: ''
      },
      // 验证规则
      goodsRules: {
        goods_name: { required: true, message: '请输入商品名称', trigger: 'blur' },
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' },
          { validator: checkGoodsPrice, trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' },
          { validator: checkGoodsPrice, trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' },
          { validator: checkGoodsNum, trigger: 'blur' }
        ]
      },
      // 级联选择器数据
      cateList: [],
      selectedKeys: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 动态参数数组
      manyVals: [],
      // 静态属性数组
      onlyVals: [],
      // 商品图片数组
      fileList: [

      ],
      // 头部请求对象
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 图片预览对话框可见
      imgPreviewDialogVisible: false,
      // 当前图片路径
      curentImgPath: ''
    }
  },
  methods: {
    // 商品价格改变
    priceChange () {

    },
    // 商品重量改变
    weightChange () {

    },
    // 商品数量改变
    numberChange () {

    },
    // 获取分类列表
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 3 } })
      this.cateList = res.data
    },
    // 检查分类列表是否被选择
    checkCateList () {
      console.log(this.selectedKeys)
      if (this.selectedKeys.length !== 3) {
        this.$message.warning('请选择商品分类')
        return false
      } else return true
    },
    // 获取选中tabs的序号
    getTabIndex (item) {
      if (this.checkCateList()) {
        this.active = parseInt(item.name)
      }
    },
    // 级联选择器已选中,获取参数信息
    async hasChoosen () {
      const { data: resOnly } = await this.$http.get(`categories/${this.selectedKeys[2]}/attributes`, { params: { sel: 'only' } })
      const { data: resMany } = await this.$http.get(`categories/${this.selectedKeys[2]}/attributes`, { params: { sel: 'many' } })
      // 将参数值转换成数组
      resMany.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
      })
      // 将数据存储到参数数组
      this.manyVals = resMany.data
      this.onlyVals = resOnly.data
      // 将选中id赋值给goods_cat
      this.goodsForm.goods_cat = this.selectedKeys
      console.log(resOnly)
      console.log(resMany)
    },
    // 预览图片
    handlePreview (file) {
      this.imgPreviewDialogVisible = true
      this.curentImgPath = file.url
    },
    // 将图片临时路径存储到对象中
    handleSuccess (res) {
      const newPic = {
        pic: res.data.tmp_path
      }
      this.goodsForm.pics.push(newPic)
    },
    handleRemove (file) {
      const filePath = file.response.data.tmp_path
      const index = this.goodsForm.pics.findIndex(ele =>
        ele.pic === filePath
      )
    },
    // 添加商品
    addGoods () {
      this.$refs.goodsRuleFormRef.validate(async (valid, failValue) => {
        if (!valid) {
          return this.$message.error('请按规范填写表单')
        }
        if (valid) {
          const form = _.cloneDeep(this.goodsForm)
          // 将selectedKeys转换为字符串储存到goods_cat
          console.log(form)
          form.goods_cat = form.goods_cat.join(',')
          this.manyVals.forEach(item => {
            const newInfo = {
              attr_id: item.attr_id,
              attr_value: item.attr_vals.join(' ')
            }
            this.goodsForm.attrs.push(newInfo)
          })
          this.onlyVals.forEach(item => {
            const newInfo = {
              attr_id: item.attr_id,
              attr_value: item.attr_vals
            }
            this.goodsForm.attrs.push(newInfo)
          })
          form.attrs = this.goodsForm.attrs
          // 3.发送数据
          const { data: res } = await this.$http.post('goods', form)
          if (res.meta.status !== 201) {
            return this.$message('添加商品失败')
          }
          this.$message.success('添加商品成功')
          this.active = 5
        }
      })
    }
  }
}
</script>
<style lang="less" scoped>
    .el-steps {
        margin-top: 15px;
    }
    .el-button {
        margin-top: 10px;
    }
</style>
