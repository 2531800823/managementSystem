<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="6">
          <el-input
            placeholder="请输入内容"
            class="input-with-select"
            clearable
            v-model="queryInfo.query"
            @clear="getList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="6"
          ><el-button type="primary" @click="dialogVisible = true"
            >添加用户</el-button
          ></el-col
        >
      </el-row>

      <!-- 用户列表 -->
      <el-table :data="userList" border stripe>
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="getState(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- 修改 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              @click="showEdit(scope.row.id)"
            ></el-button>
            <!-- 删除 -->
            <el-button
              type="warning"
              icon="el-icon-delete"
              @click="editDelete(scope.row.id)"
            ></el-button>
            <!-- 修改状态 -->
            <el-tooltip
              effect="dark"
              content="修改状态"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="danger"
                icon="el-icon-setting"
                @click="updataJuese(scope.row)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>

      <!-- 添加用户对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="dialogVisible"
        width="50%"
        @closed="resetClosed"
      >
        <!-- 表单 -->
        <el-form
          :model="ruleForm"
          :rules="rules"
          ref="ruleFormRef"
          label-width="100px"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="ruleForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="ruleForm.password" type="password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="ruleForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="ruleForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button type="primary" @click="addUser">确 定</el-button>
          <el-button @click="dialogVisible = false">取 消</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户对话框 -->
      <el-dialog
        title="修改用户"
        :visible.sync="editUser"
        width="50%"
        @close="emitClosed"
      >
        <!-- 表单 -->
        <el-form
          :model="editForm"
          :rules="rules"
          ref="editFormRef"
          label-width="70px"
        >
          <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button type="primary" @click="editUserInfo">确 定</el-button>
          <el-button @click="editUser = false">取 消</el-button>
        </span>
      </el-dialog>

      <!-- 修改角色的对话框 -->
      <el-dialog title="分配角色" :visible.sync="updataJue" width="30%">
        <div>
          <p>当前的用户：{{ updataInfo.username }}</p>
          <p>当前的角色：{{ updataInfo.role_name }}</p>
        </div>
        <el-select v-model="selectConent" placeholder="请选择">
          <el-option
            v-for="item in rouleList"
            :key="item.id"
            :label="item.roleName"
            :value="item.id"
          >
          </el-option>
        </el-select>
        <span slot="footer" class="dialog-footer">
          <el-button @click="updataJue = false">取 消</el-button>
          <el-button type="primary" @click="reloInfo">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getList()
  },
  data() {
    // 邮箱验证规则
    var emailCheck = (rule, value, cb) => {
      let exp = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/
      if (exp.test(value)) cb()
      cb(new Error("邮箱格式不正确"))
    }

    // 手机号验证规则
    var mobeilCheck = (rule, value, cb) => {
      let exp =
        /^(13[0-9]|14[5|7]|15[0|1|2|3|4|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$/
      if (exp.test(value)) return cb()
      return cb(new Error("手机号格式不正确"))
    }
    return {
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 2,
      },
      userList: [],
      total: 0,
      // 控制对话框的显示隐藏
      dialogVisible: false,
      //添加用户表单验证
      ruleForm: {
        username: "",
        password: "",
        email: "",
        mobile: "",
      },
      // 添加用户表单验证
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          {
            min: 3,
            max: 10,
            message: "长度在 3 到 10 个字符",
            trigger: "blur",
          },
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          {
            min: 5,
            max: 10,
            message: "长度在 5 到 10 个字符",
            trigger: "blur",
          },
        ],
        email: [
          {
            required: true,
            validator: emailCheck,
            message: "请输入正确的邮箱码",
            trigger: "blur",
          },
        ],
        mobile: [
          {
            required: true,
            validator: mobeilCheck,
            message: "请输入正确的手机号",
            trigger: "blur",
          },
        ],
      },
      // 修改用户对话框
      editUser: false,
      // 查询到的用户信息
      editForm: {},
      // 修改角色的对话框
      updataJue: false,
      // 要修改用户的表单
      updataInfo: {},
      // 角色列表
      rouleList: [],
      selectConent: "",
    }
  },
  methods: {
    async getList() {
      const { data: res } = await this.$http.get("users", {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200) return this.$message.error("获取数据失败")
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 改变 个数 触发
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getList()
    },
    // 改变 页数
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getList()
    },
    // 更新用户状态信息
    async getState(info) {
      const { data: res } = await this.$http.put(
        `users/${info.id}/state/${info.mg_state}`
      )
      if (res.meta.status === 200) return this.$message.success("修改成功")
      info.mg_state = !info.mg_state
      return this.$message.error("修改失败")
    },
    // 监听添加用户对话框关闭
    resetClosed() {
      this.$refs.ruleFormRef.resetFields()
    },
    // 添加用户确定
    addUser() {
      // 预校验
      this.$refs.ruleFormRef.validate(async (val) => {
        if (!val) return
        // 发起用户请求
        // console.log(this.ruleForm)
        const { data: res } = await this.$http.post("users", this.ruleForm)
        // console.log(res)
        if (res.meta.status !== 201) return this.$message.error("添加用户失败")
        this.$message.success("添加成功")
        this.dialogVisible = false
        this.getList()
      })
    },
    // 修改用户弹窗
    async showEdit(id) {
      this.editUser = true
      // 根据 id 获取参数
      const { data: res } = await this.$http.get(`users/${id}`)
      if (res.meta.status !== 200) this.$message.error("请求用户失败")
      this.editForm = res.data
    },
    // 修改用户关闭重置表单
    emitClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 更新用户
    editUserInfo() {
      this.$refs.editFormRef.validate(async (val) => {
        if (!val) return
        const { data: res } = await this.$http.put(
          "users/" + this.editForm.id,
          {
            email: this.editForm.email,
            mobile: this.editForm.mobile,
          }
        )
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.$message.success("更新成功")
        this.editUser = false
      })
    },
    // 删除用户
    async editDelete(id) {
      const confirm = await this.$confirm(
        "此操作将永久删除该用户, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
        // 不过异常并返回 错误对象
      ).catch((err) => err)

      // 点击确定返回confirm
      // 点击取消不过异常 cancel
      if (confirm !== "confirm") return this.$message.info("已取消删除")
      const { data: res } = await this.$http.delete("users/" + id)
      if (res.meta.status !== 200) return this.$message.error("删除失败")
      this.$message.success("删除成功")
      this.getList()
    },
    // 点击修改用户角色
    async updataJuese(user) {
      this.updataJue = true
      this.updataInfo = user
      const { data: res } = await this.$http.get("roles")
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)

      this.rouleList = res.data
    },
    // 提交修改角色
    async reloInfo() {
      if (!this.selectConent) return (this.updataJue = false)

      const { data: res } = await this.$http.put(
        `users/${this.updataInfo.id}/role`,
        {
          rid: this.selectConent,
        }
      )
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success("设置角色成功")
      this.updataJue = false
      this.selectConent = ""
      this.updataInfo = {}
      this.getList()
    },
  },
}
</script>

<style lang="less" secpod>
.el-table {
  margin-top: 20px;
}
</style>