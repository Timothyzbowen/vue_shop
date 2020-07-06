<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区 -->
        <el-card>
            <!-- 警告区 -->
            <el-alert
                title="注意：只允许为第三级分类设置相关参数！"
                type="warning"
                show-icon :closable="false">
            </el-alert>
            <!-- 选择区 -->
            <span>选择商品分类：</span>
            <el-cascader
                expand-trigger='hover'
                v-model="selectedKeys"
                :options="cateList"
                :props="cascaderProps"
                @change="hasChoosen"
                >
            </el-cascader>
            <!-- Tab栏 -->
            <el-tabs v-model="activeName" @tab-click="handleClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="small" :disabled="isdisable" @click="addDialog">添加参数</el-button>
                    <el-table :data="manyParamData" border stripe >
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <!-- 循环渲染标签 -->
                                <el-tag closable v-for="(item,i) in scope.row.attr_vals" :key="i" @close="delTag(i,scope.row)">{{item}}</el-tag>
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    >
                                </el-input>
                                <!-- 输入框和标签切换 -->
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" size="small" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                <el-button type="danger" size="small" icon="el-icon-delete" @click="removeParam(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" size="small" :disabled="isdisable" @click="addDialog">静态属性</el-button>
                    <el-table :data="onlyParamData" border stripe >
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <!-- 循环渲染标签 -->
                                <el-tag closable v-for="(item,i) in scope.row.attr_vals" :key="i" @close="delTag(i,scope.row)">{{item}}</el-tag>
                                <!-- 输入框和标签切换 -->
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    >
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" size="small" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                <el-button type="danger" size="small" icon="el-icon-delete" @click="removeParam(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>
        <!-- 添加对话框 -->
        <el-dialog
            :title="addTitle"
            :visible.sync="addDialogVisible"
            width="50%">
            <el-form :model="addRuleForm" :rules="addRules" ref="addRuleFormRef" label-width="100px" class="demo-ruleForm">
                <el-form-item :label="lableName" prop="attr_name">
                    <el-input v-model="addRuleForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="cancleAdd">取 消</el-button>
                <el-button type="primary" @click="addParam">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改对话框 -->
        <el-dialog
            :title="editTitle"
            :visible.sync="editDialogVisible"
            width="50%">
            <el-form :model="editRuleForm" :rules="editRules" ref="editRuleFormRef" label-width="100px" class="demo-ruleForm">
                <el-form-item :label="lableName" prop="attr_name">
                    <el-input v-model="editRuleForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="cancleAdd">取 消</el-button>
                <el-button type="primary" @click="editParam">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
  created () {
    this.getCateList()
    this.getParamList()
  },
  data () {
    return {
      cateList: [],
      selectedKeys: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      activeName: 'many',
      paramsList: [],
      manyParamData: [],
      onlyParamData: [],
      addDialogVisible: false,
      editDialogVisible: false,
      addRuleForm: {
        attr_name: ''
      },
      addRules: {
        attr_name: { required: true, message: '请输入参数名称', trigger: 'blur' }
      },
      editRuleForm: {
        attr_name: ''
      },
      editRules: {
        attr_name: { required: true, message: '请输入参数名称', trigger: 'blur' }
      }

    }
  },
  computed: {
    isdisable () {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    catId () {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[this.selectedKeys.length - 1]
      }
      return null
    },
    addTitle () {
      if (this.activeName == 'many') {
        return '添加动态参数'
      } else {
        return '添加静态属性'
      }
    },
    editTitle () {
      if (this.activeName == 'many') {
        return '修改动态参数'
      } else {
        return '修改静态属性'
      }
    },
    lableName () {
      if (this.activeName == 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 3 } })
      this.cateList = res.data
    },
    async hasChoosen () {
      this.getParamData()
    },
    async getParamData () {
      const { data: res } = await this.$http.get(`categories/${this.catId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) this.$message.error('获取失败')
      // 处理attr_val
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      // 根据不同情况挂载到不同数组中
      if (this.activeName == 'many') {
        this.manyParamData = res.data
      } else {
        this.onlyParamData = res.data
      }
      console.log(res)
    },
    handleClick () {
      this.getParamData()
      this.$refs.addRuleFormRef.resetFields()
    },
    addDialog () {
      this.addDialogVisible = true
    },
    cancleAdd () {
      this.addDialogVisible = false
      this.$refs.addRuleFormRef.resetFields()
    },
    addParam () {
      this.$refs.addRuleFormRef.validate(async valid => {
        if (!valid) return
        else {
          const { data: res } = await this.$http.post(`categories/${this.catId}/attributes`, { attr_name: this.addRuleForm.attr_name, attr_sel: this.activeName })
          if (res.meta.status !== 201) this.$message.error('添加参数失败')
          this.$message.success('添加参数成功')
          this.addDialogVisible = false
          this.getParamData()
        }
      })
    },
    async showEditDialog (id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get(`categories/${this.catId}/attributes/${id}`, { params: { attr_sel: this.activeName } })
      this.editRuleForm = res.data
    },
    async editParam () {
      const { data: res } = await this.$http.put(`categories/${this.catId}/attributes/${this.editRuleForm.attr_id}`, { attr_name: this.editRuleForm.attr_name, attr_sel: this.activeName })
      if (res.meta.status !== 200) this.$message.error('更新失败')
      this.$message.success('更新成功')
      this.editDialogVisible = false
      this.getParamData()
    },
    async removeParam (id) {
      const confirm = await this.$confirm('此操作将永久删除该值, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirm == 'confirm') {
        const { data: res } = await this.$http.delete(`categories/${this.catId}/attributes/${id}`)
        if (res.meta.status !== 200) return this.$message.error('删除失败')
        this.$message.success('删除成功')
        this.getParamData()
      }
      if (confirm == 'cancel') {
        this.$message('删除取消')
      }
    },
    async handleInputConfirm (row) {
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
    showInput (row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    delTag (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    },
    async saveAttrVals (row) {
      console.log(row.attr_vals)
      const { data: res } = await this.$http.put(`categories/${this.catId}/attributes/${row.attr_id}`, { attr_name: row.attr_name, attr_sel: row.attr_sel, attr_vals: row.attr_vals.join(' ') })
      console.log(row)
    }

  }
}
</script>
<style lang="less" scoped>
    .el-alert {
        margin-bottom: 20px;
    }
    .el-tag {
        margin-left: 20px;
    }
    .el-input {
        width: 110px;
        margin-left: 10px;
    }
    .el-button {
        margin-left: 10px;
    }
</style>
