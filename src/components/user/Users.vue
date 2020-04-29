<template>
  <div>
      <!--面包屑导航-->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
      </el-breadcrumb>
      <!--卡片视图-->
      <el-card>
          <!--搜索与添加区域-->
          <el-row :gutter="20">
              <!--搜索框-->
              <el-col :span="8">
                  <el-input clearable @clear="getUserList" v-model="queryInfo.query" placeholder="请输入内容">
                      <el-button @click="getUserList" slot="append" icon="el-icon-search"></el-button>
                  </el-input>
              </el-col>
              <!--添加按钮-->
              <el-col :span="16">
                  <el-button type="primary" @click="openAddUserDialog">添加</el-button>
              </el-col>
          </el-row>
          <!--表格-->
          <el-table :data="userList" border stripe>
              <el-table-column type="index"></el-table-column>
              <el-table-column prop="username" label="姓名">
              </el-table-column>
              <el-table-column prop="email" label="邮箱">
              </el-table-column>
              <el-table-column prop="mobile" label="电话">
              </el-table-column>
              <el-table-column prop="role_name" label="角色">
              </el-table-column>
              <el-table-column label="状态">
                  <template slot-scope="scope">
                      <el-switch v-model="scope.row.mg_state" @change="listenState(scope.row)"></el-switch>
                  </template>
              </el-table-column>
              <el-table-column label="操作" width="200px">
                  <template slot-scope="scope">
                      <el-button size="mini" type="primary" icon="el-icon-edit" @click="openEditDialog(scope.row.id)"></el-button>
                      <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeUser(scope.row.id)"></el-button>
                       <el-tooltip :enterable="false" effect="dark" content="分配角色" placement="top">
                           <el-button size="mini" type="warning" icon="el-icon-location"></el-button>
                       </el-tooltip>
                  </template>
              </el-table-column>
          </el-table>
          <!--分页-->
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[3, 5, 10, 20]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
          </el-pagination>
      </el-card>
      <!--实现添加功能的对话框-->
      <el-dialog
        title="添加用户"
        :visible.sync="addUserDialog"
        width="50%"
        @close="addUserFormClosed">
        <!--表单-->
        <el-form :model="addUserForm" :rules="addUserFormRules" ref="addUserFormRef" label-width="70px">
          <el-form-item label="姓名" prop="username">
            <el-input v-model="addUserForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addUserForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addUserForm.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="addUserForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!--底部-->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addUserDialog = false">取 消</el-button>
          <el-button type="primary" @click="addConfirm">确 定</el-button>
        </span>
      </el-dialog>
      <!--实现编辑功能的对话框-->
      <el-dialog
        title="编辑信息"
        :visible.sync="editDialog"
        width="50%"
        @close="editDialogClosed">
        <!--表单-->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
          <el-form-item label="姓名">
            <el-input disabled v-model="editForm.username"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!--底部-->
        <span slot="footer">
          <el-button @click="editDialog = false">取 消</el-button>
          <el-button type="primary" @click="editConfirm(editForm.id)">确 定</el-button>
        </span>
      </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^([0-9a-zA-Z-_])+@([0-9a-zA-Z-_])+(\.[0-9a-zA-Z-_])/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('邮箱格式不正确'))
    }
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1[3|4|5|7|8][0-9]{9}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('手机格式不正确'))
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 3
      },
      userList: [],
      total: 0,
      addUserDialog: false,
      addUserForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addUserFormRules: {
        username: [
          { required: true, message: '请输入用户姓名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
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
      editDialog: false,
      editForm: {
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
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取用户列表数据失败')
      console.log(res.data)
      this.total = res.data.total
      this.userList = res.data.users
    },
    handleSizeChange (newsize) {
      this.queryInfo.pagesize = newsize
      this.getUserList()
    },
    handleCurrentChange (newpagenum) {
      this.queryInfo.pagenum = newpagenum
      this.getUserList()
    },
    async listenState (userInfo) {
      console.log(userInfo.mg_state)
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (res.meta.status !== 200) return this.$message.error('修改状态失败')
      this.$message.success('状态修改成功')
    },
    openAddUserDialog () {
      this.addUserDialog = true
    },
    addUserFormClosed () {
      this.$refs.addUserFormRef.resetFields()
    },
    addConfirm () {
      this.$refs.addUserFormRef.validate(async (valid) => {
        console.log(valid)
        if (!valid) return this.$message.error('表单信息错误')
        const { data: res } = await this.$http.post('users', this.addUserForm)
        console.log(res)
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.getUserList()
        this.addUserDialog = false
      })
    },
    async openEditDialog (id) {
      const { data: res } = await this.$http.get(`users/${id}`)
      if (res.meta.status !== 200) return this.$message.error('读取用户信息失败')
      this.editForm = res.data
      this.editDialog = true
    },
    editConfirm (id) {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return this.$message.error('表单信息错误')
        const { data: res } = await this.$http.put(`users/${id}`, this.editForm)
        if (res.meta.status !== 200) return this.$message.error('修改失败')
        this.$message.success('修改成功')
        this.getUserList()
        this.editDialog = false
      })
    },
    editDialogClosed () {
      this.$refs.editFormRef.reserFields()
    },
    async removeUser (id) {
      const confirmResult = await this.$confirm('确定删除吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete(`users/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getUserList()
    }
  }
}
</script>

<style lang="less" scoped>
.el-card{
    margin-top: 20px;
}
</style>
