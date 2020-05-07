<template>
	<div class="login_container">
		<!-- <img src="../assets/" alt="logo.png"> -->
		<div class="login_box">
			<!-- 头像 -->
			<img src="../assets/logo.png" class="logo">

			<!-- 登陆表单 -->
			<el-form class="login_form" ref="loginFormRef" :model="loginForm" :rules="loginFormRules">
				<!-- 用户名 -->
				<el-form-item prop="username" >
					<el-input v-model="loginForm.username" prefix-icon="el-icon-user-solid" placeholder="请输入用户名"></el-input>
				</el-form-item>
				<!-- 密码 -->
				<el-form-item prop="password" >
					<el-input  v-model="loginForm.password" prefix-icon="el-icon-lock" show-password  placeholder="请输入密码"></el-input>
				</el-form-item>
				<!-- 按钮 -->
				<el-form-item class="btns">
					<el-button type="primary" @click="login">登录</el-button>
				  	<el-button type="info" @click="resetLoginForm">重置</el-button>
				</el-form-item>
				 
			</el-form>

			</div>
				

		</div>
	</template>
	<script>
	export default {
		name: "",
		data() {
			return {
				// 这是登陆表单的数据绑定对象
				loginForm: {
					username: '',
					password: ''
				},
				// 这是表单的验证规则对象
				loginFormRules: {
					// 验证用户名是否合法
					username: [
						{
							required: true,
							 message: '请输入用户名',
								trigger: 'blur'
						},
						{
							min: 3,
							max: 12,
							message: '长度在3到12个字符',
							trigger: 'blur'

						}
					],
					// 验证密码是否合法
					password: [
						{
							required: true,
							message: '请输入密码',
							trigger: 'blur'
						},
						{
							min: 6, 
							max: 16, 
							message: '长度在6到16个字符',
							trigger: 'blur'
						}
					]
					
				}

			}
		},
		methods: {

			///点击重置按钮 重置登陆表单
			resetLoginForm() {
				//console.log(this);
				this.$refs.loginFormRef.resetFields();
			},
			login () {
				this.$refs.loginFormRef.validate( async valid =>{
					if(!valid) return;
					const { data: res} = await this.$http.post('login', this.loginForm);
					if(res.meta.status !== 200)
						return this.$message.error("用户名或密码错误");
					 this.$message.success("登陆成功");
					 // 将登陆成功之后的 token ， 保存到客户端的sessionStorage 中
					 	// 1.1 项目中除了登陆之外的API接口 必须在的登陆之后才能访问
					 	// 1.2 token 只应在当前网站打开期间生效，所以将 token 保存在sessionStorage  中
					 // console.log(res);
					 window.sessionStorage.setItem("token",res.data.token);
					 // 2.通过编程式导航跳转到后台主页，路由地址是 /home
					 this.$router.push('/home');
				});
				
			}
		}
	};
	</script>
	<style lang="less" scoped>
	.login_container{
		height: 100%;  
		background-color: #73a55a;
		

	}
	.login_box{
		width: 450px;
		height: 300px;
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%,-50%);
		background-color: white;
		border-radius: 10px;

	}
	.logo{
		border: 7px solid #fbf7f7;
		background-color: #eef3ec;
		border-radius: 50%;
		margin-top: -20%;
		margin-left:  28%;
		box-shadow: 0 0  10px #ddd;
	}
	.login_form{
		padding: 0 25px;

	}
	.btns{
		display: block;
		float: right;
		
	}

	</style>