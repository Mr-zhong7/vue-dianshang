<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert :closable="false" title="注意：只允许第三级分类设置相关参数" type="warning" show-icon>
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cart_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader v-model="selectedCateKeys" :options="catelist" :props="cascaderProps" @change="paramsChanged">
          </el-cascader>
        </el-col>
      </el-row>
      <!-- tab页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 动态参数数据面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button size="mini" type="primary" :disabled="isbtnDisabled" @click="addDialogVisible = true">动态参数
          </el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableDate" border>
            <!-- 展开列 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable @close="handleClosed(i,scope.row)" v-for="(item,i) in scope.row.attr_vals" :key="i">
                  {{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue"
                  ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <!-- 添加按钮 -->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag
                </el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="editShowDialog(scope.row.attr_id)">编辑
                </el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeparams(scope.row.attr_id)">删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button size="mini" type="primary" :disabled="isbtnDisabled" @click="addDialogVisible = true">静态属性
          </el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableDate" border>
            <!-- 展开列 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable @close="handleClosed(i,scope.row)" v-for="(item,i) in scope.row.attr_vals" :key="i">
                  {{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue"
                  ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <!-- 添加按钮 -->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag
                </el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="静态属性" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="editShowDialog(scope.row.attr_id)">编辑
                </el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeparams(scope.row.attr_id)">删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数/属性对话框 -->
    <el-dialog :title="'添加'+titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <!-- 添加参数和静态属性表单数据 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParamsData">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改展示对话框数据 -->
    <el-dialog :title="'添加'+titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <!-- 添加参数和静态属性表单数据 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParamsData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        // 获取商品分类列表数据
        catelist: [],
        //级联选择器配置对象
        cascaderProps: {
          checkStrictly: true,
          expandTrigger: 'hover',
          value: 'cat_id',
          children: 'children',
          label: 'cat_name'
        },
        //选中项的父级分类
        selectedCateKeys: [],
        //当前激活的选项
        activeName: "many",
        //动态参数数据
        manyTableDate: [],
        //静态属性数据
        onlyTableDate: [],
        // 添加对话框参数表单数据
        addDialogVisible: false,
        // 添加表单数据对象
        addForm: {
          attr_name: ''
        },
        //添加表单数据验证规则
        addFormRules: {
          attr_name: [{
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }]
        },
        //修改对话框控制与隐藏
        editDialogVisible: false,
        //修改表单数据
        editForm: {},
        //修改表单数据验证规则
        editFormRules: {
          attr_name: [{
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }]
        },
        // 切换文本框和输入框
        // inputVisible:false,
        // //输入框的数据
        // inputValue: '',
        //不需要这两个了，因为在getParamsList定义了 这两个会导致一个输入变动另一个也变动
      }
    },

    created() {
      this.getCateList()
    },

    methods: {
      // 获取商品分类列表数据
      async getCateList() {
        const {
          data: res
        } = await this.$http.get('categories')

        if (res.meta.status !== 200) {
          return this.$message.error("获取商品分类失败")
        }

        this.catelist = res.data
        console.log(this.catelist);

      },
      //选择项触发这个函数
      paramsChanged() {
        this.getParamsList()

      },
      //   点击激活的选项触发这个函数
      handleTabClick() {
        this.getParamsList()
        console.log(this.activeName);

      },
      //获取三级参数数据
      async getParamsList() {
        if (this.selectedCateKeys.length !== 3) {
          this.selectedCateKeys = []
          this.manyTableDate = []
          this.onlyTableDate = []
          return
        }
        // console.log(this.selectedCateKeys);
        const {
          data: res
        } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: {
            sel: this.activeName
          }
        })
        if (res.meta.status !== 200) {
          return this.$message.error("获取三级参数失败")
        }

        res.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(" ") : []
          //添加两个属性，因为每行数据是不同的，不加一个输入另一个也输入
          item.inputVisible = false
          item.inputValue = ''
        })
        console.log(res.data);
        if (this.activeName === 'many') {
          this.manyTableDate = res.data
        } else {
          this.onlyTableDate = res.data
        }
      },
      //监听添加对话框关闭事件
      addDialogClosed() {
        this.$refs.addFormRef.resetFields()
      },
      //添加参数并提交
      addParamsData() {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) return
          // console.log(valid);

          const {
            data: res
          } = await this.$http.post(`categories/${this.cateId}/attributes`, {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          })

          if (res.meta.status !== 201) {
            return this.$message.error("添加参数失败")
          }

          this.$message.success("添加参数成功")
          this.addDialogVisible = false
          this.getParamsList()
        })
      },
      //修改展示对话框
      async editShowDialog(attr_id) {
        const {
          data: res
        } = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`, {
          params: {
            attr_sel: this.activeName
          }
        })

        if (res.meta.status !== 200) {
          return this.$message.error('查询id失败')
        }
        // this.$message.success("查询id成功")
        this.editForm = res.data
        this.editDialogVisible = true
      },
      //修改确认
      editParamsData() {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) return

          const {
            data: res
          } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          })

          if (res.meta.status !== 200) {
            return this.$message.error("编辑提交失败")
          }

          this.$message.success("编辑提交成功")
          this.editDialogVisible = false
          this.getParamsList()
        })
      },
      //修改关闭重置
      editDialogClosed() {
        this.$refs.editFormRef.resetFields()
      },
      //删除按钮
      async removeparams(attr_id) {
        const confirmRsult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

        if (confirmRsult !== 'confirm') {
          return this.$message.info("已取消删除")
        }
        const {
          data: res
        } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)

        if (res.meta.status !== 200) {
          return this.$message.error('删除参数失败')
        }
        this.$message.success('删除参数成功')
        this.getParamsList()

      },
      //控制文本框和输入框的切换
      async handleInputConfirm(row) {
        if (row.inputValue.trim().length === 0) {
          row.inputValue = ''
          row.inputVisible = false
          return
        }
        // 如果不为零 后续操作
        row.attr_vals.push(row.inputValue.trim())
        row.inputValue = ''
        row.inputVisible = false

        // 发起数据请求保存到服务器上
        this.saveAattrVals(row)

      },
      //展现输入框
      showInput(row) {
        row.inputVisible = true
        this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
      },
      //保存到服务器的数据，共用大家使用
      async saveAattrVals(row) {
        const {
          data: res
        } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(" ")
        })
        if (res.meta.status !== 200) {
          return this.$message.error("添加参数失败")
        }
        this.$message.success("添加参数成功")
      },
      //删除参数
      handleClosed(i, row) {
        row.attr_vals.splice(i, 1)
        this.saveAattrVals(row)
      }
    },


    computed: {
      //显示是动态参数还是静态属性
      isbtnDisabled() {
        if (this.selectedCateKeys.length !== 3) {
          return true
        }
        return false
      },
      //选中的三级id
      cateId() {
        if (this.selectedCateKeys.length === 3) {
          return this.selectedCateKeys[2]
        }
        return null
      },
      titleText() {
        if (this.activeName === "many") {
          return '动态参数'
        } else {
          return '静态属性'
        }
      }
    }
  }

</script>

<style lang="less" scoped>
  .cart_opt {
    margin: 15px 0;
  }

  .el-tag {
    margin: 10px;
  }

  .input-new-tag {
    width: 120px;
  }

</style>
