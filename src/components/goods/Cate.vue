<template>
  <div>
    <!-- 面包屑导航区域 start-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 面包屑导航区域 end-->
    <el-card>
      <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
      <!-- 商品分类数据列表列表 start -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :expand-type="false"
        :selection-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
        class="Treetable"
      >
        <!-- 是否有效列 -->
        <template slot="isok" slot-scope="scope">
          <i
            v-if="scope.row.cat_deleted === false"
            class="el-icon-success"
            style="color: lightGreen"
          ></i>
          <i v-else class="el-icon-error" style="color: red"></i>
        </template>
        <!-- 排序列 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag
            v-else-if="scope.row.cat_level === 1"
            type="success"
            size="mini"
            >二级</el-tag
          >
          <el-tag v-else type="warning" size="mini">三级</el-tag>
        </template>
        <!-- 操作列 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button type="primary" icon="el-icon-delete" size="mini"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 商品分类数据列表 end -->
      <!-- 添加分类对话框 start -->
      <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%"
        @close="addCateDialogClosed"
      >
        <!-- 添加分类主体 -->
        <el-form
          :model="addCateForm"
          :rules="addCaterules"
          ref="addCateDialogRef"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="分类分级">
            <el-cascader
              v-model="seletedKeys"
              :options="parentCateList"
              :props="cascaderProps"
              @change="parentCateChanged"
              clearable
            ></el-cascader>
          </el-form-item>
        </el-form>
        <!-- 添加分类底部 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 添加分类对话框 end -->
      <!-- 分页 start -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 3, 4]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 分页 end -->
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Cate',
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      total: 0,
      //商品分类数据列表
      cateList: [],
      //为tree-table指定列
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
        },
        {
          label: '是否有效',
          type: 'template',
          //当前列使用的模版
          template: 'isok',
        },
        {
          label: '排序',
          type: 'template',
          //当前列使用的模版
          template: 'order',
        },
        {
          label: '操作',
          type: 'template',
          //当前列使用的模版
          template: 'opt',
        },
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0,
      },
      addCaterules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
        ],
      },
      //父级分类的列表
      parentCateList: [],
      //指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        lable: 'cat_name',
        children: 'children'
      },
      //选中的父级分类的ID数组
      seletedKeys: [],
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取商品分类数据列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200) {
        return thi.$message.error('获取商品分类列表失败')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    //监听pagesize改变事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    //监听pagenum改变事件
    handleCurrentChange(newNum) {
      this.queryInfo.pagenum = newNum
      this.getCateList()
    },
    //获取父级分类的数据列表
    async getPramentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 },
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败')
      }
      this.parentCateList = res.data
    },
    //选择项发生变化
    parentCateChanged() {
      if(this.seletedKeys.length > 0) {
        this.addCateForm.cat_pid = this.seletedKeys[this.seletedKeys.length - 1]
        this.addCateForm.cat_level = this.seletedKeys.length
        return
      } else {
        this.addCateForm.cat_pid = 0,
        this.addCateForm.cat_level = 0
      }
    },
    //添加分类对话框显示
    showAddCateDialog() {
      this.getPramentCateList()
      this.addCateDialogVisible = true
    },
    //添加分类对话框关闭重置
    addCateDialogClosed() {
      this.$refs.addCateDialogRef.resetFields()
      
    },
    //添加分类提交
    addCate() {
      this.$refs.addCateDialogRef.validate(async valid => {
        if(!valid) return
        const {data: res} = await this.$http.post('categories', this.addCateForm)
        if(res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })

    },
  },
}
</script>

<style lang="less" scoped>
.Treetable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>