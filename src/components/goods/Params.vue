<template>
    <div>
             <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/'}">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <el-alert show-icon title="警告⚠只允许为第三级分类设置参数" type="warning" :closable="false"></el-alert>

            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <!-- 选择商品分类的级联选择框 -->
                        <el-cascader expand-trigger="hover" :options="cateList" :props="cateProps" v-model="selectedCateKeys" @change="handleChange"></el-cascader>
                </el-col>
            </el-row>

            <!-- tab 页签区域 -->
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <el-tab-pane label="动态参数" name="many">
                    <!-- 添加参数的按钮 -->
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
                    <!-- 动态参数表格 -->
                    <el-table :data="manyTaleData" border stripe>
                        <!-- 展开行 -->
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">{{item}}</el-tag>
                                <!-- 输入的文本框 -->
                                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                                </el-input>

                                <!-- 添加按钮 -->
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                         <el-table-column label="属性值" prop="attr_vals"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                                <el-button type="danger" size="mini" icon="el-icon-edit" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <!-- 添加属性的按钮 -->
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加属性</el-button>
                       <!-- 静态属性参数表格 -->
                    <el-table :data="onlyTaleData" border stripe>
                         <!-- 展开行 -->
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">{{item}}</el-tag>
                                <!-- 输入的文本框 -->
                                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                                </el-input>

                                <!-- 添加按钮 -->
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="属性值" prop="attr_vals"></el-table-column>
                             <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                                <el-button type="danger" size="mini" icon="el-icon-edit" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>

            </el-tabs>

        </el-card>
        <!-- 添加参数的对话框 -->
        <el-dialog :title="'添加'+titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <!-- 添加参数的form表单 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible=false">取消</el-button>
                <el-button type="primary" @click="addParams">确定</el-button>
            </span>

        </el-dialog>
        <!-- 修改参数的对话框 -->
        <el-dialog :title="'修改'+titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <!-- 修改参数的form表单 -->
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
                <el-form-item label="新属性名" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
                <el-form-item label="新属性值" prop="attr_vals">
                    <el-input v-model="editForm.attr_vals"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible=false">取消</el-button>
                <el-button type="primary" @click="editParams">确定</el-button>
            </span>

        </el-dialog>
    </div>
</template>

