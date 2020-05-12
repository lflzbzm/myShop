<template>
  <div>
      <!--面包屑导航-->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
      </el-breadcrumb>
      <!--卡片视图-->
      <el-card>
        <!--按钮-->
        <el-button type="primary">添加角色</el-button>
        <!--表格-->
        <el-table :data="roleList" border stripe>
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-row v-for="(item1, i1) in scope.row.children" :key="item1.id" :class="['bdbottom', i1===0 ?'bdtop':'', 'vcenter']">
                <!--渲染一级权限-->
                <el-col :span="5">
                  <el-tag closable @close="removeRightById(scope.row, item1.id)"> {{item1.authName}} </el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!--渲染二三级权限-->
                <el-col :span="19" >
                  <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="[i2===0 ?'':'bdtop', 'vcenter']">
                    <!--渲染二级权限-->
                    <el-col :span="5">
                      <el-tag closable type="success" @close="removeRightById(scope.row, item2.id)"> {{item2.authName}} </el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!--渲染三级权限-->
                    <el-col :span="19">
                      <el-tag closable @close="removeRightById(scope.row, item3.id)" type="warning"  v-for="(item3) in item2.children" :key="item3.id" > {{item3.authName}} </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <el-table-column type="index"></el-table-column>
          <el-table-column prop="roleName" label="角色名称" width="180"></el-table-column>
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
      <!--分配权限的对话框-->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialog"
        width="50%"
        @close="setRightDialogClosed">
        <!--权限树-->
        <el-tree ref="treeRef" :default-checked-keys="defkeys" default-expand-all show-checkbox node-key="id" :data="rightList" :props="treeProps"></el-tree>
        <span slot="footer">
          <el-button @click="setRightDialog = false">取 消</el-button>
          <el-button type="primary" @click="allotRights"></el-button>
        </span>
      </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      roleList: [],
      setRightDialog: false,
      rightList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defkeys: [],
      // 当前即将分配角色权限的id
      roleId: ''
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    async getRoleList () {
      const { data: res } = await this.$http.get('/roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      console.log(res.data)
      this.roleList = res.data
    },
    async removeRightById (roleInfo, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除！')
      const { data: res } = await this.$http.delete(`roles/${roleInfo.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      this.$message.success('删除权限成功！')
      roleInfo.children = res.data
    },
    async showSetRightDialog (roleInfo) {
      this.roleId = roleInfo.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限树失败！')
      this.rightList = res.data
      // console.log(this.rightList)
      this.getLeafKeys(roleInfo, this.defkeys)
      this.setRightDialog = true
    },
    // 递归函数
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    setRightDialogClosed () {
      this.defkeys = []
    },
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败！')
      this.$message.success('分配权限成功！')
      this.getRoleList()
      this.setRightDialog = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}
.bdtop{
  border-top: solid 1px #eee;
}
.bdbottom{
  border-bottom: solid 1px #eee;
}
.vcenter{
  display: flex;
  align-items: center;
}
</style>
