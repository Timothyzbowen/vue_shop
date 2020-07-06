<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区 -->
        <el-card>
        <!-- 输入框 -->
            <el-row :gutter="20">
                    <el-col :span="9">
                        <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear='getUserlist'>
                            <el-button slot="append" icon="el-icon-search" @click="getUserlist"></el-button>
                        </el-input>
                    </el-col>
                    <el-col :span="2">
                        <el-button type="primary" :span="5" @click="dialogVisible = true">添加用户</el-button>
                    </el-col>
            </el-row>
        <!-- 表格区 -->
            <el-table :data="userlist" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column prop="username" label="姓名"></el-table-column>
                <el-table-column prop="email" label="邮箱"></el-table-column>
                <el-table-column prop="mobile" label="电话"></el-table-column>
                <el-table-column prop="role_name" label="角色"></el-table-column>
                <el-table-column label="状态">
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180px">
                    <!-- 操作区 -->
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUser(scope.row.id)"></el-button>
                        <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRoleDialog(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加用户对话框 -->
        <el-dialog
            title="添加用户"
            :visible.sync="dialogVisible"
            width="50%"
            @close="dialogClosed">
            <el-form :model="addForm" :rules="addFormRules" ref="addRuleFormRef" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password" type="password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible=false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改用户对话框 -->
        <el-dialog
            title="修改用户"
            :visible.sync="editDialogVisible"
            width="50%"
            @close="editDialogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editRuleFormRef" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUser">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 分配角色对话框 -->
        <el-dialog
            title="分配角色"
            :visible.sync="setRoleDialogVisible"
            width="50%"
            @close="editDialogClosed">
            <div>
                <p>当前的用户:{{ roleInfo.username }}</p>
                <p>当前的角色:{{ roleInfo.role_name }}</p>
                <p>分配新角色:
                    <el-select v-model="roleId" placeholder="请选择">
                        <el-option
                        v-for="item in rolelist"
                        :key="item.id"
                        :label="item.roleName"
                        :value="item.id">
                        </el-option>
                    </el-select>
                </p>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRoleDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="setRole">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
  created () {
    this.getUserlist()
  },
  data () {
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        return callback()
      } else {
        callback(new Error('请输入合法邮箱'))
      }
    }
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return callback()
      } else {
        callback(new Error('请输入合法手机号'))
      }
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 4
      },
      userlist: [],
      total: 0,
      dialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [
            		{ required: true, message: '请输入用户名', trigger: 'blur' },
            		{ min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' }
          		],
          		password: [
          			{ required: true, message: '请输入密码', trigger: 'blur' },
            		{ min: 3, max: 12, message: '长度在 3 到 12 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
            		{ validator: checkMobile, trigger: 'blur' }
        ]

      },
      editDialogVisible: false,
      editForm: {
        id: 0,
        username: '',
        email: '',
        mobile: ''
      },
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
            		{ validator: checkMobile, trigger: 'blur' }
        ]
      },
      setRoleDialogVisible: false,
      roleInfo: {},
      rolelist: [],
      roleId: ''

    }
  },
  methods: {
    async getUserlist () {
      const { data: ret } = await this.$http.get('users', { params: this.queryInfo })
      console.log(ret.data)
      this.userlist = ret.data.users
      this.total = ret.data.total
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserlist()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserlist()
    },
    async userStateChange (userinfo) {
      const { data: ret } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (ret.meta.status !== 200) {
        userinfo.mg_status = !userinfo.mg_status
        return this.$message.error('修改失败')
      } else if (ret.meta.status == 200) {
        return this.$message.success('修改成功')
      }
    },
    dialogClosed () {
      this.$refs.addRuleFormRef.resetFields()
    },
    // 添加用户事件
    addUser () {
      // 表单预校验
      this.$refs.addRuleFormRef.validate(async (isSuccess, failValue) => {
        if (!isSuccess) {
          return
        }
        if (isSuccess) {
          const { data: ret } = await this.$http.post('users', this.addForm)
          if (ret.meta.status !== 201) this.$message.error('添加用户失败')
          if (ret.meta.status == 201) {
            this.$message.success('添加用户成功')
            this.dialogVisible = false
            this.getUserlist()
          }
        }
      })
    },
    // 获取用户信息
    async showEditDialog (id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取用户信息失败')
      if (res.meta.status == 200) {
        this.editForm = res.data
      }
    },
    editDialogClosed () {
      this.$refs.editRuleFormRef.resetFields()
    },
    // 修改用户信息
    editUser () {
      this.$refs.editRuleFormRef.validate(async (isSuccess, failValue) => {
        if (!isSuccess) {
          return
        }
        if (isSuccess) {
          const { data: res } = await this.$http.put('users/' + this.editForm.id, this.editForm)
          if (res.meta.status !== 200) this.$message.error('修改用户失败')
          if (res.meta.status == 200) {
            this.$message.success('修改用户成功')
            this.editDialogVisible = false
            this.getUserlist()
          }
        }
      })
    },
    async deleteUser (id) {
      console.log(id)
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) this.$message.error('删除失败')
      if (res.meta.status == 200) {
        this.$message.success('删除成功')
        if (id == this.userlist[this.userlist.length - 1].id) {
          this.queryInfo.pagenum -= 1
        }
        this.getUserlist()
      }
    },
    async showSetRoleDialog (role) {
      this.setRoleDialogVisible = true
      this.roleInfo.id = role.id
      this.roleInfo.username = role.username
      this.roleInfo.role_name = role.role_name
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败')
      this.rolelist = res.data
    },
    async setRole () {
      const { data: res } = await this.$http.put(`users/${this.roleInfo.id}/role`, { rid: this.roleId })
      if (res.meta.status !== 200) return this.$message.error('分配角色失败')
      this.$message.success('分配角色成功')
      this.getUserlist()
      this.setRoleDialogVisible = false
      this.roleId = ''
    }

  }
}
</script>
<style lang="less" scoped>

</style>
