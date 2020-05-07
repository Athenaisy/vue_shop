<template>
	<div>

		<!-- 面包屑导航区域 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>用户管理</el-breadcrumb-item>
			<el-breadcrumb-item>用户列表</el-breadcrumb-item>

		</el-breadcrumb>


		<el-card> 

			<el-row :gutter="20">
				<el-col :span="8">
					<el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">

						<el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
					</el-input>
				</el-col>
				<el-col :span="4">
					<el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
				</el-col>
			</el-row>

			<!--  用户列表区 -->
			<el-table :data="userlist" border stripe>
				<el-table-column label="#" type="index" ></el-table-column>
				<el-table-column label="姓名" prop="username"></el-table-column>
				<el-table-column label="邮箱" prop="email"></el-table-column>
				<el-table-column label="电话" prop="mobile"> </el-table-column>
				<el-table-column  label="身份" prop="role_name"></el-table-column>
				<el-table-column label="状态" prop="mg_state">
					<!-- 定义了一个作用域插槽
					slot-scope   接收了当前作用域的数据 
				     通过 scope.row  拿到这一行的 数据
				     通过 v-model 绑定 scope.row 的 属性值 mg_state 这一行的状态数据 true 或 false-->
				     <!--  指定了 prop 也定义了插槽  则插槽会覆盖prop 因此可删除 -->
				     <template slot-scope="scope">
				     	<el-switch
				     	v-model="scope.row.mg_state"
				     	active-color="#13ce66"
				     	inactive-color="#ccc" 
				     	@change="userStateChanged(scope.row)">
				     	</el-switch>
				     <!--  {{scope.row}} -->
				 		</template>
				</el-table-column>
				
				<el-table-column label="操作"> 
					<template slot-scope="scope"> 
						<!-- 修改按钮 -->
						<el-button type="primary" icon="el-icon-edit" size="mini" circle @click="showEditDialog(scope.row.id)"></el-button>  
						<!-- 删除按钮  -->
						<el-button type="danger" icon="el-icon-delete" size="mini" circle @click="removeUserById(scope.row.id)"></el-button>
						<!-- 分配角色按钮 -->
						<el-tooltip  effect="dark" content="分配角色" placement="right-start" :enterable="false">
							<el-button type="info" icon="el-icon-setting" size="mini" circle>

							</el-button>
						</el-tooltip> 
					</template>
				</el-table-column> 
			</el-table>
			
			<!-- 分页区域 -->
			<el-pagination
				@size-change="handleSizeChange"
				@current-change="handleCurrentChange"
				:current-page="queryInfo.pagenum"
				:page-sizes="[1, 2, 5, 10]"
				:page-size="pagesize"
				layout="prev, pager, next, jumper,  sizes, total"
				:total="total">
			</el-pagination> 
		</el-card>

	<!-- 添加用户的对话框 -->
	<el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
		<el-form :model="addForm" 
				 :rules="addFormRules"
				 ref="addFormRef"
				 label-width="70px">
			<el-form-item label="用户名" prop="username">
				<el-input v-model="addForm.username"></el-input>
			</el-form-item> 
			<el-form-item label="密码" prop="password">
				<el-input v-model="addForm.password"></el-input>
			</el-form-item>
			<el-form-item label="邮箱" prop="email">
				<el-input v-model="addForm.email"></el-input>
			</el-form-item>
			<el-form-item label="电话" prop="mobile">
				<el-input v-model="addForm.mobile"></el-input>
			</el-form-item>

		</el-form>
			<div slot="footer">
				<el-button @click="addDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="addUser">确 定</el-button>
			</div> 
	</el-dialog>

		<!-- 修改用户的对话框 -->
		<el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%">
 			<el-form :model="editForm"  :rules="editFormRules" ref="editFormRef" label-width="70px">
				<el-form-item label="用户名" disabled>
					<el-input v-model="editForm.username" ></el-input>
				</el-form-item> 
			
				<el-form-item label="邮箱" prop="email">
					<el-input v-model="editForm.email"></el-input>
				</el-form-item>
				<el-form-item label="电话" prop="mobile">
					<el-input v-model="editForm.mobile"></el-input>
				</el-form-item>
			</el-form>
 			 <!-- 页脚 -->
  			<span slot="footer" class="dialog-footer">
    			<el-button @click="editDialogVisible = false">取 消</el-button>
   			 	<el-button type="primary" @click="editUserInfo">确 定</el-button>
 		 	</span>
		</el-dialog>

		
</div>

