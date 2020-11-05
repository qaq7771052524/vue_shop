<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-alert
        class="alert_c"
        title="注意：只允许第三级分类设置参数"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <el-row>
        <el-col>
          <span
            >选择商品分类:
            <el-cascader
              v-model="selectedKeys"
              :options="cateList"
              expandTrigger="hover"
              :props="casProps"
              @change="handleCasChange"
            ></el-cascader>
          </span>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="dialogVisible = true"
            >添加参数</el-button
          >
          <el-table :data="manyTabDate" border stripe>
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <el-button size="mini" class="el-icon-edit" type="primary"
                >编辑</el-button
              >
              <el-button size="mini" class="el-icon-delete" type="danger"
                >删除</el-button
              >
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态参数" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="dialogVisible = true"
            >添加参数</el-button
          >
          <el-table :data="onlyTabDate" border stripe>
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template v-slot="slotProps">
                <el-button size="mini" class="el-icon-edit" type="primary"
                  >编辑</el-button
                >
                <el-button size="mini" class="el-icon-delete" type="danger"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <el-dialog
      :title="'添加' + isText"
      :visible.sync="dialogVisible"
      width="50%"
      @close="closeDialog"
    >
      <el-form
        :model="dialogForm"
        :rules="dialogFormRules"
        ref="dialogFormRef"
        label-width="100px"
      >
        <el-form-item :label="isText" prop="attr_name">
          <el-input v-model="dialogForm.attr_name"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="hadleAddParams"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cateList: [],
      selectedKeys: [],
      casProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
      },
      activeName: 'many',
      manyTabDate: [],
      onlyTabDate: [],
      dialogVisible: false,
      dialogForm: {
          attr_name:''
      },
      dialogFormRules:{
          attr_name: [{
              required:true,
              message:"请输入名字",
              trigger:'blur'
          }]
      }
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status != 200) {
        return this.$message.error('获取分类列表失败')
      }
      this.cateList = res.data
      console.log(res.data)
    },
    async handleCasChange() {
      this.getParamsDate()
    },
    handleTabClick() {
      this.getParamsDate()
    },
    async getParamsDate() {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        return
      }
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName },
        }
      )
      if (res.meta.status !== 200) {
        this.$message.error('获取动静态参数失败')
      }
      if (this.activeName == 'many') {
        this.manyTabDate = res.data
      } else {
        this.onlyTabDate = res.data
      }
      console.log(res.data)
    },
    closeDialog() {
        this.$refs.dialogFormRef.resetFields()
    },
    hadleAddParams() {
        this.$refs.dialogFormRef.validate(async (valid) => {
            if(!valid) 
            return this.$message.error("参数验证失败")
            const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`, {
                attr_name:this.dialogForm.attr_name,
                attr_sel:this.activeName
            })
            if(res.meta.status!==201) {
                return this.$message.error("提交参数失败")
            }
            this.dialogVisible = false
            this.getParamsDate()
        })
    }
  },
  created() {
    this.getCateList()
  },
  computed: {
    isBtnDisabled() {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    cateId() {
      if (this.selectedKeys.length == 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    isText() {
      if (this.activeName == 'many') {
        return '动态参数'
      } else {
        return '静态参数'
      }
    },
  },
}
</script>

<style lang="less" scoped>
.alert_c {
  margin: 16px 0;
}
</style>