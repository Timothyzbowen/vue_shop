<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区 -->
        <el-card>
        <!-- 输入框 -->
            <el-row>
                <el-col>
                    <el-button type="primary">添加角色</el-button>
                </el-col>
            </el-row>
        <!-- 表格区 -->
            <el-table :data="rolelist" border stripe>
                <el-table-column type="expand">
                    <!-- 展开区 -->
                    <template slot-scope="scope">
                        <el-row v-for=" (role, index) in scope.row.children" :key="role.id" :class="['bd-bottom', index === 0 ? 'bd-top' : '', 'vcenter']">
                            <!-- 一级区域 -->
                            <el-col :span="6" >
                                <el-tag closable @close="removeRoleById(scope.row, role.id)">{{ role.authName }}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 二级区域 -->
                            <el-col :span="18">
                                    <el-row v-for=" (role1, index) in role.children" :key="role1.id" :class="[index !== role.children.length - 1 ? 'bd-bottom' : '', 'vcenter']">
                                        <el-col :span="6">
                                            <el-tag type="success" closable @close="removeRoleById(scope.row, role1.id)">{{ role1.authName }}</el-tag>
                                            <i class="el-icon-caret-right"></i>
                                        </el-col>
                                        <!-- 三级区域 -->
                                        <el-col :span="18">
                                            <el-tag type="warning" v-for="role2 in role1.children" :key="role2.id" closable @close="removeRoleById(scope.row, role2.id)">{{ role2.authName }}</el-tag>
                                        </el-col>
                                    </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column prop="roleName" label="角色名称"></el-table-column>
                <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
                <el-table-column label="操作" width="350px">
                    <!-- 操作区 -->
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightsDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="50%"
            @close="setRightDialogClose"
            >
            <el-tree :data="rightslist" :props="treeProps" ref="treeRef" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKey"></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRight">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
  created () {
    this.getRolelist()
  },
  data () {
    return {
      rolelist: [],
      rightslist: [],
      setRightDialogVisible: false,
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defKey: [],
      roleid: ''
    }
  },
  methods: {
    async getRolelist () {
      const { data: res } = await this.$http.get('roles')
      console.log(res)
      this.rolelist = res.data
    },
    async removeRoleById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('删除取消')
      } else {
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) return this.$message.info('删除失败')
        if (res.meta.status == 200) {
          this.$message.success('删除成功')
          role.children = res.data
        }
      }
    },
    async showSetRightsDialog (role) {
      this.roleid = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.info('获取权限列表失败')
      this.rightslist = res.data
      this.getdefKey(role, this.defKey)
      console.log(this.defKey)
      this.setRightDialogVisible = true
    },
    getdefKey (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getdefKey(item, arr)
      })
    },
    setRightDialogClose () {
      this.defKey = []
    },
    async allotRight () {
      const ridarr = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const ridstr = ridarr.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleid}/rights`, { rids: ridstr })
      this.getRolelist()
      this.setRightDialogVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
    .el-tag {
        margin-top: 10px;
        margin-bottom: 10px;
        margin-left: 5px;
    }
    .bd-bottom {
        border-bottom: 1px solid #eeeeee;
    }
    .bd-top {
        border-top: 1px solid #eeeeee;
    }
    .vcenter {
        display: flex;
        align-items: center;
    }
</style>
