<template>
  <div>
<!--    面包屑导航区-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
<!--    卡片视图区-->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="9">
          <!--      input输入框-->
          <el-input placeholder="请输入内容" v-model="queryParams.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
<!--      用户列表表格数据渲染-->
      <el-table
              :data="userList"
              stripe
              border
              style="width: 100%">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column
                prop="username"
                label="姓名"
                width="180">
        </el-table-column>
        <el-table-column
                prop="email"
                label="邮箱"
                width="180">
        </el-table-column>
        <el-table-column
                prop="mobile"
                label="电话"
                width="180">
        </el-table-column>
        <el-table-column
                prop="role_name"
                label="角色"
                width="180">
        </el-table-column>
        <el-table-column
                prop="mg_state"
                label="状态"
                width="180">
          <template slot-scope="scope">
            <el-switch
                    @change="userStateChange(scope.row)"
                    v-model="scope.row.mg_state"
                    active-color="#13ce66"
                    inactive-color="#ff4949">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUser(scope.row.id)"></el-button>
              <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
              </el-tooltip>
          </template>
      </el-table-column>
      </el-table>
      <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="queryParams.pagenum"
              :page-sizes="[3, 5, 10, 20]"
              :page-size="queryParams.pagesize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total">
      </el-pagination>
    </el-card>
<!--    添加用户弹出对话框-->
    <el-dialog @close="addDialogClose"
            title="添加用户"
            :visible.sync="addDialogVisible"
            width="50%">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
    </el-dialog>
<!--    修改信息弹出对话框-->
    <el-dialog
            title="修改用户"
            :visible.sync="editDialogVisible"
            width="50%"
    @close="editDialogReset">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
    </el-dialog>
<!--    分配角色弹出对话框-->
    <el-dialog
            title="分配角色"
            :visible.sync="setRoleDialogVisible"
            width="50%" @close="setRoleDialogClosed">
     <div>
       <p>当前用户：{{userInfo.username}}</p>
       <p>当前角色：{{userInfo.role_name}}</p>
       <p>选择角色：
         <template>
           <el-select v-model="selectedRoleId" placeholder="请选择">
             <el-option
                     v-for="item in rolesList"
                     :key="item.id"
                     :label="item.roleName"
                     :value="item.id">
             </el-option>
           </el-select>
         </template>
       </p>
     </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: "User",
    data() {
      // 验证邮箱规则
      var checkEmail = (rule, value, cb) => {
        // 验证邮箱的正则表达式
        const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/

        if (regEmail.test(value)) {
          // 合法的邮箱
          return cb()
        }

        cb(new Error('请输入合法的邮箱'))
      }

      // 验证手机号的规则
      var checkMobile = (rule, value, cb) => {
        // 验证手机号的正则表达式
        const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57]|19[78])[0-9]{8}$/

        if (regMobile.test(value)) {
          return cb()
        }

        cb(new Error('请输入合法的手机号'))
      }
      return {
        queryParams:{
          query: '',
          pagenum: 1,
          pagesize: 3,
        },
        userList: [],
        total: 0,
        addDialogVisible: false,
        addForm: {
          username: '',
          password: '',
          mobile: '',
          email: '',
        },
        addFormRules:{
          username: [
            { required: true, message: '请输入用户名', trigger:'blur' },
            { min:3, max:10, message: '用户名长度在3-10个字符之间', trigger: 'blur'}
          ],
          password: [
            { required: true, message: '请输入密码', trigger:'blur' },
            { min:6, max:15, message: '密码长度在6-15个字符之间', trigger: 'blur'}
          ],
          mobile:[
            { required: true, message: '请输入手机号', trigger:'blur' },
            { validator: checkMobile, trigger: 'blur' }
          ],
          email:[
            { required: true, message: '请输入邮箱', trigger:'blur' },
            { validator: checkEmail, trigger: 'blur' }
          ],
        },
        editDialogVisible: false,
        editForm: {},
        editFormRules: {
          email:[
            { required: true, message: '请输入邮箱', trigger:'blur' },
            { validator: checkEmail, trigger: 'blur' }
          ],
          mobile:[
            { required: true, message: '请输入手机号', trigger:'blur' },
            { validator: checkMobile, trigger: 'blur' }
          ],
        },
        setRoleDialogVisible: false,
        userInfo: {},
        rolesList: [],
        selectedRoleId: '',
      }
    },
    created() {
      this.getUserList()
    },
    methods: {
      async getUserList() {
         const {data: userinfo} = await this.$http.get('users',{
          params: this.queryParams
        })
        if (userinfo.meta.status !== 200)
        {
          return this.$message.error('获取用户列表失败')
        }
        this.userList = userinfo.data.users;
        this.total = userinfo.data.total;
      },
      handleSizeChange(newSize) {
        // console.log(newSize);
        this.queryParams.pagesize = newSize;
        this.getUserList();
      },
      handleCurrentChange(newPage) {
        // console.log(newPage);
        this.queryParams.pagenum = newPage;
        this.getUserList();
      },
      async userStateChange(userinfo) {
        const {data:res} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
        // console.log(res);
        if(res.meta.status !== 200) {
          userinfo.mg_state = !userinfo.mg_state;
          return this.$message.error('更新用户状态失败')
        }
        this.$message.success('用户状态更新成功')
      },
      addDialogClose() {
        this.$refs.addFormRef.resetFields();
      },
      addUser() {
        this.$refs.addFormRef.validate(async valid => {
          if(!valid) return;
          const {data:res} = await this.$http.post('users',this.addForm);
          if(res.meta.status !== 201) {
            this.$message.error(res.meta.msg)
          }else {
            this.$message.success(res.meta.msg);
            this.addDialogVisible = false;
            this.getUserList();
          }
        })
      },
      async showEditDialog(id) {
        const {data:res} = await this.$http.get('users/' + id)
        console.log(res);
        if (res.meta.status !== 200) {
          this.$message.error('查询用户信息失败')
        } else {
          this.editForm = res.data;
        }
        this.editDialogVisible = true;
      },
      editDialogReset() {
        this.$refs.editFormRef.resetFields();
      },
      editUserInfo() {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) return;
          const {data:res} = await this.$http.put('users/' + this.editForm.id,
              {
                email: this.editForm.email,
                mobile: this.editForm.mobile
          });
          if (res.meta.status !== 200){
            return this.$message.error('更新用户信息失败！')
          }
          this.editDialogVisible = false;
          this.getUserList();
          this.$message.success('更新用户信息成功')
        })
      },
      async deleteUser(id) {
        const deleteRes  = await this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if(deleteRes !== 'confirm') {
         return this.$message.info('已取消删除')
        }else {
          const { data:res } = await this.$http.delete('users/' + id);
          if(res.meta.status !== 200) {
            this.$message.error('数据删除失败');
          }else {
            this.$message.success('数据删除成功');
            this.getUserList();
          }
        }

      },
      async setRole(userinfo) {
        this.userInfo = userinfo;
        const {data:res} = await this.$http.get('roles')
        if(res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败')
        }else {
          this.rolesList = res.data;
        }
        this.setRoleDialogVisible = true;
      },
      async saveRoleInfo() {
        if (!this.selectedRoleId) {
          return this.$message.error('请选择要分配的角色！')
        }
        const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`,{
          rid:this.selectedRoleId
        })
        if(res.meta.status !== 200) {
          return this.$message.error('修改角色失败！')
        }else {
          this.$message.success('更新角色成功')
          this.getUserList()
          this.setRoleDialogVisible = false;
        }
      },
      setRoleDialogClosed() {
        this.selectedRoleId  = '';
      }
    },
  }
</script>

<style scoped>

</style>