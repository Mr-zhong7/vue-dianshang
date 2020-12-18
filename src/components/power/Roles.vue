<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoles">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data=roleslsit border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['btnbuttom',i1===0 ? 'btntop' : '','vcenter']" v-for="(item1,i1) in scope.row.children"
              :key="item1.id">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRoles(scope.row,item1.id)">
                  {{item1.authName}}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级权限 -->
              <el-col :span="19">
                <el-row :class="[i2===0 ? '':'btntop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRoles(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable
                      @close="removeRoles(scope.row,item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑
            </el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRolesById(scope.row.id)">删除
            </el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showRightsDialog(scope.row)">分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色数据 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRolesInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色数据 -->
    <el-dialog title="编辑角色" :visible.sync="editDialogVisible" width="50%" @close=editDialogClosed>
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRolesInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限 -->
    <el-dialog title="分配权限" :visible.sync="RightDialogVisible" width="50%" @close="setDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightlist" :props="treeProps" ref="treeRef" show-checkbox node-key="id" default-expand-all
        :default-checked-keys="defkeys"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="RightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="alloRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        roleslsit: [],
        addDialogVisible: false,
        // 添加用户对象
        addForm: {
          roleName: '',
          roleDesc: ''
        },
        // 校验用户规则
        addFormRules: {
          roleName: [{
            required: true,
            message: '请输入角色名称',
            trigger: 'blur'
          }, ],
          roleDesc: [{
            required: true,
            message: '请输入角色描述',
            trigger: 'blur'
          }, ],
        },
        //控制编辑显示与隐藏
        editDialogVisible: false,
        //编辑用户对象
        editForm: {},
        //校验角色验证
        editFormRules: {
          roleName: [{
            required: true,
            message: '请输入角色名称',
            trigger: 'blur'
          }, ],
          roleDesc: [{
            required: true,
            message: '请输入角色描述',
            trigger: 'blur'
          }, ],
        },
        //控制权限显示与隐藏
        RightDialogVisible: false,
        //获取权限列表数据
        rightlist: [],
        //树形控件
        treeProps: {
          children: 'children',
          label: 'authName'
        },
        //默认勾选已有的权限
        defkeys: [],
        //点击分配权限按钮打开对话框获取的当前的id
        rolesId: ''

      };
    },

    created() {
      // 所有角色数据
      this.getRolesList()
    },

    methods: {
      // 获取所有角色数据
      async getRolesList() {
        const {
          data: res
        } = await this.$http.get('roles')

        if (res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败')
        }
        this.roleslsit = res.data
        console.log(this.roleslsit);

      },
      //添加角色
      addRoles() {
        this.addDialogVisible = true
      },
      //监听添加用户关闭事件
      addDialogClosed() {
        this.$refs.addFormRef.resetFields()
      },
      //添加角色数据
      addRolesInfo() {
        this.$refs.addFormRef.validate(async valid => {
          const {
            data: res
          } = await this.$http.post('roles', this.addForm)

          if (res.meta.status !== 201) return this.$message.error("添加角色失败")

          this.addDialogVisible = false
          this.getRolesList()
          this.$message.success("添加角色成功")

        })
      },
      //点击编辑按钮获取角色数据
      async showEditDialog(id) {
        // console.log(id);
        const {
          data: res
        } = await this.$http.get('roles/' + id)

        console.log(res);
        if (res.meta.status !== 200) {
          return this.$message.error("获取角色数据失败")
        }

        this.editForm = res.data
        this.editDialogVisible = true
      },
      // 关闭表单重置
      editDialogClosed() {
        this.$refs.editFormRef.resetFields()
      },
      //编辑角色并提交
      editRolesInfo() {
        this.$refs.editFormRef.validate(async valid => {
          // console.log(valid);
          if (!valid) return
          const {
            data: res
          } = await this.$http.put(`roles/${this.editForm.roleId}`, {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc
          })

          if (res.meta.status !== 200) {
            return this.$message.error("提交角色失败")
          }

          this.getRolesList()
          this.editDialogVisible = false
          this.$message.success("提交角色成功")
        })
      },
      //删除角色数据
      async removeRolesById(id) {
        // console.log(id);
        const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

        if (confirmResult !== "confirm") {
          return this.$message.info("取消删除")
        }
        const {
          data: res
        } = await this.$http.delete('roles/' + id)

        if (res.meta.status !== 200) {
          return this.$message.error("删除角色失败")
        }
        this.$message.success("删除角色成功")
        this.getRolesList()
      },
      //根据Id删除对应的权限
      async removeRoles(role, RightId) {
        const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        // console.log(role.id,RightId);

        if (confirmResult !== 'confirm') {
          return this.$message.info("取消删除")
        }

        const {
          data: res
        } = await this.$http.delete(`roles/${role.id}/rights/${RightId}`)

        if (res.meta.status !== 200) {
          return this.$message.error("删除权限失败")
        }

        // this.getRolesList()如果这样他是会合并起来
        role.children = res.data //看文档是这样说的，返回的data, 是当前角色下最新的权限数据
      },
      //分配权限
      async showRightsDialog(role) {
        this.rolesId = role.id
        const {
          data: res
        } = await this.$http.get('rights/tree')

        if (res.meta.status !== 200) {
          return this.$message.error("获取权限失败")
        }

        this.rightlist = res.data
        console.log(this.rightlist);
        //递归获取三级id
        this.getLeafkeys(role, this.defkeys)
        this.RightDialogVisible = true
      },
      //获取三级权限的id 用递归
      getLeafkeys(node, arr) {
        if (!node.children) {
          return arr.push(node.id)
        }

        node.children.forEach(item =>
          this.getLeafkeys(item, arr)
        )
      },
      //监听重置对话框
      setDialogClosed() {
        this.defkeys = []
      },
      //给角色分配权限
      async alloRights() {
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ]

        const idStr = keys.join(',')

        const {
          data: res
        } = await this.$http.post(`roles/${this.rolesId}/rights`, {
          rids: idStr
        })

        if (res.meta.status !== 200) {
          return this.$message.error("分配权限失败")
        }
        this.$message.success("分配权限成功")
        this.getRolesList()
        this.RightDialogVisible = false
      }
    },

  }

</script>

<style lang="less" scoped>
  .el-tag {
    margin: 7px;
  }

  .btntop {
    border-top: 1px solid #eeeeee;
  }

  .btnbuttom {
    border-bottom: 1px solid #eeeeee;
  }

  .vcenter {
    display: flex;
    align-items: center;
  }

</style>
