<template>
	<div class="login_container">
		<div class="login_box">
			<div class="avatar">
				<img src="../assets/logo.png" alt="">
			</div>
			<el-form ref="loginFormRef" label-width="0px" :model="loginForm" :rules="loginFormRules" class="login_form">
  				<el-form-item prop="username">
    				<el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
  				</el-form-item>
  				<el-form-item prop="password">
    				<el-input type="password" v-model="loginForm.password" prefix-icon="iconfont icon-3702mima"></el-input>
  				</el-form-item>
  				<el-form-item class="login_btns">
    				<el-button type="primary" @click="login">登录</el-button>
    				<el-button type="info" @click="loginFormReset">重置</el-button>
  				</el-form-item>
  			</el-form>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				loginForm: {
					username: '',
					password: ''
				},
				loginFormRules: {
					username: [
            			{ required: true, message: '请输入用户名', trigger: 'blur' },
            			{ min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' }
          				],
          			password: [
          				{ required: true, message: '请输入密码', trigger: 'blur' },
            			{ min: 3, max: 12, message: '长度在 3 到 12 个字符', trigger: 'blur' }
          				]
				}
			}
		},
		methods: {
			loginFormReset() {
				this.$refs.loginFormRef.resetFields()
			},
			login() {
				this.$refs.loginFormRef.validate( async valid => {
					if(!valid) return;
					const { data: result } = await this.$http.post('login', this.loginForm)
					if(result.meta.status !==200) { this.$message.error({
						message: '登录失败',
						duration: 1000
					})}
					else {this.$message.success({
						message: '登录成功',
						duration: 1000
						})
						window.sessionStorage.setItem('token', result.data.token)
						this.$router.push('/home')
				}

				})
			}
		}
	}
</script>

<style lang="less" scoped>
	.login_container {
		background-color: #2b4b6b;
		height: 100%;
	}
	.login_box {
		width: 450px;
		height: 300px;
		background-color: #fff;
		border-radius: 3px;
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%,-50%);
		.avatar {
			height: 100px;
			width: 100px;
			background: #fff;
			border-radius: 50%;
			padding: 10px;
			position: absolute;
			left: 50%;
			box-shadow: 0 0 5px #D8BFD8;
			transform: translate(-50%,-50%);
			img {
				width: 100%;
				border: 1px solid #D8BFD8;
				border-radius: 50%;
				background-color: #F5F5F5;
			}
		}
		.login_form {
			position: absolute;
			width: 100%;
			top: 70px;
			padding: 10px;
			box-sizing: border-box;
		}
		.login_btns {
			display: flex;
			justify-content: flex-end;
		}
	}
</style>
