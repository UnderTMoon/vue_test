<template>
  <div>
    <!--面包屑导航区-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图区域-->
    <el-card>
      <!--搜索与添加区域-->
      <el-row :gutter="20">
        <el-col :span="9">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <!--用户列表区域-->
      <el-table :data="userList" stripe border>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="userName"></el-table-column>
        <el-table-column label="电话" prop="phone"></el-table-column>
        <el-table-column label="性别" prop="sex"></el-table-column>
        <el-table-column label="家庭住址" prop="address"></el-table-column>
        <el-table-column label="身份角色" prop="role"></el-table-column>

        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>

        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!--修改按钮-->
            <el-button type="primary" icon="el-icon-edit"
                       size="mini" @click="showEditDialog(scope.row.userId)"></el-button>
            <!--删除按钮-->
            <el-button type="danger" icon="el-icon-delete" size="mini"
              @click="removeUserById(scope.row.userId)"></el-button>
            <!--分配角色按钮-->
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!--分页区域-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 1]"
        :page-size="queryInfo.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <!--添加用户对话框-->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible"
               width="50%" @close="addDialogClosed">
      <!--内容主体区-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px" >
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="addForm.userName"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="addForm.phone"></el-input>
        </el-form-item>
      </el-form>
      <!--底部区域-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!--修改用户对话框-->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible"
      width="50%" @click="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules"
               ref="editFormRef" label-width="100px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.userName" disabled></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="editForm.phone" ></el-input>
        </el-form-item>
        <el-form-item label="详细地址" prop="address">
          <el-input v-model="editForm.address" ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: 'Users',
    data () {
      //验证邮箱规则
      var checkEmail = (rule,value,cb) => {
        // 验证邮箱正则表达式
        const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
        if(regEmail.test(value)){
          //  合法邮箱
          return cb()
        }
        cb(new Error('请输入合法邮箱'))
      }
      //验证手机号规则
      var checkPhone = (rule,value,cb) => {
        // 验证手机号正则表达式
        const regEmail = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
        if(regEmail.test(value)){
          //  合法手机号
          return cb()
        }
        cb(new Error('请输入正确手机号'))
      }
      return {
        queryInfo: {
          query: '',
          //当前所在页
          pageNo: 1,
          //当前每页显示数据条数
          pageSize: 5
        },
        userList: [],
        total: 0,
        //控制添加用户对话框 显示、隐藏
        addDialogVisible: false,
        //  添加用户的表单数据
        addForm: {
          userName: '',
          password: '',
          phone: ''
        },
        //添加表单验证规则对象
        addFormRules: {
          userName: [
            {
              required: true,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 10,
              message: '用户名长度在3-10个字符间',
              trigger:'blur'
            }
          ],
          password: [
            {
              required: true,
              message: '请输入密码',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 16,
              message: '密码长度在3-16个字符间',
              trigger:'blur'
            }
          ],
          phone: [
            {
              required: true,
              message: '请输入手机号',
              trigger: 'blur'
            },
            {
              validator: checkPhone,
              trigger:'blur'
            }
          ]
        },
      //  控制修改用户对话框显示与隐藏
        editDialogVisible: false,
      //  查询到用户详细信息对象
        editForm: {},
      //  修改表单中的验证规则
        editFormRules: {
          phone:[
            {
              required:true,message:'用输入用户电话',
              trigger:'blur'
            },
            {
              validator:checkPhone,trigger:'blur'
            }
          ],
          address:[
            {
              required:true,message:'用输入用户住址',
              trigger:'blur'
            },
          ]
        }
      }
    },
    created () {
      this.getUserList()
    },
    methods: {
      async getUserList () {
        const { data: res } = await this.$http.post("/users/list", this.queryInfo)
        console.log(res)
        if (res.code != '200') {
          return this.$message.error(res.reason)
        }
        this.userList = res.data
        this.total = res.page.total
      },
      //监听pageSize改变事件
      handleSizeChange (newSize) {
        this.queryInfo.pageSize = newSize
        this.getUserList()
      },
      //监听页码值改变的事件
      handleCurrentChange (newPage) {
        this.queryInfo.pagenum = newPage
        this.getUserList()
      },
      //监听switch 开关状态改变
      async userStateChanged (userInfo) {
        const { data: res } = await this.$http.post('', JSON.stringify(userInfo))
        if (res.code !== '200') {
          userInfo.mg_state = !userInfo.mg_state
          return this.$message.error('更新用户状态失败！')
        }
        this.$message.success('更新用户状态成功！')
      },
    // 监听添加用户对话框关闭事件
      addDialogClosed() {
        this.$refs.addFormRef.resetFields()
      },
    //点击按钮，添加新用户
      addUser() {
        this.$refs.addFormRef.validate(async valid => {
          if(!valid) return
          //可以添加用户
          const {data: res} = await this.$http.post('/users/add', this.addForm)
          if(res.code !== '200'){
            this.$message.error('添加用户失败！')
          }
          this.$message.success('添加用户成功！')
          //添加用户对话框关闭
          this.addDialogVisible = false
        //重新获取用户列表
          this.getUserList()
        })
      },
      //展示编辑用户对话框
      async showEditDialog(id) {
        console.log(id)
        var data = {"userId": id}
        const {data: res} = await this.$http.post('users/detail', data)
        if(res.code !== '200'){
          return this.$message.error('查询用户信息失败！')
        }
        this.editForm = res.data
        this.editDialogVisible = true
      },
    //  监听修改用户对话框关闭事件
      editDialogClosed() {
        this.$refs.editFormRef.resetFields()
      },
    //  修改用户信息并提交
      editUserInfo() {
         this.$refs.editFormRef.validate(async valid => {
          if(!valid) return
        //  发起修改用户信息请求
           const {data:res} = await this.$http.post('users/edit',this.editForm)
           if(res.code !== '200') return this.$message.error('更新用户信息失败！')
           //关闭对话框+刷新数据列表+提示成功
           this.editDialogVisible = false
           this.getUserList()
           this.$message.success('更新用户信息成功！')
        })
      },
    //  根据Id删除用户信息
      async removeUserById(id){
         const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

      //  用户确认删除。返回字段为字符串 confirm
      //  用户取消删除。返回字段为字符串 cancel
        console.log(confirmResult)
        if(confirmResult !== 'confirm'){
          return this.$message.info('已取消删除！')
        }
        var data = {"userId": id}
        const {data:res} = await this.$http.post('users/delete',data)
        if(res.code !== '200') return this.$message.error('删除用户失败！')
        this.$message.success('删除用户成功！')
        this.getUserList()
      }
    }
  }
</script>

<style lang="less" scoped>

</style>
