<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 输入框 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" clearable v-model="queryInfo.query" @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <!-- 添加用户 -->
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 用户列表区域 -->
      <el-table :data="userList" stripe border>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)">
            </el-button>
            <!-- 分配角色按钮 -->
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>

          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加用户对话框 -->
    <el-dialog title="提示" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <!-- 主体内容区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="adduser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
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
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      // 验证邮箱规则
      var checkEmail = (rule, value, callback) => {
        const regEmail = /^\w+@\w+(\.\w+)+$/
        if (regEmail.test(value)) {
          return callback()
        }
        callback(new Error('请输入合法的邮箱'))
      }

      //验证手机号规则
      var checkMobile = (rule, value, callback) => {
        const regMobile = /^1[34578]\d{9}$/
        if (regMobile.test(value)) {
          return callback()
        }
        //返回一个错误提示
        callback(new Error('请输入合法的手机号码'))
      }
      return {
        // 获取用户列表数据对象
        queryInfo: {
          query: '',
          // 当前的页数
          pagenum: 1,
          // 当前每页显示多少条数据
          pagesize: 2
        },
        userList: [],
        total: 0,
        // 控制添加对话框显示与隐藏
        addDialogVisible: false,
        // 添加用户表单对象
        addForm: {
          username: '',
          password: '',
          email: '',
          mobile: '',

        },
        //添加用户校验规则
        addFormRules: {
          username: [{
              required: true,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 5,
              message: '长度在 3 到 5 个字符',
              trigger: 'blur'
            }
          ],
          password: [{
              required: true,
              message: '请输入密码',
              trigger: 'blur'
            },
            {
              min: 6,
              max: 15,
              message: '长度在 6 到 15 个字符',
              trigger: 'blur'
            }
          ],
          email: [{
              required: true,
              message: '请输入邮箱',
              trigger: 'blur'
            },
            {
              validator: checkEmail,
              trigger: 'blur'
            }
          ],
          mobile: [{
              required: true,
              message: '请输入手机号',
              trigger: 'blur'
            },
            {
              validator: checkMobile,
              trigger: 'blur'
            }
          ],
        },
        // 控制修改按钮显示与隐藏
        editDialogVisible: false,
        //修改用户数据对象
        editForm: {},
        // 校验修改用户邮箱和手机对象
        editFormRules: {
          email: [{
              required: true,
              message: '请输入用户邮箱',
              trigger: 'blur'
            },
            {
              validator: checkEmail,
              trigger: 'blur'
            }
          ],
          mobile: [{
              required: true,
              message: '请输入用户手机号',
              trigger: 'blur'
            },
            {
              validator: checkMobile,
              trigger: 'blur'
            }
          ],
        }
      };
    },

    created() {
      // 获取用户列表数据
      this.getUserList()
    },

    methods: {
      // 获取用户列表数据
      async getUserList() {
        const {
          data: res
        } = await this.$http.get('users', {
          params: this.queryInfo
        })
        console.log(res);
        if (res.meta.status !== 200) {
          return this.$message.error('获取用户列表失败')
        }
        this.userList = res.data.users
        this.total = res.data.total
      },
      // 监听每页多少条数据
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getUserList()
      },
      //监听当前页面改变的数据
      handleCurrentChange(pageSize) {
        this.queryInfo.pagenum = pageSize
        this.getUserList()
      },
      // 监听开关的改变
      async userStateChange(userInfo) {
        console.log(userInfo);
        const {
          data: res
        } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
        if (res.meta.status !== 200) {
          userInfo.mg_state = !userInfo.mg_state
          return this.$message.error("更新用户状态失败")
        }
        this.$message.success("更新用户状态成功")
      },
      // 监听关闭的重置表单
      addDialogClosed() {
        this.$refs.addFormRef.resetFields()
      },
      // 点击按钮，添加新用户
      adduser() {
        this.$refs.addFormRef.validate(async valid => {
          // console.log(valid);
          if (!valid) return
          const {
            data: res
          } = await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) {
            this.$message.error("添加用户失败")
          }

          this.$message.success("添加用户成功")
          this.addDialogVisible = false
          this.getUserList()
        })
      },
      //修改用户表单数据
      async showEditDialog(id) {
        const {
          data: res
        } = await this.$http.get('users/' + id)
        console.log(res);
        if (res.meta.status !== 200) {
          return this.$message.error("查询用户信息失败！")
        }
        // this.$message.success("修改用户成功")
        this.editForm = res.data
        this.editDialogVisible = true
      },
      //修改用户重置事件
      editDialogClosed() {
        this.$refs.editFormRef.resetFields()
      },
      //修改用户信息并提交
      editUserInfo() {
        this.$refs.editFormRef.validate(async valid => {
          // console.log(config);
          if (!valid) return
          const {
            data: res
          } = await this.$http.put(`users/${this.editForm.id}`, {
            email: this.editForm.email,
            mobile: this.editForm.mobile
          })

          if (res.meta.status !== 200) {
            return this.$message.error("修改用户信息失败")
          }
          this.editDialogVisible = false
          this.getUserList()
          this.$message.success("修改用户成功")
        })
      },
      // 删除用户
      async removeUserById(id) {
        const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        // console.log(confirmResult);
        
        if (confirmResult !== 'confirm') {
          return this.$message.info('取消删除')
        }

        // console.log('确认删除');
       const {data:res} =await this.$http.delete('users/'+id)

       if(res.meta.status!==200){
         return this.$message.error("删除用户失败")
       }

       this.$message.success('删除用户成功')
       this.getUserList()

      }
    },

  }

</script>

<style lang="less" scoped>

</style>
