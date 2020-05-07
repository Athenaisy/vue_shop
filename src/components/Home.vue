<template>
	
	<el-container class="home-container">
		<!-- 头部区域 -->
		<el-header>
			<div>	
				<img src="../assets/heima.png" alt="黑墨水">
				<span>	后台管理系统</span>
			</div>
			<el-button type="info" @click="logout">退出</el-button>		
		</el-header>

		<!-- 页面主体区域 -->
		<el-container>
			<!-- 侧边栏 -->
			<el-aside :width="isCollapse ? '64px' : '200px'">
				<div class="toggle-button" @click="toggleCollapse">|||
					
				</div>
				<!-- 侧边栏菜单区域-->
				<el-menu 
				:collapse="isCollapse"
				:collapse-transition="false"
				:unique-opened="true"
				:router="true"
				:default-active="activePath"
				background-color="rgb(75, 171, 145)"
				text-color="#eee"
				active-text-color="#dfe2a9">

				<!-- 一级菜单 -->
				<el-submenu 
						:index="item.id + '' "
						v-for="item in menulist" 
						:key="item.id" 
						>
					<!-- 一级菜单模板区域 -->
					<template slot="title">
						<i :class="iconsObj[item.id]"></i>
						<span>{{item.authName}}</span>
					</template>
					<!-- 二级菜单 -->
					<el-menu-item 
						:index="'/' + subItem.path + ''" 
						v-for="subItem in item.children" 
						:key="subItem.id"
						@click="saveNavState('/' + subItem.path + '')">
						<template slot="title">
							<i class="el-icon-menu"></i>
							<span>{{subItem.authName}}</span>
					</template>
				</el-menu-item>
				
					
				</el-submenu>
				
			</el-menu>
		</el-aside>
		<!-- 右侧区域 -->
		<el-main>
			<!-- 路由占位符 -->
			<router-view></router-view>
		</el-main>

	</el-container>
</el-container>
</template>


<script>
export default {
	data() {
		return {
			// 左侧菜单数据
			menulist: [],
			iconsObj: {
				'125': 'iconfont icon-users',
				'103': 'iconfont icon-tijikongjian',
				'101': 'iconfont icon-shangpin',
				'102': 'iconfont icon-danju',
				'145': 'iconfont icon-baobiao'
			},
			// 默认不折叠
			isCollapse: false,
			activePath: ''
		}
	},
	created() {
		this.getMenuList()
		this.activePath = window.sessionStorage.getItem('activePath')
	},
	methods:{
		logout() {
			window.sessionStorage.clear()
			this.$router.push('/login')
		},
		// 获取所有的菜单
		async getMenuList() {
			const {data: res } = await this.$http.get('menus')
			if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
				this.menulist = res.data
			console.log(res)
		},
		toggleCollapse () {
			// 点击按钮切换菜单的折叠与展开
			this.isCollapse = !this.isCollapse

		},
		// 保存链接的激活状态
		saveNavState(activePath) {
			window.sessionStorage.setItem('activePath',activePath)
			this.activePath = activePath

		}

	}
};
</script>
<style lang="less" scoped>
.home-container{
	margin: 0;
	width: 100%;
	height: 100%;
}
.el-header{
	display: flex;
	justify-content: space-between;
	align-items: center;
	color: #fff;
	font-size: 24px;
	font-family: SimSun;
	font-weight: bold;
	>div {
		display: flex;
		align-items: center;
		span{
			margin-left: 10px;
		}
	}
	

	background-color:  #6aab9a;
	
	
}
.el-aside{
	
	background-color: rgb(75, 171, 145);
	.el-menu{
		border-right: none;
	}
}
.el-main{
	background-color: #63bda0;
	
}
.iconfont{
	margin-right: 10px;
	color: #eee;
}
.toggle-button{
	color: #ccc;
	font-size: 12px;
	line-height: 24px;
	text-align: center;
	letter-spacing: 0.1em;
	background-color: #629c8d;
	cursor: pointer;
}
</style>