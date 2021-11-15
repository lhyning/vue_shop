<template>
    <div>
        <!-- 面包屑导航栏 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{path: '/home'}">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- 表格区域 -->
            <tree-table class="TreeTable" :data="categoryList" :columns="columns" :selection-type="false" :expand-type="false" :show-index="true" index-text="#" border>
                <!-- 是否有效 -->
                <template slot="isUseful" slot-scope="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted==false" style="color:lightgreen;"></i>
                    <i class="el-icon-error" v-else style="color:red;"></i>
                </template>
                <!-- 排序 -->
                <template slot="order" slot-scope="scope">
                    <el-tag v-if="scope.row.cat_level==0" size="small">一级</el-tag>
                    <el-tag type="success" v-else-if="scope.row.cat_level==1" size="small">二级</el-tag>
                    <el-tag type="warning" v-else size="small">三级</el-tag>
                </template>
                <!-- 操作按钮 -->
                <template slot="btn">
                    <el-button type="primary" icon="el-icon-edit" size="small">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="small">删除</el-button>
                </template>
            </tree-table>
            <!-- 分页区域 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[2, 5, 10, 20]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>

        <!-- 添加分类的对话框 -->
        <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
            <!-- 添加分类的表单 -->
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类：">
                     <el-cascader :options="cateParentList" :props="cascaderProps" v-model="selectedkeys" @change="parentCateChange" ref="cascaderRef" clearable></el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialogVisible = false">取消</el-button>
                <el-button type="primary" @click="addCate">确定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data(){
        return {
            // 查询条件
            queryInfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            // 商品分类列表
            categoryList: [],
            // 总数
            total: 0,
            // tree-table表格样式属性设置:列，
            columns: [{
                label: '分类名称',
                prop: 'cat_name'
            },
            {
                label: '是否有效',
                prop: 'cat_deleted',
                type: 'template',
                template: 'isUseful'
            },
            {
                label: '排序',
                prop: 'cat_level',
                type: 'template',
                template: 'order'
            },
            {
                label: '操作',
                prop: '',
                type: 'template',
                template: 'btn'
            }],
            addCateDialogVisible: false,
            // 添加分类的form表单数据对象
            addCateForm: {
                cat_pid: 0,
                cat_name: '',
                cat_level: 0
            },
            // 添加分类的验证规则对象
            addCateFormRules: {
                cat_name: [
                    { required: true, message: '请输入分类名称', trigger: 'blur' }
                ]
            },
            // 父级分类的列表
            cateParentList: [],
            // 指定级联选择器的配置对象
            cascaderProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children',
                checkStrictly: true,
                expandTrigger: 'hover'
            },
            // 选中的父级分类的id
            selectedkeys: []
        }
    },
    created() {
        this.getCategoryList()
    },
    methods: {
        async getCategoryList() {
            const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品分类数据列表失败')
            }
            this.categoryList = res.data.result
            this.total = res.data.total
        },
        // 监听pagesize 的改变
        handleSizeChange(newSize){
            this.queryInfo.pagesize = newSize
            this.getCategoryList()
        },
        // 监听当前页码值变化
        handleCurrentChange(newpageNum){
            this.queryInfo.pagenum = newpageNum
            this.getCategoryList()
        },
        // 点击按钮，显示添加分类的对话框
        showAddCateDialog(){
            this.getParentCateList()
            this.addCateDialogVisible = true
        },
        // 获取父级数据分类列表
        async getParentCateList(){
            const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
            if (res.meta.status !== 200) {
                return this.$message.error('获取父级分类数据列表失败')
            }
            this.cateParentList = res.data
        },
        // 选择项发生变化时触发的事件
        parentCateChange(){
             // 如果 selectedkeys 数组长度>0,证明选中了父级分类
            // 否则，说明没有选中任何父级分类
            if (this.selectedkeys.length > 0) {
                this.addCateForm.cat_pid = this.selectedkeys[this.selectedkeys.length - 1]
                this.addCateForm.cat_level = this.selectedkeys.length
            } else {
                // 父级分类的ID
                this.addCateForm.cat_pid = 0
                // 当前分类的等级赋值
                this.addCateForm.cat_level = 0
            }
            this.$refs.cascaderRef.dropDownVisible = false
        },
        // 点击确认按钮,添加新的分类
        addCate(){
            this.$refs.addCateFormRef.validate(async valid => {
                if (!valid) return false
                const { data: res } = await this.$http.post('categories', this.addCateForm)
                console.log(res)
                if (res.meta.status !== 201) {
                    return this.$message.error('商品分类添加失败')
                }
                this.addCateDialogVisible = false
                this.$message.success('商品分类添加成功')
                this.getCategoryList()
            })
        },
        // 监听对话框关闭，重置表单数据
        addCateDialogClosed(){
            this.$refs.addCateFormRef.resetFields()
            this.selectedkeys = []
            this.addCateForm.cat_level = 0
            this.addCateForm.cat_pid = 0
            this.addCateForm.cat_name = ''
        }
    }
}
</script>

<style lang="less" scoped>
.TreeTable{
    margin-top: 10px;
}
.el-cascader{
    width: 100%;
}
</style>