<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <tree-table class="treeTabel" :data="catelist" :columns="columns" show-index index-text="#"
        :show-row-hover="false" border :selection-type="false" :expand-type="false">
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" style="color: green;" v-if="scope.row.cat_deleted===false"></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag size="mini" v-else-if="scope.row.cat_level===1" type="success">二级</el-tag>
          <el-tag size="mini" v-else type="warning">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit" @click="editCateDialog(scope.row.cat_id)">编辑
          </el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete" @click=deleteCateDialog(scope.row.cat_id)>删除
          </el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[3, 5, 10, 15]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader v-model="selectedKeys" :options="parentCateList" :props="cascaderProps"
            @change="parenCateChanged" clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑对话框修改数据 -->
    <el-dialog title="修改分类" :visible.sync="editCateDialogVisible" width="50%" @close="editCateDialogClosed">
      <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCateInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 删除按钮对话框 -->

  </div>
</template>

<script>
  export default {
    data() {
      return {
        //查询条件
        queryInfo: {
          type: 3,
          pagenum: 1,
          pagesize: 5
        },
        // 商品分类
        catelist: [],
        //总数据条数
        total: 0,
        columns: [{
            label: '分类名称',
            prop: 'cat_name',
          },
          //   是否有效
          {
            label: '是否有效',
            type: "template",
            template: 'isok'
          },
          {
            label: '排序',
            type: "template",
            template: 'order'
          },
          {
            label: '操作',
            type: "template",
            template: 'opt'
          }
        ],
        //控制添加分类显示与隐藏
        addCateDialogVisible: false,
        //添加分类表单数据对象
        addCateForm: {
          cat_name: "",
          cat_level: 0,
          cat_pid: 0
        },
        //添加分类表单验证规则
        addCateFormRules: {
          cat_name: [{
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }, ]
        },
        //获取父级分类数据
        parentCateList: [],
        //级联选择器配置对象
        cascaderProps: {
          checkStrictly: true,
          expandTrigger: 'hover',
          value: 'cat_id',
          children: 'children',
          label: 'cat_name'
        },
        //选中项的父级分类
        selectedKeys: [],
        //编辑按钮控制显示与隐藏
        editCateDialogVisible: false,
        //编辑按钮修改表单数据
        editCateForm: {},
        editCateFormRules: {
          cat_name: [{
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }, ]
        }
      };
    },

    created() {
      this.getCateList()
    },

    methods: {
      //获取商品分类数据
      async getCateList() {
        const {
          data: res
        } = await this.$http.get('categories', {
          params: this.queryInfo
        })
        if (res.meta.status !== 200) {
          return this.$message.error("获取商品分类数据失败")
        }
        console.log(res.data);
        this.catelist = res.data.result
        this.total = res.data.total
      },
      //监听条数变化
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getCateList()
      },
      //监听页数
      handleCurrentChange(newPage) {
        this.queryInfo.pagenum = newPage
        this.getCateList()
      },
      //添加分类按钮
      showAddCateDialog() {
        this.getParentCateList()
        this.addCateDialogVisible = true
      },
      //获取父级分类数据
      async getParentCateList() {
        const {
          data: res
        } = await this.$http.get('categories', {
          params: {
            type: 2
          }
        })

        if (res.meta.status !== 200) {
          return this.$message.error("获取父级分类数据失败")
        }

        this.parentCateList = res.data
        console.log(res.data);

      },
      //选中项发生变化触发这个函数
      parenCateChanged() {
        // console.log(this.selectedKeys);
        if (this.selectedKeys.length > 0) {
          this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
          this.addCateForm.cat_level = this.selectedKeys.length
          return
        } else {
          this.addCateForm.cat_pid = 0
          this.addCateForm.cat_level = 0

        }
      },
      //点击对话框确认按钮并提交
      addCate() {
        this.$refs.addCateFormRef.validate(async valid => {
          if (!valid) return

          const {
            data: res
          } = await this.$http.post('categories', this.addCateForm)

          if (res.meta.status !== 201) {
            return this.$message.error("添加分类失败")
          }

          this.$message.success("添加分类成功")
          this.getCateList()
          this.addCateDialogVisible = false
        })

        console.log(this.addCateForm);

      },
      //关闭对话框重置
      addCateDialogClosed() {
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = []
        this.addCateForm.cat_level = 0
        this.addCateForm.cat_pid = 0
      },
      //点击编辑按钮修改分类名称
      async editCateDialog(id) {
        // console.log(id);
        const {
          data: res
        } = await this.$http.get('categories/' + id)

        if (res.meta.status !== 200) {
          return this.$message.error("查询分类名称失败")
        }
        console.log(res);

        this.editCateForm = res.data
        this.editCateDialogVisible = true
      },
      //编辑提交分类
      editCateInfo() {
        this.$refs.editCateFormRef.validate(async valid => {
          if (!valid) return

          const {
            data: res
          } = await this.$http.put("categories/" + this.editCateForm.cat_id, {
            cat_name: this.editCateForm.cat_name
          })

          if (res.meta.status !== 200) {
            return this.$message.error("修改分类失败")
          }

          this.$message.success("修改分类成功")
          this.getCateList()
          this.editCateDialogVisible = false
        })
      },
      //关闭编辑对话框重置
      editCateDialogClosed() {
        this.$refs.editCateFormRef.resetFields()
      },
      //删除按钮
     async deleteCateDialog(id) {
       const confirmResult=await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)

        if(confirmResult!=='confirm'){
          return this.$message.info('取消删除')
        }

        // console.log('确认删除');
        const {data:res}=await this.$http.delete('categories/'+id)
        
        if(res.meta.status!==200){
          return this.$message.error("删除分类失败")
        }
        this.$message.success('删除分类成功')
        this.getCateList()
      }
    },

  }

</script>

<style lang="less" scoped>
  .treeTabel {
    margin-top: 15px;
  }

  .el-cascader {
    width: 100%;
  }

</style>
