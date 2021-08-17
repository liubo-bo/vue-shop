<template>
	<div>
		<!-- 面包屑导航区 -->
		<el-breadcrumb separator="/">
		  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
		  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
		  <el-breadcrumb-item>参数列表</el-breadcrumb-item>
		</el-breadcrumb>
		<!-- 卡片视图 -->
		<el-card>
			<!-- 警告区域 -->
			<el-alert title="注意!只允许为第三类分类设置相关参数" 
			type="warning" :closable="false" show-icon>			
			</el-alert>
			<!-- 选择商品分类区域 -->
			<el-row class="cat_opt">
				<el-col>
					<span>选择商品分类:</span>
					<!-- 选择商品分类的级联选择框 -->
					<el-cascader
					   v-model="selectedCateKeys"
					   :options="catelist"				  
						:props="cateProps"
					   @change="handlechange"
						clearable ></el-cascader>
				</el-col>
			</el-row>
			  <el-tabs v-model="activeName" @tab-click="handleTabClick">
			    <el-tab-pane label="动态参数" name="many">
					<el-button type="primary" size="mini"
					:disabled="isBtnDisabled"
					@click="addDialogVisible = true">添加参数</el-button>
					<!-- 动态参数表格 -->
					<el-table :data="manyTableData" border stripe>
						<!-- 展开行 -->
						<el-table-column type="expand">
							<template slot-scope="scope">
								<!-- 循环渲染tag标签 -->
								<el-tag v-for="(item,index) in scope.row.attr_vals"
								:key="index" closable @close=
								"handleclose(index, scope.row)">{{item}}</el-tag>
								<el-input
								  class="input-new-tag"
								  v-if="scope.row.inputVisible"
								  v-model="scope.row.inputValue"
								  ref="saveTagInput"
								  size="small"
								  @keyup.enter.native=
								  "handleInputConfirm(scope.row)"
								  @blur="handleInputConfirm(scope.row)"
								>
								</el-input>
								<el-button v-else class="button-new-tag" size="small" 
								@click="showInput(scope.row)">+ New Tag</el-button>
							</template>
						</el-table-column>
						<!-- 索引列 -->
						<el-table-column type="index"></el-table-column>
						<el-table-column label="参数名称" prop="attr_name">
						</el-table-column>
						<el-table-column label="操作">
							<template slot-scope="scope">
								<el-button type="primary" icon="el-icon-edit" 
								size="mini" @click="showEditDialog((scope.row.attr_id))">
								编辑</el-button>
								<el-button type="danger" icon="el-icon-delete"
								 size="mini" @click="remobeParams(scope.row.attr_id)">
								 删除</el-button>
							</template>
						</el-table-column>
					</el-table>
					</el-tab-pane>
			    <el-tab-pane label="静态属性" name="only">
					<el-button type="primary" size="mini"
					:disabled="isBtnDisabled" 
					@click="addDialogVisible = true">添加属性</el-button>
					<!-- 静态参数表格 -->
					<el-table :data="onlyTableData" border stripe>
						<!-- 展开行 -->
						<el-table-column type="expand">
							<template slot-scope="scope">
								<!-- 循环渲染tag标签 -->
								<el-tag v-for="(item,index) in scope.row.attr_vals"
								:key="index" closable @close=
								"handleclose(index, scope.row)">{{item}}</el-tag>
								<el-input
								  class="input-new-tag"
								  v-if="inputVisible"
								  v-model="inputValue"
								  ref="saveTagInput"
								  size="small"
								  @keyup.enter.native="handleInputConfirm"
								  @blur="handleInputConfirm"
								>
								</el-input>
								<el-button v-else class="button-new-tag" size="small" 
								@click="showInput">+ New Tag</el-button>
							</template>
						</el-table-column>
						<!-- 索引列 -->
						<el-table-column type="index"></el-table-column>
						<el-table-column label="属性名称" prop="attr_name">
						</el-table-column>
						<el-table-column label="操作">
							<template slot-scope="scope">
								<el-button type="primary" icon="el-icon-edit" 
								size="mini" @click="showEditDialog(scope.row.attr_id)">
								编辑</el-button>
								<el-button type="danger" icon="el-icon-delete"
								 size="mini" @click="remobeParams(scope.row.attr_id)">
								 删除</el-button>
							</template>
						</el-table-column>
					</el-table>
					</el-tab-pane>
			   
			  </el-tabs>
		</el-card>
		<el-dialog :title="'添加' + titleText"
		:visible.sync="addDialogVisible" 
		width="50%" @close="addDialogClosed" >
			<!-- 添加参数区域 -->
			<el-form :model="addForm" :rules="addFormRoles"
			ref="addFormRef" label-width="100px">
				<el-form-item :label="titleText" prop="attr_name">
					<el-input v-model="addForm.attr_name"></el-input>
				</el-form-item>
				
			</el-form>
			<!-- 底部区域 -->
			<span slot="footer" class="dialog-footer">
				<el-button @click="addDialogVisible = false">取消</el-button>
				<el-button type="primary" @click='addParams'>确定</el-button>
			</span>
		</el-dialog>
		<!-- 修改参数的对话框 -->
		<el-dialog :title="'修改' + titleText"
		:visible.sync="editDialogVisible" 
		width="50%" @close="editdialogClosed" >
			<!-- 添加参数区域 -->
			<el-form :model="editForm" :rules="editFormRoles"
			ref="editFormRef" label-width="100px">
				<el-form-item :label="titleText" prop="attr_name">
					<el-input v-model="editForm.attr_name"></el-input>
				</el-form-item>
	
			</el-form>
			<!-- 底部区域 -->
			<span slot="footer" class="dialog-footer">
				<el-button @click="editDialogVisible = false">取消</el-button>
				<el-button type="primary" @click='editParams'>确定</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