</template>
<script>
export default{
	data() {
		// 验证邮箱的规则
		var checkEmail = (rule, value, callback) => {
			// 验证邮箱的正则表达式
			const regEmail =  /^([a-zA-Z0-9_\.\-])+\@(([a-zA-Z0-9\-])+\.)+([a-zA-Z0-9]{2,4})+$/

			if(regEmail.test(value)) {
				// 合法的邮箱
				return callback()
			}
			callback(new Error('请输入合法的邮箱'))
		}

		// 验证手机号的规则
		var checkMobile = (rule, value, callback) => {
			// 验证手机号的正则表达式
			const regMobile =  /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|17[678]|18[0-9]|14[57])[0-9]{8}$/
			if(regMobile.test(value)) {
				return callback()
			}
			callback(new Error('请输入合法的电话'))
		}
		//  
		return {
			// 获取用户列表的参数对象
			queryInfo: {
				query: '',
				// 当前页数
				pagenum: 1,
				// 当前每页显示多少条数据
				pagesize: 2

			},
			userlist: [],
			total: 0,

			// 控制添加用户对话框的显示与隐藏
			addDialogVisible: false,
			// 控制修改用户信息对话框的显示与隐藏
			editDialogVisible: false,
			// 查询到的用户信息对象
			editForm: {},
			editFormRules: {
				email: [
				{
					required: true,
					message: '请输入用户邮箱',
					trigger: 'blur'
				},
				{
					validator: checkEmail,
					trigger: 'blur'
				}],
				mobile: [
				{
					required: true,
					message: '请输入电话',
					trigger: 'blur'
				},
				{
					validator: checkMobile,
					trigger: 'blur'
				}]
			},

			
			//添加用户的表单数据
			addForm: {
				username: '',
				password: '',
				email: '',
				mobile: ''
				},

			//添加表单的验证规则对象
			addFormRules: {
				username: [
				{
					required: true,
					message: '请输入用户名',
					trigger: 'blur'
				},
				{
					min: 3,
					max: 12,
					message: '用户名的长度在3-12字符之间',
					trigger: 'blur'
				}],
				password: [
				{
					required: true,
					message: '请输入密码',
					trigger: 'blur'
				},
				{
					min: 6,
					max: 12,
					message: '密码的长度在6-12字符之间',
					trigger: 'blur'
				}],
				email: [
				{
					required: true,
					message: '请输入邮箱',
					trigger: 'blur'
				},
				{
					validator: checkEmail,
					trigger: 'blur'
				}],
				mobile: [
				{
					required: true,
					message: '请输入电话',
					trigger: 'blur'
				},
				{
					validator: checkMobile,
					trigger: 'blur'
				}]
			}

		}

	},
	created() {
		this.getUserList()
	},
	methods:  {
		async getUserList() {
			const {data : res } = await this.$http.get('users', { params: this.queryInfo })
			if(res.meta.status !== 200)  return this.$message.error('获取用户列表失败')
				this.userlist = res.data.users
			this.total = res.data.total

		},
		//监听pageSize改变的事件
		handleSizeChange(newSize){
			// console.log(newSize)
			this.queryInfo.pagesize = newSize
			this.getUserList()
		},
		// 监听页码值改变的事件
		handleCurrentChange (newPage) {
			// console.log(newPage)
			this.queryInfo.pagenum = newPage
			this.getUserList()

		},
		// 监听switch 状态的的改变
		async userStateChanged (userinfo) {
			const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
			if(res.meta.status !== 200 ) {
				userinfo.mg_state = !userinfo.mg_state
				return this.$message.error('更新用户状态失败')
			}
			this.$message.success('更新用户状态成功')

		},
		addDialogClosed () {
			this.$refs.addFormRef.resetFields()

		},
		// 点击按钮，添加新用户
		addUser() {
			this.$refs.addFormRef.validate( async valid => {
				//console.log(valid)
				if(!valid) return 
					// 可以发起添加用户的网络请求
				const { data: res } = await this.$http.post('users', this.addForm)

				if(res.meta.status !== 201 ) {
					this.$message.error('添加用户失败!')
				}

				this.$message.success('添加用户成功!')
				//隐藏添加用户的对话框
				this.addDialogVisible = false
				// 重新获取用户列表数据
				this.getUserList()
			})
		},
		// 展示编辑用户的对话框
		async showEditDialog (id) {
			//console.log(id)
		const { data: res } = await	this.$http.get('users/' + id)
		if(res.meta.status !== 200) {
			return this.$message.error('查询用户信息失败')
			
		}
		this.editForm = res.data
		this.editDialogVisible = true
		},
		// 修改用户信息并提交
		editUserInfo () {
			this.$refs.editFormRef.validate( async valid => {
				//console.log(valid)
				if(!valid)  return 
					// 发起修改用户信息的请求
				const {data: res }	= await this.$http.put('users/' + this.editForm.id,
				{

					email: this.editForm.email,
					mobile: this.editForm.mobile
				})

				if(res.meta.status !== 200 ) {
					this.$message.error('更新用户信息失败')

				}
				// 关闭对话框
				this.editDialogVisible = false
				// 刷新用户列表
				this.getUserList()
				// 提示修改成功
				this.$message.success('更新用户信息成功！')

			})

		},
		// 根据id删除对应的用户信息
		async removeUserById (id) {
			// 弹框 询问用户是否删除数据
        	const confirmResult  = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示',
        	 {
         	 	confirmButtonText: '确定',
          		cancelButtonText: '取消',
          		type: 'warning'
       	 	/// catch 捕获所有错误
       		 }
       		 ).catch(err  =>  err)
          
          // 如果用户确认删除， 则返回值为字符串
          // 如果用户 取消了删除 则返回值为字符串   
          if (confirmResult !== 'confirm') {
          	return this.$message.info('已取消删除')
         	 }    
         	   const { data: res } = await this.$http.delete('users/' + id)

         	   if(res.meta.status !== 200 ){
         	   	 return this.$message.error('删除用户失败')
         	   	}
         	   	this.$message.success('删除用户成功')
         	   	this.getUserList()
        	}
        }

};
</script>
<style lang="less">
.el-breadcrumb {
	margin-bottom: 15px;
}
.el-card{
	box-shadow: 0 1px 1px 1px #eee;
}
.el-table{
	font-size: 12px;
	margin-top: 15px;
}
.el-pagination{
	margin-top: 15px;
}


</style> 