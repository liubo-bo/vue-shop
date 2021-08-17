<template>
	<div>
		<!-- 面包屑导航区 -->
		<el-breadcrumb separator="/">
		  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
		  <el-breadcrumb-item>权限管理</el-breadcrumb-item>
		  <el-breadcrumb-item>角色列表</el-breadcrumb-item>
		</el-breadcrumb>
		<!-- 卡片视图 -->
		<el-card>
			<!-- 添加角色按钮区 -->
			<el-row>
				<el-col>
					<el-button type="primary" @click="addRolesDialog = true">添加角色
					</el-button>
				</el-col>
			</el-row>
			<!-- 角色列表区域 -->
			<el-table :data="rolelist" border stripe>
				<!-- 展开列 -->
				<el-table-column type="expand">
					<template slot-scope="scope">
						<el-row :class="['bdbottom', 'vcenter',i1 === 0 ? 'bdtop':'']" 
						v-for="(item1, i1) in scope.row.children"
						:key="item1.id">
							<!-- 渲染一级权限 -->
							<el-col :span='5'>
								<el-tag class="el-tag" closable
										@close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>	
								<i class="el-icon-caret-right"></i>
							</el-col>
							<!-- 渲染二级和三级权限 -->
							<el-col :span='19'>
								<!-- 通过循环嵌套渲染二级权限 -->
								<el-row v-for="(item2, i2) in item1.children"
								:key="item2.id" :class="['vcenter', i2 === 0 ? '':'bdtop']">
									<!-- 两列 -->
									<el-col :span='6'>
										<el-tag type="success" closable
										@close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
										<i class="el-icon-caret-right"></i>
									</el-col>
									<el-col :span='18'>
										<el-tag v-for="(item3, i3) in item2.children"
										:key="item3.id" type="warning" closable
										@close="removeRightById(scope.row,item3.id)">
											{{item3.authName}}
										</el-tag>
									</el-col>
								</el-row>
							</el-col>
						</el-row>
											
					</template>
				</el-table-column>
				<!-- 索引列 -->
				<el-table-column type="index"></el-table-column>
				<el-table-column label="角色名称" prop="roleName"></el-table-column>
				<el-table-column label="角色描述" prop="roleDesc"></el-table-column>
				<el-table-column label="操作" width="300px">
					<template slot-scope="scope">
						<el-button type="primary" icon="el-icon-edit" size="mini">
							编辑</el-button>
						<el-button type="danger" icon="el-icon-delete" size="mini">
							删除</el-button>
						<el-button type="warning" icon="el-icon-setting" size="mini"
						@click="showSetRightDialog(scope.row)">
							分配权限</el-button>
					</template>
				</el-table-column>
			</el-table>
		</el-card>
		<!-- 添加角色对话框 -->
		<el-dialog title="添加角色" :visible.sync="addRolesDialog"
		width="50%" @close="addRolesClosed" >
			<!-- 添加角色内容主题区域 -->
			<el-form :model="addRolesForm" :rules="addRolesRules" ref="addRolesRef" 
			label-width="80px">
				<el-form-item label="角色名称" prop="roleName">
					<el-input v-model="addRolesForm.roleName"></el-input>
				</el-form-item>
				<el-form-item label="角色描述" prop="roleDesc">
					<el-input v-model="addRolesForm.roleDesc"></el-input>
				</el-form-item>
			</el-form>
			<!-- 底部区域 -->
			<span slot="footer" class="dialog-footer">
				<el-button @click="addRolesDialog = false">取消</el-button>
				<el-button type="primary" @click='addRoles'>确定</el-button>
			</span>
		</el-dialog>
		<!-- 分配权限对话框 -->
		<el-dialog title="分配权限" :visible.sync="setRightDialogVisible"
		width="50%" @close="setRightDialogClosed" >
			<!-- 添加用户内容主题区域 -->
			<el-form :model="addRolesForm" :rules="addRolesRules" ref="addRolesRef" 
			label-width="80px">
			</el-form>
			<!-- 树形权限数据 -->
			<el-tree :data="rightlist" :props="treeProps"
			show-checkbox node-key="id" default-expand-all
			:default-checked-keys="defKyes" ref="treeRef"></el-tree>
			<!-- 底部区域 -->
			<span slot="footer" class="dialog-footer">
				<el-button @click="setRightDialogVisible = false">取消</el-button>
				<el-button type="primary" @click='allotRights'>确定</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
