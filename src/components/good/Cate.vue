<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <tree-table
        class="treeTable"
        :selection-type="false"
        :expand-type="false"
        :data="cateList"
        :columns="columns"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <template v-slot:isok="slotProps">
          <i
            class="el-icon-error"
            v-if="slotProps.row.cat_deleted == true"
            style="color: red"
          ></i>
          <i class="el-icon-success" v-else style="color: lightGreen"></i>
        </template>
        <template v-slot:sort="slotProps">
          <el-tag v-if="slotProps.row.cat_level == 0" type="success"
            >一级</el-tag
          >
          <el-tag v-else type="warning">二级</el-tag>
        </template>
        <template v-slot:opt="slotProps">
          <el-button size="mini" type="primary" icon="el-icon-edit"
            >编辑</el-button
          >
          <el-button size="mini" type="danger" icon="el-icon-delete"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <el-dialog title="添加分类" :visible.sync="cateDialogVisible" width="50%">
      <el-form
        :model="dialogForm"
        :rules="dialogFormRules"
        ref="dialogFormRef"
        label-width="100px"
        @close="dialogClosed"
      >
        <el-form-item label="活动名称" prop="cate_name">
          <el-input v-model="dialogForm.cate_name" ></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selctedKeys"
            expandTrigger="hover"
            :options="parentCateList"
            :props="cascaderProps"
            @change="handleChange"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="cateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      cateList: [],
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok',
        },
        {
          label: '排序',
          type: 'template',
          template: 'sort',
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt',
        },
      ],
      cateDialogVisible: false,
      dialogForm: {
        cate_name: '',
        cat_pid: 0,
        cat_level: 0,
      },
      dialogFormRules: {
        cate_name: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' },
        ],
      },
      parentCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
      },
      selctedKeys: [],
    }
  },
  methods: {
    getList: async function () {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类信息失败')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getList()
    },
    showDialog() {
      this.getParentCateList()
      this.cateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 2
        },
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级列表失败')
      }
      this.parentCateList = res.data
      console.log(res.data)
    },
    handleChange() {
      if (this.selctedKeys.length > 0) {
        this.dialogForm.cat_pid = this.selctedKeys[this.selctedKeys.length - 1]
        this.dialogForm.cat_level = this.selctedKeys.length
        return
      } else {
        this.dialogForm.cat_pid = 0
        this.dialogForm.cat_level = 0
      }
    },
    dialogClosed: function () {
      this.$refs.dialogFormRef.resetFields()
      this.selctedKeys = []
      this.dialogForm.cat_level = 0
      this.dialogForm.cat_pid = 0
    },
    addCate() {
      this.$refs.dialogFormRef.validate(async (valid) => {
        if (!valid) return this.$message.info("请添加正确格式的分类")
        const { data: res } = await this.$http.post(
          'categories',
          this.dialogForm
        )
        if (res.meta.status !== 201) {
            console.log(this.dialogForm);
            console.log(res.meta.status);
          return this.$message.error('添加分类失败')

        }
        this.getList()
        this.$message.success('添加分类成功')
        this.cateDialogVisible = false
      })
    },
  },
  created() {
    this.getList()
  },
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>