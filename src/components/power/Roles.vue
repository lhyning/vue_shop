<template>
    <div>
        <!-- 面包屑导航 -->
          <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片区域 -->
        <el-card>
            <!-- 添加角色的按钮 -->
            <el-row>
                <el-col>
                    <el-button type="primary">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 角色列表展示区域 -->
            <el-table :data="rolesList" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <!-- 栅格系统的布局组件 -->
                        <el-row  v-for="(item1, i) in scope.row.children" :key="item1.id" :class="['bdBottom',i === 0 ? 'bdTop':'','vertical']">
                            <!-- 渲染一级权限 -->
                            <el-col :span="6">
                                <el-tag closable @close="removeRightByid(scope.row,item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级和三级权限 -->
                            <el-col :span="18">
                                <!-- for 循环嵌套渲染二级权限 -->
                                <el-row v-for="(item2, j) in item1.children" :key="item2.id" :class="[j === 0?'':'bdTop','vertical']">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightByid(scope.row,item2.id)">{{item2.authName}}</el-tag>
                                         <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <!-- for 循环嵌套渲染三级权限 -->
                                        <el-tag  v-for="item3 in item2.children" :key="item3.id" type="warning" closable @close="removeRightByid(scope.row,item3.id)">{{item3.authName}}</el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column label="#" type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="small">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="small">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="small" @click="showSetRightsDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>

        <!-- 分配权限的对话框 -->
        <el-dialog title="分配权限" :visible.sync="setRightsDialogVisible" width="50%" @close="setRightsDialogClosed">
            <!-- 树形结构图 -->
            <el-tree :data="rightsTree" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="refTree"></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightsDialogVisible = false" >取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            // 用户角色列表
            rolesList: [],
            setRightsDialogVisible: false,
            // 角色的权限
            rightsTree: [],
            // 树形结构图绑定的属性
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            // 默认选中的节点的id
            defKeys: [],
            // 当前需要分配权限的角色 id
            nodeId: ''
        }
    },
    created() {
        // 获取用户角色信息
        this.getRolesList()
    },
    methods: {
         // 获取用户角色信息
        async getRolesList() {
            const { data: res } = await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('获取用户角色列表失败')
            }
            this.rolesList = res.data
            return this.$message.success('获取用户角色列表成功')
        },
        // 根据id 删除用户的三级权限
        async removeRightByid(role, rihtId) {
            const confirmResult = await this.$confirm('确定删除用户此权限？', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('用户取消了删除动作')
            }
            // 发起删除的网络请求
            const { data: res } = await this.$http.delete('roles/' + role.id + '/rights/' + rihtId + '')
               console.log(res)
            if (res.meta.status !== 200) {
                return this.$message.error('权限删除失败')
            }
            role.children = res.data
            return this.$message.success('权限删除成功')
        },
        // 显示设置权限的对话框
        async showSetRightsDialog(allNode) {
            this.nodeId = allNode.id
            // 请求获取所有的权限数据
            const { data: res } = await this.$http.get('rights/tree')
            if (res.meta.status !== 200) {
                return this.$message.err('权限获取失败')
            }
            this.rightsTree = res.data
            this.getLeavKeys(allNode, this.defKeys)
            this.setRightsDialogVisible = true
        },
        // 递归函数获取角色下的所有权限的id，并将 id 保存到数组 defKeys 中
        getLeavKeys(node, arr) {
            // 判断此节点是否是三级节点，三级节点没有children
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => {
                this.getLeavKeys(item, arr)
            })
        },
        // 监听关闭分配权限的对话框动作
        setRightsDialogClosed(){
            this.defKeys = []
        },
        // 给当前角色分配权限
        async allotRights() {
            // 被选中状态权限的id，以及半选中状态的id
            const keys = [
                ...this.$refs.refTree.getCheckedKeys(),
                ...this.$refs.refTree.getHalfCheckedKeys()
            ]
            const idKyeys = keys.join(',')
            const { data: res } = await this.$http.post(`roles/${this.nodeId}/rights`, { rids: idKyeys })
            if (res.meta.status !== 200) {
                return this.$message.error('权限分配失败')
            }
            this.setRightsDialogVisible = false
            this.getRolesList()
            return this.$message.success('权限分配成功')
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag{
    margin: 6px;
}

.bdTop{
    border-top: 1px solid #eeeeee;
}

.bdBottom{
    border-bottom: 1px solid #eeeeee;
}

.vertical{
    display: flex;
    align-items: center;
}
</style>