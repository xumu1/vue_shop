<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-eleme">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card>
      <el-alert title="警告提示的文案" type="warning" :closable="false"></el-alert>
      <el-row class="cat_opt">
        <el-col :span="5">
          <span>选择商品分类</span>
          <!-- 商品分类选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            :props="cateProps"
            @change="handleChange"
            clearable
          ></el-cascader>
        </el-col>
      </el-row>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      catelist: [],
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedCateKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取所有商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取所有商品分类列表失败')
      }
      this.catelist = res.data
      console.log(res.data)
    },
    // 级联选择变化时调用
    handleChange() {
      console.log(this.selectedCateKeys)
    }
  }
}
</script>

// scoped防止组件间的冲突
<style lang="less" scoped>
.cat_opt {
  margin-top: 15px;
}
</style>
