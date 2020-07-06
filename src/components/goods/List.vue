<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区 -->
        <el-card>
            <!-- 输入框 -->
            <el-row :gutter="20">
                <!-- 搜索框 -->
                <el-col :span="9">
                    <el-input placeholder="请输入内容" v-model="goodsKeyword" clearable @clear='getGoodsList'>
                        <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                    </el-input>
                </el-col>
                <!-- 添加商品按钮 -->
                <el-col :span="2">
                    <el-button type="primary" :span="5" @click="toAddGoodsPath">添加商品</el-button>
                </el-col>
            </el-row>
            <!-- 商品列表表格 -->
            <el-table :data="goodsData" border stripe size="small">
                <el-table-column type="index" align="center"></el-table-column>
                <el-table-column prop="goods_name" label="商品名称" width="700px"></el-table-column>
                <el-table-column prop="goods_price" label="商品价格(元)" width="100px" align="center"></el-table-column>
                <el-table-column prop="goods_weight" label="商品重量" width="80px" align="center"></el-table-column>
                <el-table-column prop="add_time" label="创建时间" align="center"></el-table-column>
                <el-table-column label="操作" align="center">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeGoods(scope.row)"></el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="10"
                layout="total, sizes, prev, pager, next, jumper"
                :total="goodsTotal">
            </el-pagination>
        </el-card>
    </div>
</template>
<script>
export default {
    created() {
        //创建时渲染商品列表
        this.getGoodsList()
    },
    data() {
        return {
            //请求参数信息
            queryInfo: {
                query: '',
                pagenum: 1,
                pagesize: 10
            },
            //商品搜索关键字
            goodsKeyword: '',
            //商品数据信息
            goodsData: [],
            //商品总数
            goodsTotal: 0
        }
    },
    methods: {
        //获取商品信息
        async getGoodsList() {
            this.queryInfo.query = this.goodsKeyword
            const{ data: res} = await this.$http.get('goods',{params: this.queryInfo})
            console.log(res)
            //将时间由毫秒转换为正常格式
            res.data.goods.forEach( (item,index) => {
                item.add_time = new Date(item.add_time).toLocaleString()
                item.upd_time = new Date(item.upd_time).toLocaleString()
            })
            if(res.meta.status !== 200) return this.$message.error("获取商品列表失败")
            this.goodsData = res.data.goods
            this.goodsTotal = res.data.total
            if(this.goodsData.length == 0) {
                this.$message("未搜索到相关商品")
            }else {
                this.$message.success("获取商品列表成功")
            }
        },
        //页面大小改变
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize
            this.getGoodsList()
        },
        //页码改变
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage
            this.getGoodsList()
        },
        //删除商品
        async removeGoods(item) {
            //弹出提示框
            const confirm = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            })
            if(confirm == 'confirm') {
                //删除商品
                this.$http.delete(`goods/${item.goods_id}`)
                this.$message.success("删除商品成功")
                this.getGoodsList()
            }else {
                //取消删除
                this.$message("取消删除")
            }
        },
        //前往添加商品页面
        toAddGoodsPath() {
            this.$router.push('/goods/add')
        }
    }
}
</script>
<style lang="less" scoped>
    .el-pagination {
        margin-top: 10px;
    }
</style>