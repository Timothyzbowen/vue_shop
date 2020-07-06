<template>
		<el-container class="homeContainer">
  			<el-header>
				<div>
					<img src="../assets/heima.png" alt="">
					<span>电商后台管理系统</span>
				</div>
				<el-button type="info" @click="logout">退出</el-button>
			</el-header>
		<el-container>
			<el-aside :width = "isCollapse ? '65px' : '200px'">
				<div class="toggleCollapse" @click="toggle">|||</div>
				<el-menu class="elMenu" background-color="#333744" text-color="#fff" unique-opened
				:collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
      				<el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
						  <!-- 一级菜单 -->
        				<template slot="title">
          					<i :class="iconsObj[item.id]"></i>
          					<span>{{ item.authName }}</span>
        				</template>
						<!-- 二级菜单 -->
        				<el-menu-item :index="'/' + subitem.path" v-for="subitem in item.children" :key="subitem.id" @click="active('/' + subitem.path)">
							<template slot="title">
        						<i class="el-icon-menu"></i>
        						<span>{{ subitem.authName }}</span>
							</template>
      					</el-menu-item>
      				</el-submenu>
      			</el-menu>
			</el-aside>
			<el-main>
				<router-view></router-view>
			</el-main>
		</el-container>
		</el-container>
</template>

<script>
export default {
  created () {
    this.getMenulist()
    this.activePath = window.sessionStorage.getItem('currentPath')
  },
  data () {
    return {
      menulist: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenulist () {
      const { data: ret } = await this.$http.get('menus')
      this.menulist = ret.data
      console.log(ret.data)
    },
    toggle () {
      this.isCollapse = !this.isCollapse
    },
    active (currentPath) {
      window.sessionStorage.setItem('currentPath', currentPath)
      this.activePath = currentPath
    }
  }
}
</script>

<style lang="less" scoped>
	.homeContainer {
		height: 100%;
	}
	.el-header {
		background-color: #373d41;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding-left: 0;
		> div {
			display: flex;
			align-items: center;
			span {
				font-size: 20px;
				color: white;
				margin-left: 10px;
			}
		}
	}
	.el-aside {
		background-color: #333744;
	}
	.el-main {
		background-color: #eaedf1;
	}
	.elMenu {
		border-right: none;
	}
	.iconfont {
		margin-right: 10px;
	}
	.toggleCollapse{
		background-color: #4a5064;
		color: #fff;
		text-align: center;
		font-size: 12px;
		line-height: 22px;
		letter-spacing: 3px;
		cursor: pointer;
	}
</style>
