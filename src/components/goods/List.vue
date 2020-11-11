<template>
  <div>
    <!-- 面包屑导航区域 start-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 面包屑导航区域 end-->

    <!-- 卡片视图区 strat -->
    <el-card>
      <!-- 搜索与添加 strat -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getGoodsList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getGoodsList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 搜索与添加 end -->
      <!-- table表格区 strat -->
      <el-table :data="goodslist" border>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称" width="480">
        </el-table-column>
        <el-table-column prop="goods_price" label="商品价格(元)">
        </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量">
        </el-table-column>
        <el-table-column label="创建时间" width="150">
          <template slot-scope="scope">
            {{ scope.row.add_time | dataFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteGoods(scope.row.goods_id)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- table表格区 end -->
      <!-- 分页功能 strat -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 分页功能 end -->
    </el-card>
    <!-- 卡片视图区 end -->
  </div>
</template>

<script>
export default {
  name: 'List',
  data() {
    return {
      //查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10,
      },
      //商品列表
      goodslist: [],
      //总数据条数
      total: 0,
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    //获取商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo,
      })
      if (res.meta.status != 200) {
        return this.$message.error('获取商品列表失败')
      }
      this.total = res.data.total
      this.goodslist = res.data.goods
    },
    //
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize
      this.getGoodsList()
    },
    //
    handleCurrentChange(newnum) {
      this.queryInfo.pagenum = newnum
      this.getGoodsList()
    },
    //删除商品功能
    async deleteGoods(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).catch((err) => err)
      if(confirmResult != 'confirm') {
        return this.$message.error('已取消删除')
      }
      const { data: res } = await this.$http.delete('goods/' + id)
      if (res.meta.status != 200) {
        return this.$message.error('删除商品失败')
      }
      this.$message.success('删除成功')
      this.getGoodsList()
    },
    //
    goAddpage() {
      this.$router.push('/goods/add')
    },
    //
  },
}
</script>

<style lang="less" scoped>
</style>