export default {
	data() {
		return {
			catelist: [],
			cateProps: {
				expandTrigger: 'hover',
				value: 'cat_id',
				label: 'cat_name',
				children: 'children',
			},
			//级联选择框双向绑定
			selectedCateKeys: [],
			//被激活的页标签名称
			activeName: 'many',
			//动态和静态数据
			manyTableData: [],
			onlyTableData: [],
			addDialogVisible: false,
			addForm: {
				attr_name: ''
			},
			addFormRoles: {
				attr_name: [
					{ required: true, message: '请输入参数名称',
					trigger: 'blur'}
				]
			},
			editFormRoles: {
				attr_name: [
					{ required: true, message: '请输入参数名称',
					trigger: 'blur'}
				]
			},
			editDialogVisible: false,
			editForm: {
				attr_name: ''
			}
		}
	},
	created() {
		this.getCateList()
	},
	methods: {
		async getCateList() {
			const { data: res } = await this.$http.get('categories')
			if(res.meta.status !== 200) {
				return this.$message.error('获取商品分类失败')
			}
			this.catelist = res.data	
			//console.log(res.data)
		},
		handlechange() {
			this.getParamsData()
		},
		//Tab页标签点击事件
		handleTabClick() {
			//console.log(this.activeName)
			this.getParamsData()
		},
		async getParamsData() {
			//console.log(this.selectedCateKeys)
			//选中的不是三级分类
			if(this.selectedCateKeys.length !== 3) {
				this.selectedCateKeys = []
				this.manyTableData = []
				this.onlyTableData = []
			}
			//console.log(this.selectedCateKeys)
			//根据所选分类ID和面板获取对应数据
			const { data: res } = await this.$http.get(
			`categories/${this.cateId}/attributes`,{
				params: { sel: this.activeName }
			})
			
			if(res.meta.status !== 200) {
				this.$message.error('获取参数列表失败')
			}
			//console.log(res.data)
			res.data.forEach(item => {
					item.attr_vals = item.attr_vals ?
					item.attr_vals.split(' ') : [],
					//控制文本框的显示与隐藏
					item.inputVisible = false
					item.inputValue = ''
			})
			//console.log(res.data)
			
			if(this.activeName === 'many') {
				this.manyTableData = res.data
			} else {
				this.onlyTableData = res.data
			}
					
		},
		addDialogClosed() {
			this.$refs.addFormRef.resetFields()
		},
		addParams() {
			this.$refs.addFormRef.validate(async valid => {
				if(!valid) return
				const { data: res } = await this.$http.post(`
				categories/${this.cateId}/attributes`,
				{
					attr_name: this.addForm.attr_name,
					attr_sel: this.activeName
				}
				)
				if(res.meta.status !== 201) {
					this.$message.error('添加参数失败')
				}
				this.$message.success('添加参数成功')
				this.getParamsData()
				this.addDialogVisible = false 
				
			})
		},
		async showEditDialog(attr_id) {
			// 查询当前参数的信息
			const { data: res } = await this.$http.get(`
			categories/${this.cateId}/attributes/${attr_id}`,
			 { params: { attr_sel: this.activeName }
			 })
			 
			 if(res.meta.status !== 200) {
			 	this.$message.error('获取参数信息失败')
			 }
			 this.editForm = res.data
			 console.log(res.data)
			 
			this.editDialogVisible = true
		},
		editdialogClosed() {
			this.$refs.editFormRef.resetFields()
		},
		editParams() {
			this.$refs.editFormRef.validate(async valid => {
				if(!valid) return
				const { data: res } = await this.$http.put(`
				categories/${this.cateId}/attributes
				/${this.editForm.attr_id}`,
				{
					attr_name: this.editForm.attr_name,
					attr_sel: this.activeName
				}
				)
				if(res.meta.status !== 200) {
					this.$message.error('修改参数失败')
				}
				this.$message.success('修改参数成功')
				this.getParamsData()
				this.editDialogVisible = false	
		    })	
		},
		// 删除对应参数项
		async remobeParams(attr_id) {
			const confirmResult =  await this.$confirm(
			'此操作将永久删除该参数, 是否继续?', '提示', {
			          confirmButtonText: '确定',
			          cancelButtonText: '取消',
			          type: 'warning'
			        }).catch(err => {
						return err
					})
					if(confirmResult !== 'confirm') {
						return this.$message.info('已取消删除')
					}
					//没有取消操作
					const {data: res} = await this.$http.delete(`
					categories/${this.cateId}
					/attributes/${attr_id}`)
					
					if(res.meta.status !== 200) {
					 	return this.$message.error('删除参数失败')
					}
					this.$message.success('删除参数成功')
					this.getParamsData()
		},
		async handleInputConfirm(row) {
			if(row.inputValue.trim().length === 0) {
				row.inputValue = ''
				row.inputVisible = false
				return
			}
			//如果没有retrun，则需做后一步处理
			row.attr_vals.push(row.inputValue.trim())
			row.inputValue = ''
			row.inputVisible = false
			this.saveAttrVals(row)
		},
		//对attr_vals的操作保存到数据库
		async saveAttrVals(row) {
			
			//需要请求保存参数信息
			const { data: res } = await this.$http.put(`
			categories/${this.cateId}/attributes/${row.attr_id}`,
			 {
				 attr_name: row.attr_name,
				 attr_sel: row.attr_sel,
				 attr_vals: row.attr_vals.join(' ')
			 })
			 
			 
			 if(res.meta.status !== 200) {
			 	return this.$message.error('修改参数失败')
			 }
			 this.$message.success('修改参数成功')
			 
		},
		showInput(row) {
			row.inputVisible = true
			//让文本框自动获得焦点
			//$nextTick方法：页面重新渲染后调用回调函数中的方法
			this.$nextTick( _ => {
				this.$refs.saveTagInput.$refs.input.focus()
			})
		},
		//删除对应参数可选项
		handleclose(i, row) {
			row.attr_vals.splice(i, 1)
			this.saveAttrVals(row)
		}
	},
	computed: {
		//如果按钮需要被禁用，则返回true，否则false
		isBtnDisabled() {
			if(this.selectedCateKeys.length !== 3) {
				return true
			}
			return false
		},
		//当前选中的三级分类id
		cateId() {
			if(this.selectedCateKeys.length === 3) {
				return this.selectedCateKeys[2]
			}
			return null
		},
		titleText() {
			if(this.activeName === 'many') {
				return '动态参数'
			}
			return '静态属性'
		}	
	}
}
</script>

<style lang="less" scoped>
.cat_opt {
	margin: 15px 0;
} 
.el-tag {
	margin: 10px;
}
.input-new-tag {
	width: 120px;
}
</style>
