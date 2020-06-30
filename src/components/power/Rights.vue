<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区 -->
        <el-card>
        <!-- 表格区 -->
            <el-table :data="rightslist" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column prop="authName" label="权限名称"></el-table-column>
                <el-table-column prop="path" label="路径"></el-table-column>
                <el-table-column prop="level" label="权限等级">
                    <template slot-scope="scope">
                        <el-button :type="rightsLevel[scope.row.level][1]" size="small" plain>{{ rightsLevel[scope.row.level][0] }}级</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
    </div>
</template>

<script>
export default {
    created() {
        this.getRightslist()
    },
    data() {
        return {
           rightslist: [],
           rightsLevel: {
               '0': ["一", "primary"],
               '1': ["二", "success"],
               '2': ["三", "warning"]
           }
            
        }
    },
    methods: {
        async getRightslist() {
            const{data: res} = await this.$http.get('rights/list')
            this.rightslist = res.data
        }
    }
}
</script>

<style lang="less" scoped>

</style>