<script>
export default {
    data(){
        return {
            // 商品分类列表
            cateList: [],
            // 级联选择框的配置对象
            cateProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            // 级联选择框双向绑定
            selectedCateKeys: [],
            // 被激活的页签的名称
            activeName: 'many',
            manyTaleData: [],
            onlyTaleData: [],
            // 控制添加对话框的显示隐藏
            addDialogVisible: false,
            // 添加参数的表单数据对象
            addForm: {
                attr_name: ''
            },
            // 添加表单的验证规则的对象
            addFormRules: {
                attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
            },
            // 修改参数的对话框显示隐藏
            editDialogVisible: false,
            // 修改参数的表单数据对象
            editForm: {},
            // 修改参数的验证规则的对象
            editFormRules: {
                attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }],
                attr_vals: [{ required: true, message: '请输入参数值', trigger: 'blur' }]
            }
        }
    },
    created(){
        this.getCateList()
    },
    methods: {
        // 获取所有商品分类列表
        async getCateList(){
        const { data: res } = await this.$http.get('categories')
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品分类失败')
            } else {
                this.cateList = res.data
            }
        },
        // 级联选择框选中项变化，出发这个函数
        handleChange(){
         this.getParamsData()
        },
        async getParamsData(){
            // 如果选中的不是三级选项
            if (this.selectedCateKeys.length !== 3) {
                this.selectedCateKeys = []
                this.manyTaleData = []
                this.onlyTaleData = []
                return false
            } else {
                // 根据所选分类的id 和当前所处的面板，获取对应的参数
                const { data: res } = await this.$http.get(`categories/${this.catId}/attributes`, { params: { sel: this.activeName } })
                if (res.meta.status !== 200) {
                     return this.$message.error('获取参数列表失败')
                }
                res.data.forEach(element => {
                    element.attr_vals = element.attr_vals ? element.attr_vals.split(' ') : []
                    // 为每一行添加自己的布尔值
                    element.inputVisible = false
                    // 添加自己的input 值
                    element.inputValue = ''
                })
                if (this.activeName === 'many') {
                    this.manyTaleData = res.data
                } else {
                    this.onlyTaleData = res.data
                }
            }
        },
        // Tab页签点击事件的处理函数
        handleTabClick(){
            this.getParamsData()
        },
        // 监听对话框的关闭事件
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },
        // 添加参数的事件
        addParams(){
            this.$refs.addFormRef.validate(async vali => {
                if (!vali) {
                    return false
                } else {
                     const { data: res } = await this.$http.post(`categories/${this.catId}/attributes`, {
                         attr_name: this.addForm.attr_name,
                         attr_sel: this.activeName
                     })
                     if (res.meta.status !== 201) {
                         return this.$message.error('添加参数失败')
                     } else {
                        this.$message.success('添加参数成功')
                        this.addDialogVisible = false
                        this.getParamsData()
                     }
                }
            })
        },
        // 点击修改的触发事件
        async showEditDialog(id){
            // 对话框打开之前需要进行的准备：根据id获取参数信息,显示到对话框
            const { data: res } = await this.$http.get(`categories/${this.catId}/attributes/${id}`, {
                attr_sel: this.activeName
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取参数失败')
            } else {
                this.editForm = res.data
            }
            this.editDialogVisible = true
        },
        // 修改参数并提交
        editParams(){
            this.$refs.editFormRef.validate(async vali => {
                if (!vali) {
                    return false
                } else {
                    const { data: res } = await this.$http.put(`categories/${this.catId}/attributes/${this.editForm.attr_id}`, {
                            attr_name: this.editForm.attr_name,
                            attr_sel: this.activeName,
                            attr_vals: this.editForm.attr_vals
                    })
                     if (res.meta.status !== 200) {
                        return this.$message.error('参数修改失败' + res.meta.msg)
                    } else {
                        this.editDialogVisible = false
                        this.getParamsData()
                        return this.$message.success('参数修改成功')
                    }
                }
            })
        },
        // 监听修改对话框的关闭事件
        editDialogClosed(){
            this.$refs.editFormRef.resetFields()
        },
        // 点击删除的触发事件
        async removeParams(id){
            // 询问用户是否删除数据
            const confirmResult = await this.$confirm('此操作将永久删除该参数，是否继续？', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => {
                return err
            })
            // 用户点击取消：confirmResult == cancel，确认：confirm
            if (confirmResult === 'cancel') {
                  return this.$message.info('用户取消了删除')
            } else {
                // 发起删除的网络请求
                const { data: res } = await this.$http.delete(`categories/${this.catId}/attributes/${id}`)
                if (res.meta.status !== 200) {
                    return this.$message.error('删除失败' + res.meta.msg)
                } else {
                    this.getParamsData()
                    return this.$message.success('删除成功')
                }
            }
        },
        // 失去焦点或者按下回车键时触发
        async handleInputConfirm(row){
            if (row.inputValue.trim().length === 0) { // 输入的是一堆空格
                row.inputValue = ''
                row.inputVisible = false
                return false
            }
            // 如果输入的字符正确，则需要进行后续操作
            row.attr_vals.push(row.inputValue)
            row.inputValue = ''
            row.inputVisible = false
            // 发送网络请求，更新到后台保存
            this.saveAttrVals(row)
        },
         // 发送网络请求，属性值更新到后台保存
        async saveAttrVals(row){
            const { data: res } = await this.$http.put(`categories/${this.catId}/attributes/${row.attr_id}`, {
                        attr_name: row.attr_name,
                        attr_sel: row.attr_sel,
                        attr_vals: row.attr_vals.join(' ')
                    })
            if (res.meta.status !== 200) {
                return this.$message.error('参数修改失败' + res.meta.msg)
            } else {
                return this.$message.success('参数修改成功')
            }
        },
        // 添加按钮触发
        showInput(row){
            row.inputVisible = true
            // 让文本框自动活得焦点
            // $nextTick 作用：页面重新渲染之后，才执行其中的回调函数，包子focus时input已经存在，避免报错
            this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus()
            })
        },
        // 关闭tag标签触发的方法
        handleClose(i, row){
            // 删除i位置的一个元素，attr_vals保留的剩下的
            row.attr_vals.splice(i, 1)
              // 发送网络请求，更新到后台保存
            this.saveAttrVals(row)
        }
     },
    computed: {
        // 如果按钮需要被禁用，返回true
        isBtnDisabled(){
            if (this.selectedCateKeys.length !== 3) {
                return true
            } else {
                return false
            }
        },
        // 当前选中的三级分类的ID
        catId(){
            if (this.selectedCateKeys.length === 3) {
                return this.selectedCateKeys[2]
            }
            return null
        },
        // 根据激活的tab页签动态计算对话框标题文本
        titleText(){
            if (this.activeName === 'many') {
                return '动态参数'
            }
            return '静态属性'
        }
    }
}
</script>

<style lang="less" scoped>
.cat_opt{
    margin: 15px 0;
}
.el-tag{
    margin: 10px;
}
.input-new-tag{
    width: 130px;
}
</style>