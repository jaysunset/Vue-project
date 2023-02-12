<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" separator=">">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->

      <el-row :gutter="25">
        <el-col :span="9"
          ><el-input
            placeholder="请输入内容"
            clearable
            @clear="getUserList()"
            v-model="queryInfo.query"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList()"
            ></el-button> </el-input
        ></el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userlist" style="width: 100%" border stripe>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="username" label="姓名"> </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="mobile" label="电话"> </el-table-column>
        <el-table-column prop="role_name" label="角色"> </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope"
            ><!--获取当前单元格  -->

            <!-- {{ scope.row.mg_state }}获取当前单元格对应的值 -->
            <el-switch
              v-model="scope.row.mg_state"
              @change="changeMgState(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-tooltip
              class="item"
              effect="dark"
              content="修改"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="primary"
                icon="el-icon-edit"
                size="mini"
                @click="showEditDialog(scope.row.id)"
              ></el-button>
            </el-tooltip>
            <!-- 删除按钮 -->
            <el-tooltip
              class="item"
              effect="dark"
              content="删除"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="danger"
                icon="el-icon-share"
                size="mini"
                @click="removeUserById(scope.row.id)"
              ></el-button>
            </el-tooltip>

            <!-- 分配角色按钮 -->
            <el-tooltip
              class="item"
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
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
        :page-sizes="[2, 4, 6, 8]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加用户的对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="70px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 用户修改对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form
        :model="editForm"
        :rules="editFormFormRules"
        ref="editFormRef"
        label-width="70px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogClosed">取 消</el-button>
        <el-button type="primary" @click="editUserInfo(editForm.id)"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {




    // 定义用户邮箱规则
    var checkEmail = (rule, value, callback) => {
      // 定义邮箱的正则表达式
      let eRgx = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/;
      if (!value) {
        return callback(new Error("邮箱不能为空"))
      }
      setTimeout(() => {

        if (!value.match(eRgx)) {//判断是否合法
          callback(new Error("请输入正确格式的邮箱地址"))
        } else {
          callback()
        }
      })
    }
    //定义用户手机规则
    var checkPhone = (rule, value, callback) => {
      //定义电话的正则表达式
      let pRgx = /^(13[0-9]|14[5|7]|15[0|1|2|3|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$/;
      if (!value) {
        return callback(new Error("电话不能为空"))
      }
      setTimeout(() => {

        if (!value.match(pRgx)) {//判断是否合法
          callback(new Error("请输入正确格式的手机号"))
        } else {
          callback()
        }
      })
    }
    return {
      //获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 4
      },
      userlist: [],
      total: 0,
      //添加用户的表单数据
      addForm: {
        username: "",
        password: '',
        email: '',
        mobile: ''
      },
      // 查询到的用户信息对象
      editForm: {

      },
      // 添加表单的验证规则对象
      addFormRules: {
        username: [{
          required: true,
          message: '请输入正确的用户名',
          trigger: 'blur'
        }, {
          min: 3, max: 10,
          message: '长度在 3 到 10 个字符',
          trigger: 'blur'
        }],
        password: [{
          required: true,
          message: '请输入正确的密码',
          trigger: 'blur'
        }, {
          min: 6, max: 10,
          message: '长度在 6 到 10 个字符',
          trigger: 'blur'
        }],
        email: [{
          required: true,
          message: '请输入正确的邮箱',
          trigger: 'blur'
        }, {
          validator: checkEmail,
          trigger: 'blur'
        }],
        mobile: [{
          required: true,
          message: '请输入正确的手机号',
          trigger: 'blur'
        }, {
          validator: checkPhone,
          trigger: 'blur'
        }]
      },
      //修改用户信息的校验规则
      editFormFormRules: {
        email: [{
          required: true,
          message: '请输入正确的邮箱',
          trigger: 'blur'
        }, {
          validator: checkEmail,
          trigger: 'blur'
        }],
        mobile: [{
          required: true,
          message: '请输入正确的手机号',
          trigger: 'blur'
        }, {
          validator: checkPhone,
          trigger: 'blur'
        }]
      },


      //控制添加对话框的显示与隐藏
      addDialogVisible: false,
      // 控制修改用户对话框的显示与隐藏
      editDialogVisible: false
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      console.log(res);
      if (res.meta.status !== 200) { return this.$message.error("获取用户列表失败") }
      this.userlist = res.data.users;
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    //监听 页码值 改变的事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听switch开关的改变
    async changeMgState(userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`,)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error("修改用户状态失败")
      }
      this.$message.success("更新用户状态成功")
    },
    // 监听添加对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    // 点击按钮 添加用户 valid是一个布尔值 表示校验是否成功
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        // 发送请求 添加该用户
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加用户失败')

        }
        this.$message.success('添加用户成功')
        //隐藏添加的对话框
        this.addDialogVisible = false
        this.getUserList();
      })
    },
    // 展示编辑用户对话框 
    async showEditDialog(id) {
      // this.$refs.addFormRef.resetFields();
      this.editDialogVisible = true;
      const { data: res } = await this.$http.get('users/' + id);
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败!');
      }
      this.editForm = res.data

    },
    // 监听修改用户对话框事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
      this.editDialogVisible = false
    },
    // 修改用户信息并提交
    editUserInfo(id) {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // 发送请求 修改该用户
        const { data: res } = await this.$http.put('users/' +
          id,
          { email: this.editForm.email, mobile: this.editForm.mobile })
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户失败')

        }
        this.$message.success('修改用户成功')
        //隐藏添加的对话框
        this.editDialogVisible = false
        this.getUserList();
      })
    },
    // 根据id删除对应的用户信息
    async removeUserById(id) {
      // 弹窗询问用户是否删除数据
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err);
      // 如果用户确认删除，则返回值为字符串 confirm
      //如果用户取消删除,则返回字符串 cancel
      if (confirmResult !== "confirm") {
        return this.$message.info('已取消删除!')
      }
      // 删除该用户
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败!')
      }
      this.$message.success('删除成功!')
      this.getUserList()
    }
  }
}
</script>

<style>
</style>