<template>
	<section>
			<div v-title :data-title="this.$route.name"></div>
	    <el-row class="toolbar">
	    	<el-button type="primary" @click="handleAdd">新增职位</el-button>
	    </el-row>
      <el-table 
	      border 
	      :data="titleList" 
	      v-loading="loading" 
	      highlight-current-row
	      style="width: 100%">
	      <el-table-column type="index" width="55"></el-table-column>
	      <el-table-column prop="id" label="职位编号" sortable></el-table-column>
	      <el-table-column prop="titleName" label="职位名称"></el-table-column>
	      <el-table-column prop="description" label="职位描述"></el-table-column>
	      <el-table-column prop="updateTime" label="更新时间" sortable :formatter="formatTime"></el-table-column>
	      <el-table-column label="操作">
	        <template scope="scope">
	          <!-- <el-button type="primary" size="small" @click="handleDetail(scope.row)">详情</el-button> -->
	          <el-button type="warning" size="small" @click="handleEdit(scope.row)">编辑</el-button>
	          <el-button type="danger" size="small" @click="handleDelete(scope.row)">删除</el-button>
	        </template>
	      </el-table-column>
	    </el-table>
	    <el-row class="m-t">
	      <el-pagination
	        @size-change="handleSizeChange"
	        @current-change="handleCurrentChange"
	        :current-page="pageNo"
	        :page-size="pageSize"
	        :page-sizes="[10, 20, 30, 40]"
	        layout="total, sizes, prev, pager, next, jumper"
	        :total="total"
	        class="pull-right">
	      </el-pagination>
	    </el-row>
	    <el-dialog :visible.sync="titleFormVisible" :title="titleFormTitle">
	    	<el-row>
	    		<el-col :span="18" :offset="3">
			    	<el-form :model="titleForm" ref="titleForm" :rules="rules" label-width="120px">
			    		<el-form-item label="职位名称" prop="titleName">
			    			<el-input v-model="titleForm.titleName" placeholder="职位名称"></el-input>
			    		</el-form-item>
			    		<el-form-item label="职位描述" prop="description">
			    			<el-input type="textarea" v-model="titleForm.description" placeholder="职位描述"></el-input>
			    		</el-form-item>
			    	</el-form>
	    		</el-col>
	    	</el-row>
	    	<div slot="footer">
	    		<el-button @click="titleFormVisible = false">取消</el-button>
	    		<el-button type="primary" @click="submitForm">确定</el-button>
	    	</div>
	    </el-dialog>
	</section>
</template>
<script>
	// import { mapGetters } from 'vuex'
	import { getPartnerTitle, createTitle, updateTitle, delTitle } from '@/api'
	export default {
		data () {
			return {
				pageNo: 1,
				pageSize: 10,
				total: 0,
				loading: false,
				titleList: [],
				titleForm: {
					titleName: '',
					description: '',
				},
				titleFormTitle: '',
				titleFormVisible: false,
				rules: {
					titleName: [
						{ required: true, message: '请输入职位名称', trigger: 'blur'}
					],
					description: [
						{ required: true, message: '请输入职位描述', trigger: 'blur'}
					]
				}
			}
		},
		methods: {
			formatTime(row) {
				return this.$moment(row.updateTime).format('YYYY-MM-DD HH:mm')
			},
			getTitleList() {
				let params = {
					pageNo: this.pageNo,
					pageSize: this.pageSize
				}
				this.loading = true;
				getPartnerTitle(params).then(res => {
					this.loading = false;
					if(res.data.code === '0001') {
						this.titleList = res.data.result.titleList
						this.total = res.data.result.pageInfo.total
					} else {
						this.$message.error(res.data.message)
					}
				}).catch(err => {
					this.loading = false;
					console.log(err)
				})
			},
			// 职位新增
			handleAdd() {
				this.titleForm = {
					id: '',
					titleName: '',
					description: '',
				}
				this.titleFormTitle = '新增职位'
				this.titleFormVisible = true;
			},
			// 职位编辑
			handleEdit(row) {
				this.titleForm = {
					id: row.id,
					titleName: row.titleName,
					description: row.description
				}
				this.titleFormTitle = '职位编辑'
				this.titleFormVisible = true;
			},
			// 新增/编辑提交
			submitForm() {
				this.$refs.titleForm.validate(valid => {
					if(!valid) return;
					let data = Object.assign({}, this.titleForm);
					console.log(data)
					if(data.id) {
						updateTitle(data).then(res => {
							if(res.data.code === '0001') {
								this.$message.success(res.data.message)
								this.getTitleList()
							} else {
								this.$message.error(res.data.message)
							}
						}).catch(err => {
							console.log(err)
							this.$catchError(err)
						})
					} else {
						createTitle(data).then(res => {
							if(res.data.code === '0001') {
								this.$message.success(res.data.message)
								this.getTitleList()
							} else {
								this.$message.error(res.data.message)
							}
						}).catch(err => {
							console.log(err)
							this.$catchError(err)
						})
					}
					this.titleFormVisible = false;
				})
			},
			// 删除职位
			handleDelete(row) {
				this.$confirm(`确定删除 ${row.titleName} 职位？`, '提示', {type: 'warning'}).then(() => {
					let data = {
						id: row.id
					}
					delTitle(data).then(res => {
						if(res.data.code === '0001') {
							this.$message.success(res.data.message)
							this.getTitleList()
						} else {
							this.$message.error(res.data.message)
						}
					}).catch(err => {
						console.log(err)
						this.$catchError(err)
					})
				})
			},
			handleSizeChange(val) {
				this.pageSize = val
				this.getTitleList()
			},
			handleCurrentChange(val) {
				this.pageNo = val
				this.getTitleList()
			}
		},
		mounted() {
			this.getTitleList()
		}
	}
</script>