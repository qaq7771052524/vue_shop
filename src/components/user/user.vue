<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索添加用户区域 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <div class="grid-content bg-purple">
            <el-input
              placeholder="请输入内容"
              v-model="queryInfo.query"
              clearable
              @clear="getList"
            >
              <el-button
                slot="append"
                icon="el-icon-search"
                @click="getList"
              ></el-button>
            </el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="grid-content bg-purple-light">
            <el-button type="primary" @click="addDialog = true"
              >添加用户</el-button
            >
          </div>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="userList" style="width: 100%" stripe border>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名"> </el-table-column>
        <el-table-column prop="mobile" label="电话"> </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="role_name" label="角色"> </el-table-column>
        <el-table-column label="状态" width="70px">
          <template v-slot="datas">
            <el-switch
              v-model="datas.row.mg_state"
              active-color="#13ce66"
              inactive-color="#ff4949"
              @change="userStateChange(datas.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <!-- 作用域插槽 的问题 就是tmeplate 是默认的插槽吗 还是什么  再去看下官方文档-->
          <!-- 只要不是放在带v-slot名字带template名字里内容的 一律看作默认插槽的内容  -->
          <template v-slot="slotProps">
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
            ></el-button>
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
            ></el-button>
            <el-tooltip content="设置角色" placement="top" :enterable="false">
              <el-button
                type="warning"
                size="mini"
                icon="el-icon-setting"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
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
    </el-card>
    <!-- 对话框区域 -->
    <el-dialog title="填写账号" :visible.sync="addDialog" width="50%">
      <el-form
        ref="dialogForm"
        :model="dialogForm"
        label-width="80px"
        :rules="dialogRules"
      >
        <el-form-item label="用户名" prop="name">
          <el-input v-model="dialogForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="dialogForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="dialogForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="dialogForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="addDialog = false">取 消</el-button>
        <el-button type="primary" @click="addDialog = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    var chekckEmail = (rule, value, callback) => {
        const regEmail = /^([A-Za-z0-9_\-\.])+\@([A-Za-z0-9_\-\.])+\.([A-Za-z]{2,4})$/;
        if(regEmail.test(value)) {
            return callback();
        }
        callback(new Error("请输入合法的邮箱"));

    };
    var checkMobile = (rule, value, callback) => {
        const regMobile = /^[1]([3-9])[0-9]{9}$/;
        if(regMobile.test(value)) {
            return callback();
        }
        callback(new Error("请输入合法的手机号"));

    };
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2,
      },
      total: 0,
      userList: [],
      addDialog: false,
      dialogForm: {
        username: '',
        password: '',
        email: '',
        mobile: '',
      },
      dialogRules: {
        name: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 5,
            max: 10,
            message: '长度在 5 到 10 个字符',
            trigger: 'blur',
          },
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '长度在 6 到 15 个字符',
            trigger: 'blur',
          },
        ],
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' },{
            validator:chekckEmail,
            trigger: 'blur'
        }],
        mobile: [
          { required: true, message: '请输入手机', trigger: 'blur' },
          {
            min: 10,
            max: 15,
            message: '长度在 10 到 15 个字符',
            trigger: 'blur',
          },
          {
              validator:checkMobile,
              trigger: 'blur'
          }
        ],
      },
    }
  },
  methods: {
    userStateChange: async function (datas) {
      const { data: res } = await this.$http.put(
        `users/${datas.id}/state/${datas.mg_state}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('更新状态失败！')
        datas.mg_state = !datas.mg_state
      }
      this.$message.success('更新状态成功!')
    },
    handleSizeChange: function (size) {
      console.log(size)
      this.queryInfo.pagesize = size
      this.getList()
    },
    handleCurrentChange: function (current) {
      console.log(current)
      this.queryInfo.pagenum = current
      this.getList()
    },
    getList: async function () {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200) {
        return this.$message.error('管理员列表获取失败')
      }
      this.total = res.data.total
      this.userList = res.data.users
      console.log(res)
    },
  },
  created() {
    this.getList()
  },
}
</script>

<style lang="less" scoped>
.el-table {
  margin-top: 20px;
}
</style>