export default {
	data() {
		return {
			rolelist: [],
			addRolesForm: {
					roleName: '',
					roleDesc: ''
				},
			addRolesRules: {
				roleName: [
					{ required: true, message: '请输入角色名称', trigger: 'blur'},
					{ min: 2, max: 10, message: '用户名的长度在3-10个字符之间', trigger: 'blur'}
				],
				roleDesc: [
					{ required: true, message: '请输入角色描述', trigger: 'blur'},
					{ min: 3, max: 10, message: '用户名的长度在3-10个字符之间', trigger: 'blur'}
				]
			},
			addRolesDialog: false,	
			setRightDialogVisible: false,
			rightlist: [],
			treeProps: {
				label: 'authName',
				children: 'children'
			},
			//默认选中的节点ID值
			defKyes: [],
			roleId: ''
		}
	},
	created() {
		this.getRolesList()
	},
	methods: {
		//获取所有角色列表
		async getRolesList() {
			const { data: res } = await this.$http.get('roles')
			if(res.meta.status !== 200) {
				return this.$message.error('获取角色列表失败')
			}
			this.rolelist = res.data
		},
		addRolesClosed() {
			this.$refs.addRolesRef.resetFields()		
		},
		
		//添加角色
		addRoles() {
			 this.$refs.addRolesRef.validate( async valid => {
				 //console.log(valid)
				 //如果不通过直接返回
				 if(!valid) return
				 //如果通过，可以发起添加用户的网络请求
				 const { data: res } = await this.$http.post
				 ('roles', this.addRolesForm)
				 
				 if(res.meta.status !== 201) {
					 this.$message.error('添加用户失败')
				 }
				 this.$message.success('添加用户成功')
				 this.getRolesList() 
				 this.addRolesDialog = false
			 })
		},
		async removeRightById(role, rightId) {
			const confirmResult =  await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
			          confirmButtonText: '确定',
			          cancelButtonText: '取消',
			          type: 'warning'
			        }).catch(err => {
						return err
					})
					if(confirmResult !== 'confirm') {
						return this.$message.info('已取消删除')
					}
					const { data: res } = await this.$http.delete(`
					roles/${role.id}/rights/${rightId}`)
					if(res.meta.status !== 200) {
						return this.$message.error('删除用户失败')
					} 
					this.$message.success('删除用户成功')
					role.children = res.data
		},
		//展示分配去哪先的对话框
		async showSetRightDialog(role) {
			// 获取所有权限数据
			this.roleId = role.id
			const { data: res } = await this.$http.get('rights/tree')
			if(res.meta.status !== 200) {
				return this.$message.error('获取权限数据失败')
			} 
			// 把获取的权限数据保存到data
			this.rightlist = res.data
			this.setRightDialogVisible = true
			
			this.getLeafKeys(role, this.defKyes)
		},
		// 通过递归函数获取三级权限的数据id 保存到数组中
		getLeafKeys(node, arr) {
			if(!node.children) {
				return arr.push(node.id)
			}
			
			node.children.forEach(item => 
			this.getLeafKeys(item, arr)
			)
		},
		setRightDialogClosed() {
			this.defKyes = []
		},
		//点击确定分配权限
		async allotRights() {
			const keys = [
				...this.$refs.treeRef.
				getCheckedKeys(),
				...this.$refs.treeRef.
				getHalfCheckedKeys()
			]
			
			const idStr = keys.join(',')
			const { data: res } = await this.$http.post(`
			roles/${this.roleId}/rights`, { rids: idStr })
			if(res.meta.status !== 200) {
				return this.$message.error('分配权限失败')
			}
			this.$message.success('分配权限成功')
			this.getRolesList()
			this.setRightDialogVisible = false
		}
	}
	
}
</script>

<style lang="less" scoped>
.el-tag {
	margin: 7px;
}
.bdtop {
	border-top: 1px solid #eee;
}
.bdbottom {
	border-bottom: 1px solid #eee;
}
.vcenter {
	display: flex;
	align-items: center;
}
</style>
