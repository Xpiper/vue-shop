<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="roleList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!-- 渲染一级权限 -->
            <el-row :class="['bdbottom',index===0?'bdtop':'bdbottom','vcenter']"
                    v-for="(item,index) in scope.row.children"
                    :key="item.id">
              <el-col :span="5">
                <el-tag closable @close="deleteUserRight(scope.row,item.id)">{{ item.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <!-- 渲染二级权限 -->
                <el-row v-for="(qtem,qindex) in item.children" :key="qtem.id"
                        :class="[qindex===0?'':'bdtop','vcenter']">
                  <el-col :span="6">
                    <el-tag @close="deleteUserRight(scope.row,qtem.id)" closable type="success">{{ qtem.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable type="warning" v-for="(ctem) in qtem.children" :key="ctem.id"
                            @close="deleteUserRight(scope.row,ctem.id)">{{ ctem.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" circle size="mini"></el-button>
            <el-button type="danger" icon="el-icon-delete" circle size="mini"></el-button>
            <el-tooltip class="item" effect="dark" content="分配权限" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" circle size="mini"
                         @click="showSetRightDialog(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog
      title="提示"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClose">
      <el-tree :data="rightTree"
               :props="rightTreeProps"
               :default-checked-keys="defKeys"
               default-expand-all
               node-key="id"
               show-checkbox
               ref="rightTreeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRight">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Role',
  data () {
    return {
      roleList: [],
      rightTree: [],
      rightTreeProps: {
        label: 'authName',
        children: 'children'
      },
      setRightDialogVisible: false,
      defKeys: [],
      roleId: ''
    }
  },
  methods: {
    async getRoleList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.roleList = res.data
    },
    deleteUserRight (role, rightId) {
      this.$confirm('此操作将永久删除该用户权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        role.children = res.data
        return this.$message.success(res.meta.msg)
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    async showSetRightDialog (role) {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      console.log(role)
      this.rightTree = res.data
      this.getLeafKeys(role, this.defKeys)
      this.roleId = role.id
      this.setRightDialogVisible = true
    },
    getLeafKeys (node, arr) {
      if (!node.children) {
        arr.push(node.id)
      } else {
        node.children.forEach(item => this.getLeafKeys(item, arr))
      }
    },
    setRightDialogClose () {
      this.defKeys = []
    },
    async allotRight () {
      const keys = [
        ...this.$refs.rightTreeRef.getCheckedKeys(),
        ...this.$refs.rightTreeRef.getHalfCheckedKeys()
      ]
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: keys.join(',') })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.getRoleList()
      this.setRightDialogVisible = false
      return this.$message.success(res.meta.msg)
    }
  },
  created () {
    this.getRoleList()
  }
}
</script>

<style scoped lang="less">
  .el-tag {
    margin: 7px;
  }

  .bdtop {
    border-top: 1px solid #eee;
  }

  .bdbottom {
    border-bottom: 1px solid #eee;
  }

  .vcenter {
    display: flex;
    align-items: center;
  }
</style>
