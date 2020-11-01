<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-table :data="tableList" stripe border>
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="authName" label="权限名称"> </el-table-column>
        <el-table-column prop="path" label="权限路径"> </el-table-column>
        <el-table-column prop="level" label="权限等级">
          <template v-slot='slotProps'>
            <el-tag v-if="slotProps.row.level==0">等级一</el-tag>
            <el-tag type="success" v-else-if="slotProps.row.level==1">等级二</el-tag>
            <el-tag type="warning" v-else>等级三</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableList: []
    }
  },
  created: function () {
    this.getList()
  },
  methods: {
    getList: async function () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message(error('获取列表失败'))
      }
      this.tableList = res.data
      console.log(res.data)
    },
  },
}
</script>

<style lang="less" scoped>
</style>