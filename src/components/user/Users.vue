<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/'}">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片区域 -->
        <el-card>
            <!-- 搜索与添加区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search"  @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>
            <!-- 用户列表区域 -->
            <el-table :data="userList" border stripe>
                <el-table-column label="#" type="index"></el-table-column>
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态" prop="mg_state">
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <!-- 修改按钮 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="editUserDialog(scope.row.id)"></el-button>
                        <!-- 删除按钮 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserInfo(scope.row.id)"></el-button>
                        <!-- 分配角色按钮 -->
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRolesDialog(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区域 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper"  :total="queryInfo.total"></el-pagination>
        </el-card>
        <!-- 添加用户的提醒对话框 -->
        <el-dialog title="添加用户信息" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <!-- 内容主体区域 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px" >
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部按钮区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改用户的对话框 -->
        <el-dialog title="修改用户信息" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <!-- 内容主体区域 -->
            <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="70px" >
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
            <!-- 底部按钮区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUser()">确 定</el-button>
            </span>
        </el-dialog>

        <!--用户分配角色对话框  -->
        <el-dialog title="分配角色" :visible.sync="setRolesVisible" width="50%" @close="setRolesDialogClose">
            <el-form >
                <div>
                    <p>当前用户：{{userRolesObjInfo.username}}</p>
                    <p>当前角色：{{userRolesObjInfo.role_name}}</p>
                    <p>选择角色：
                        <el-select v-model="selectedValue" placeholder="请选择" @change="selectedValue=item.roleName">
                            <el-option v-for="item in rolesList" :key="item.id" :label="item.roleName" :value="item.id">
                            </el-option>
                        </el-select>
                    </p>
                </div>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRolesVisible=false">取消</el-button>
                <el-button type="primary" @click="saveSetRoles">确定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        // 验证邮箱的规则
        var checkEmail = (rule, value, callback) => {
            // 验证邮箱的正则
            const regEmail = /\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*/
            if (regEmail.test(value)) {
                return callback()
            }
            callback(new Error('请输入合法的邮箱'))
        }
        // 验证手机号的规则
         var checkMobile = (rule, value, callback) => {
            // 验证手机号的正则
            const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|166|17[3678]|18[0-9]|14[57])[0-9]{8}$/
            if (regMobile.test(value)) {
                return callback()
            }
            callback(new Error('请输入合法的手机号'))
        }
        return {
            // 获取用户列表的参数对象
            queryInfo: {
                query: '',
                // 当前的页数
                pagenum: 1,
                pagesize: 2
            },
            userList: [{
                id: 502,
                username: 'root',
                mobile: '12332145665',
                type: 1,
                email: '123Admin@163.com',
                create_time: '2017-11-09T20:36:26.000Z',
                mg_state: true, // 当前用户的状态
                role_name: '超级管理员'
            }],
            total: 0,
            // 控制添加用户的对话框显示与隐藏
            addDialogVisible: false,
             // 控制修改用户的对话框显示与隐藏
            editDialogVisible: false,
            // 添加用户的表单对象
            addForm: {
                username: '',
                password: '',
                email: '',
                mobile: ''
            },
            // 添加表单的验证规则的对象
            addFormRules: {
                username: [
                    { required: true, message: '请输入用户名', trigger: 'blur' },
                    { min: 3, max: 10, message: '用户名长度在3到10之间', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    { min: 6, max: 15, message: '用户名长度在6到15之间', trigger: 'blur' }
                ],
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur' }
                ],
                mobile: [
                    { required: true, message: '请输入手机', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur' }
                ]
            },
            // 被修改的用户信息的数据对象
            editForm: {},
            // 控制分配角色的对话框显示或隐藏
            setRolesVisible: false,
            // 根据id获取的当前用户角色的信息
            userRolesObjInfo: {},
            // 获取的所有的角色的列表
            rolesList: {},
            // 下拉框选中的值
            selectedValue: ''
        }
    },
    created() {
        this.getUserList()
    },
    methods: {
        async getUserList() {
            const { data: res } = await this.$http.get('users', { parmas: this.queryInfo })
            console.log(res)
            if (res.meta.status !== 200) return this.$message.error(res.meta.msg + '; 获取用户列表失败！')
            this.userList = res.data.users
            this.total = res.data.total
        },
        // 监听 newSize 的事件
        handleSizeChange(newSize){
            console.log(newSize)
            this.queryInfo.pagesize = newSize
            this.getUserList()
        },
        // 监听当前页码的事件
         handleCurrentChange(newPage){
             console.log(newPage)
             this.queryInfo.pagenum = newPage
             this.getUserList()
         },
         // 监听 switch 开关状态的改变
         async userStateChanged(userInfo){
             console.log(userInfo)
             const { data: res } = await this.$http.put(
                 `users/${userInfo.id}/state/${userInfo.mg_state}`
             )

             if (res.meta.status !== 200) {
                userInfo.mg_state = !userInfo.mg_state
                return this.$message.error('更新用户状态失败')
             }
             return this.$message.success('更新用户状态成功')
         },
        //  监听对话框关闭的重置事件
         addDialogClosed() {
            //  console.log(this.$refs)
             this.$refs.addFormRef.resetFields()
         },
        //  添加用户并提交
        addUser() {
            // 前的预校验
            this.$refs.addFormRef.validate(async valid => {
                // console.log(valid)
                if (!valid) return false
                // 发起添加用户的网络请求
                const { data: res } = await this.$http.post('users', this.addForm)
                console.log(res)
                if (res.meta.status !== 201) return this.$message.error('添加用户失败')
                this.addDialogVisible = false
                this.getUserList()
                return this.$message.success('添加用户成功')
            })
        },
        // 修改用户信息的对话框
        async editUserDialog(id) {
            this.editDialogVisible = true
            // 根据id 获取用户信息
            const { data: res } = await this.$http.get('users/' + id)
            // 将获取的信息保存到修改对象中
            console.log(res)
            if (res.meta.status !== 200) {
                return this.$message.error('用户信息查询失败')
            }
            this.editForm = res.data
        },
        // 重置修改用户信息的对话框
        editDialogClosed() {
            this.$refs.editFormRef.resetFields()
        },
        // 修改用户信息并提交
        editUser() {
            // 前的预校验
            this.$refs.editFormRef.validate(async valid => {
                if (!valid) return false
                // 验证成功则发起修改用户信息的网络请求
                const { data: res } = await this.$http.put('users/' + this.editForm.id, {
                    email: this.editForm.email,
                    mobile: this.editForm.mobile
                })
                if (res.meta.status !== 200) return this.$message.error('用户信息更新失败')
                // 关闭对话框
                this.editDialogVisible = false
                // 刷新用户列表
                this.getUserList()
                // 提示更新成功
                return this.$message.success('用户更新成功')
            })
        },
        // 根据id删除用户信息
        async removeUserInfo(id) {
            // 询问用户是否删除数据
            const confirmResult = await this.$confirm('此操作将永久删除该用户，是否继续？', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => {
                return err
            })
            console.log(confirmResult)
            // 用户点击取消：cancel，确认：confirm
            if (confirmResult === 'cancel') {
                return this.$message.info('用户取消了删除')
            }
            // 发起删除的网络请求
            const { data: res } = await this.$http.delete('users/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error('删除失败')
            }
            this.getUserList()
            return this.$message.success('删除成功')
        },
        // 打开分配角色的对话框
        async showSetRolesDialog(userObj) {
            // 在对话框打开之前获取所有角色列表
            const { data: res } = await this.$http.get('roles')
             if (res.meta.status !== 200) {
                return this.$message.error('角色列表获取失败')
            }
            this.rolesList = res.data
            this.setRolesVisible = true
            this.userRolesObjInfo = userObj
        },
        // 点击按钮给用户分配角色
        async saveSetRoles() {
            if (!this.selectedValue) {
                return this.$message.error('请选择要分配的角色')
            }
            const { data: res } = await this.$http.put(`users/${this.userRolesObjInfo.id}/role`, { rid: this.selectedValue })
            if (res.meta.status !== 200) {
                return this.$message.error('用户角色分配失败')
            }
            this.$message.success('更新角色成功')
            this.getUserList()
            this.setRolesVisible = false
        },
        // 监听分配角色对话框关闭的动作
        setRolesDialogClose() {
            // 下拉框的值清空
            this.selectedValue = ''
            // 用户名当前角色清空
            this.userRolesObjInfo = {}
        }
    }
}
</script>

<style lang="less" scoped>
</style>