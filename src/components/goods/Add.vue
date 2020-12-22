<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert title="添加商品信息" type="info" center show-icon :closable="false">
      </el-alert>

      <!-- 步骤条 -->
      <el-steps :space="200" :active="activeIndex-0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tab栏区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
        <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTableLeave"
          @tab-click="tableClicked">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input type="number" v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input type="number" v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input type="number" v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader v-model="addForm.goods_cat" :options="catelist" :props="cascaderProps"
                @change="handleChange"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item v-for="item in manyTableDate" :key="item.attr_id" :label="item.attr_name">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox border v-for="(cb,i) in item.attr_vals" :key="i" :label="cb"></el-checkbox>
              </el-checkbox-group>
            </el-form-item>

          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item v-for="item1 in onlyTableDate" :key="item1.attr_id" :label="item1.attr_name">
              <el-input v-model="item1.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload :action="uploadUrL" :headers="headersObj" :on-preview="handlePreview" :on-remove="handleRemove"
              list-type="picture" :on-success="handleSuceess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <!-- 添加商品 -->
            <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 处理预览图片 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" alt="" class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
  // 深拷贝导入
  import _ from "lodash"
  export default {
    data() {
      return {
        // 当前完成的步骤条
        activeIndex: '0',
        // 添加表单对象数据
        addForm: {
          goods_name: '',
          goods_price: 0,
          goods_number: 0,
          goods_weight: 0,
          // 商品分类所属的数组
          goods_cat: [],
          // 图片
          pics: [],
          // 介绍
          goods_introduce: '',
          attrs: []
        },
        addFormRules: {
          goods_name: [{
            required: true,
            message: '请输入活动名称',
            trigger: 'blur'
          }, ],
          goods_price: [{
            required: true,
            message: '请输入活动名称',
            trigger: 'blur'
          }, ],
          goods_number: [{
            required: true,
            message: '请输入活动名称',
            trigger: 'blur'
          }, ],
          goods_weight: [{
            required: true,
            message: '请输入活动名称',
            trigger: 'blur'
          }, ],
          goods_cat: [{
            required: true,
            message: '请输入活动名称',
            trigger: 'blur'
          }, ],
        },
        //商品分类数据
        catelist: [],
        //级联选择器配置对象
        cascaderProps: {
          checkStrictly: true,
          expandTrigger: 'hover',
          value: 'cat_id',
          children: 'children',
          label: 'cat_name'
        },
        // 动态参数数据
        manyTableDate: [],
        //静态属性数据列表
        onlyTableDate: [],
        uploadUrL: 'http://127.0.0.1:8888/api/private/v1/upload',
        // 设置上传图片头
        headersObj: {
          Authorization: window.sessionStorage.getItem('token')
        },
        // 图片预览
        previewPath: '',
        // 图片显示与隐藏
        previewVisible: false
      };
    },

    created() {
      this.getCateList()
    },

    methods: {
      async getCateList() {
        const {
          data: res
        } = await this.$http.get('categories')

        if (res.meta.status !== 200) {
          return this.$message.error('获取商品分类失败')
        }
        this.catelist = res.data
        console.log(this.catelist);

      },
      // 已选中项触发这个函数
      handleChange() {
        if (this.addForm.goods_cat.length !== 3) {
          this.addForm.goods_cat = []
        }
        console.log(this.addForm.goods_cat);

      },
      // 选择商品分类才切换下一个
      beforeTableLeave(activeName, oldActiveName) {

        // console.log("离开的"+oldActiveName  );
        // console.log("进入的"+activeName  );

        if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
          this.$message.error("请先选择商品分类")
          return false
        }
      },
      //获取动态参数
      async tableClicked() {
        if (this.activeIndex === '1') {
          const {
            data: res
          } = await this.$http.get(`categories/${this.cateId}/attributes`, {
            params: {
              sel: "many"
            }
          })
          if (res.meta.status !== 200) {
            return this.$message.error("获取动态参数失败")
          }
          res.data.forEach(item => {
            item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
          })

          this.manyTableDate = res.data
          console.log(res.data);
        } else if (this.activeIndex === '2') {
          const {
            data: res
          } = await this.$http.get(`categories/${this.cateId}/attributes`, {
            params: {
              sel: "only"
            }
          })
          if (res.meta.status !== 200) {
            return this.$message.error("获取静态属性失败")
          }
          this.onlyTableDate = res.data
          console.log(res.data);

        }
      },
      // 图片预览
      handlePreview(file) {
        this.previewPath = file.response.data.url
        this.previewVisible = true
      },
      // 删除图片啊
      handleRemove(file) {
        // console.log(file);
        const filePath = file.response.data.tmp_path
        const i = this.addForm.pics.findIndex(x => {
          x.pic === filePath
        })

        this.addForm.pics.splice(i, 1)
        // console.log(this.addForm);

      },
      // 保存图片
      handleSuceess(response) {
        // console.log(response);
        const picsInfo = {
          pic: response.data.tmp_path
        }
        this.addForm.pics.push(picsInfo)
        // console.log(this.addForm);

      },
      // 添加商品
      add() {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) {
            return this.$message.error("请填写必要的表单")
          }
          // console.log(this.addForm);
          // 深拷贝安装lodash 的一个方法cloneDeep
          const form = _.cloneDeep(this.addForm)

          form.goods_cat = form.goods_cat.join(",")
          // console.log(form);
          // 处理动态参数
          this.manyTableDate.forEach(item => {
            const newInfo = {
              attr_id: item.attr_id,
              attr_value: item.attr_vals.join(' ')
            }
            this.addForm.attrs.push(newInfo)
          })

          // 处理静态属性
          this.onlyTableDate.forEach(item => {
            const newInfo = {
              attr_id: item.attr_id,
              attr_value: item.attr_vals
            }
            this.addForm.attrs.push(newInfo)
          })

          form.attrs = this.addForm.attrs
          console.log(form);
          // 发起请求
          const {
            data: res
          } = await this.$http.post('goods', form)
          if (res.meta.status !== 201) {
            return this.$message.error("添加商品失败")
          }
          this.$message.success('添加商品成功')
          this.$router.push('/goods')
        })

      }
    },
    computed: {
      cateId() {
        if (this.addForm.goods_cat.length === 3) {
          return this.addForm.goods_cat[2]
        }
        return null
      }
    }

  }

</script>

<style lange="less" scoped>
  .el-checkbox {
    margin: 0 10px 0 0 !important;
  }

  .previewImg {
    width: 100%;
  }

  .btnAdd {
    margin-top: 15px;
  }

</style>
