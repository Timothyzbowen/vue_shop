<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区 -->
        <el-card>
            <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
            <!-- 树形图区域 -->
            <zk-table
                ref="table"
                index-text="#"
                :data="cateList"
                :columns="columns"
                :border="props.border"
                :show-header="props.showHeader"
                :show-row-hover="props.showRowHover"
                :show-index="props.showIndex"
                :tree-type="props.treeType"
                :is-fold="props.isFold"
                :expand-type="props.expandType"
                :selection-type="props.selectionType">
                <template slot="isvalidate" slot-scope="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted == false" style="color:lightgreen"></i>
                    <i class="el-icon-error" style="color:red" v-else></i>
                </template>
                <template slot="order" slot-scope="scope">
                    <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag v-if="scope.row.cat_level === 1" type="success">二级</el-tag>
                    <el-tag v-if="scope.row.cat_level === 2" type="warning">三级</el-tag>
                </template>
                <template slot="set" slot-scope="scope">
                    <el-button type="primary" size="mini">编辑</el-button>
                    <el-button type="danger" size="mini">删除</el-button>
                </template>
            </zk-table>
            <!-- 分页区 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[5, 10, 20, 30]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total=total>
            </el-pagination>
            <!-- 添加分类对话框 -->
            <el-dialog
                title="添加分类"
                :visible.sync="addCateDialogVisible"
                width="50%"
                >
                <!-- 输入内容区 -->
                <el-form :model="addCateForm" :rules="CateFormRules" ref="CateFormRef" label-width="100px" >
                    <el-form-item label="分类名称" prop="cat_name" required>
                        <el-input v-model="addCateForm.cat_name"></el-input>
                    </el-form-item>
                    <el-form-item label="父级分类">
                        <el-cascader
                            expand-trigger='hover'
                            v-model="selectedKeys"
                            :options="parentCateList"
                            :props="cascaderProps"
                            clearable
                            @change="parentCateChanged">
                        </el-cascader>
                </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="resetForm">取 消</el-button>
                    <el-button type="primary" @click="addCate">确 定</el-button>
                </span>
            </el-dialog>
        </el-card>
    </div>
</template>
<script>
export default {
  created () {
    this.getCateList()
  },
  data () {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      props: {
        stripe: false,
        border: true,
        showHeader: true,
        showRowHover: false,
        showIndex: true,
        treeType: true,
        isFold: true,
        expandType: false,
        selectionType: false
      },
      cateList: [],
      total: 0,
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      CateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 获取一二级分类列表
      parentCateList: [],
      // 选中的二级分类的数组
      selectedKeys: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isvalidate'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'set'
        }
      ]
    }
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getCateList()
    },
    showAddCateDialog () {
      this.addCateDialogVisible = true
      this.getParentCateList()
    },
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      this.parentCateList = res.data
    },
    // 级联选择器选择过后
    parentCateChanged () {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_level = this.selectedKeys.length
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
      } else {
        this.addCateForm.cat_level = 0
        this.addCateForm.cat_pid = 0
      }
    },
    // 确定增加分类
    async addCate () {
      this.$refs.CateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status === 201) this.$message.success('添加成功')
        if (res.meta.status !== 201) this.$message.error('添加失败')
        this.addCateDialogVisible = false
        this.getCateList()
      }
      )
    },
    resetForm () {
      this.$refs.CateFormRef.resetFields()
      this.addCateDialogVisible = false
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }

  }
}
</script>
<style lang="less" scoped>
    .zk-table {
        margin-top: 10px;
    }
    .el-cascader {
        width: 100%;
    }
</style>
