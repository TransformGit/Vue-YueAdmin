<template>
	<section>
		<div v-title :data-title="this.$route.name"></div>
	  <!-- 账号设置 -->
		<el-card class="card-primary" v-loading="loading">
			<div slot="header">账号设置</div>
			<ul class="account-group">
				<li class="account-item">
					<i class="el-icon-circle-check"></i>
					<label>修改密码</label>
					<span>建议您定期更改密码以保护账户安全。</span>
					<el-button size="small" type="primary" @click="modifyPass">修改</el-button>
				</li>
				<li v-if="userInfo.emailVerified === 1" class="account-item">
					<i class="el-icon-circle-check"></i>
					<label>邮箱验证</label>
					<span>验证后，可用于快速找回密码。</span>
					<el-button size="small" type="info" @click="handleVerified">已验证</el-button>
				</li>
				<li v-else class="account-item">
					<i class="el-icon-warning"></i>
					<label>邮箱验证</label>
					<span>验证后，可用于快速找回密码。</span>
					<el-button size="small" type="success" @click="verifyEmail">立即验证</el-button>
				</li>
				<li v-if="userInfo.mobile" class="account-item">
					<i class="el-icon-circle-check"></i>
					<label>修改手机号</label>
					<span>当前手机号： {{mobile}}</span>
					<el-button size="small" type="primary" @click="modifyMobile">修改</el-button>
				</li>
				<li v-else class="account-item">
					<i class="el-icon-warning"></i>
					<label>绑定手机号</label>
					<span>绑定手机号，可用于快速登录账号。</span>
					<el-button size="small" type="success" @click="bindMobile">立即绑定</el-button>
				</li>
			</ul>
		</el-card>
		<!-- 修改密码 -->
		<el-dialog :visible.sync="passwordVisible" title="修改密码">
			<el-row>
				<el-col :span="18" :offset="3">
					<el-form :model="passwordForm" ref="passwordForm" :rules="rules" label-width="120px">
						<el-form-item label="原密码：" prop="oldPass">
							<el-input type="password" v-model.trim="passwordForm.oldPass" placeholder="输入原密码"></el-input>
						</el-form-item>
						<el-form-item label="新密码：" prop="newPass">
							<el-input type="password" v-model.trim="passwordForm.newPass" placeholder="输入新密码"></el-input>
						</el-form-item>
						<el-form-item label="确认密码：" prop="confirmPass">
							<el-input type="password" v-model.trim="passwordForm.confirmPass" placeholder="确认新密码"></el-input>
						</el-form-item>
					</el-form>
				</el-col>
			</el-row>
			<div slot="footer">
				<el-button @click="passwordVisible = false">取消</el-button>
				<el-button type="primary" @click="submitPass">确定</el-button>
			</div>
		</el-dialog>
		<!-- 邮箱验证 -->
		<el-dialog :visible.sync="emailFormVisible" title="激活邮箱">
			<el-row>
				<el-col :span="14" :offset="5">
					<el-form :model="emailForm" ref="emailForm" :rules="rules" label-width="120px">
						<el-form-item v-if="userInfo.email" label="邮箱号：">
							<span>{{userInfo.email}}</span>
						</el-form-item>
						<el-form-item label="激活码：" prop="activeCode">
							<el-row>
								<el-col :span="15">
									<el-input v-model.trim="emailForm.activeCode" placeholder="输入激活码"></el-input>
								</el-col>
								<el-col :span="8" :offset="1">
									<el-button type="success" :disabled="disabled" :loading="emailLoading" @click="getActiveCode">获取激活码</el-button>
								</el-col>
							</el-row>
						</el-form-item>
					</el-form>
				</el-col>
			</el-row>
			<div slot="footer">
				<el-button @click="emailFormVisible = false">取消</el-button>
				<el-button type="primary" @click="activeEmail">确定</el-button>
			</div>
		</el-dialog>
		<!-- 绑定手机号 -->
		<el-dialog :visible.sync="mobileFormVisible" :title="mobileFormTitle">
			<el-row>
				<el-col :span="14" :offset="5">
					<el-form :model="mobileForm" ref="mobileForm" :rules="rules" label-width="120px">
						<el-form-item v-if="!userInfo.mobile" label="手机号：" prop="mobile">
							<el-input v-model.trim="mobileForm.mobile" placeholder="输入手机号" style="width: 90%"></el-input>
						</el-form-item>
						<el-form-item v-if="userInfo.mobile" label="当前手机号：">
							<span>{{mobile}}</span>
						</el-form-item>
						<el-form-item v-if="userInfo.mobile" label="新手机号：" prop="mobile">
							<el-input v-model.trim="mobileForm.mobile" placeholder="输入新手机号" style="width: 90%"></el-input>
						</el-form-item>
						<el-form-item label="验证码：" prop="smsCode">
							<el-row>
								<el-col :span="15">
									<el-input v-model.trim="mobileForm.smsCode" placeholder="输入手机验证码"></el-input>
								</el-col>
								<el-col :span="8" :offset="1">
									<el-button type="success" :disabled="disabled" @click="getSmsCode">{{buttonText}}</el-button>
								</el-col>
							</el-row>
						</el-form-item>
					</el-form>
				</el-col>
			</el-row>
			<div slot="footer">
				<el-button @click="mobileFormVisible = false">取消</el-button>
				<el-button type="primary" @click="submitMobile">确定</el-button>
			</div>
		</el-dialog>
	</section>
</template>
<script>
	import { mapGetters } from 'vuex'
	import Md5 from '@/assets/js/md5'
	import { getMyInfo, updatePwd, getEmailActiveCode, emailActive, getMobileSmsCode, bindMobile } from '@/api'
	export default {
		data () {
			const validatePass = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('请再次输入新密码'));
        }
        if (value !== this.passwordForm.newPass) {
          callback(new Error('两次密码输入不一致'));
        } else {
        	callback()
        }
      }
      const validateMobile = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('请输入手机号'));
        }
        setTimeout(() => {
          if (!value.match(/^(13|14|15|17|18)\d{9}$/)) {
            callback(new Error('请输入正确手机号'));
          } else {
          	callback()
          }
        }, 0);
      }
			return {
				loading: false,
				passwordVisible: false,
				passwordForm: {
					oldPass: '',
					newPass: '',
					confirmPass: ''
				},
				mobileFormVisible: false,
				mobileForm: {
					mobile: '',
					newMobile: '',
					smsCode: '',
				},
				emailFormVisible: false,
				emailForm: {
					activeCode: ''
				},
				rules: {
					oldPass: [
						{ required: true, message: '请输入原密码', trigger: 'blur'}
					],
					newPass: [
						{ required: true, message: '请输入新密码', trigger: 'blur'},
						{ min: 8, max: 20, message: '密码长度在 8 到 20 位之间', trigger: 'blur'},
					],
					confirmPass: [
						{ required: true, validator: validatePass, trigger: 'blur'}
					],
					mobile: [
						{ required: true, validator: validateMobile, trigger: 'blur'}
					],
					smsCode: [
						{ required: true, message: '请输入验证码', trigger: 'blur'}
					],
					activeCode: [
						{ required: true, message: '请输入激活码', trigger: 'blur'}
					],
				},
				disabled: false,
				buttonText: '获取验证码',
				sendEmail: false,
				emailLoading: false,
				mobileFormTitle: '',
			}
		},
		methods: {
			// 获取用户信息
			// getUserInfo() {
			// 	this.loading = true;
			// 	getMyInfo().then(res => {
			// 		this.loading = false;
			// 		if(res.data.code === '0001') {
			// 			this.userInfo = res.data.result.userInfo;
			// 		} else {
			// 			this.$message.error(res.data.message)
			// 		}
			// 	}).catch(err => {
			// 		console.log(err)
			// 		this.loading = false;
			// 		this.$catchError(err)
			// 	})
			// },
			modifyPass() {
				this.passwordVisible = true
			},
			// 修改密码
			submitPass () {
				this.$refs.passwordForm.validate(valid => {
					if(!valid) return;
					let data = {
						oldPassword: Md5.hex_md5(this.passwordForm.oldPass),
						newPassword: Md5.hex_md5(this.passwordForm.newPass),
					}
					updatePwd(data).then(res => {
						if(res.data.code === '0001') {
							this.$message.success(res.data.message)
						} else {
							this.$message.error(res.data.message)
						}
					}).catch(err => {
						console.log(err)
						this.$catchError(err)
					})
					this.passwordVisible = false;
					this.$refs.passwordForm.resetFields()
				})
			},
			handleVerified() {
				this.$message.success(`${this.userInfo.email}邮箱已验证通过`)
			},
			// 验证邮箱
			verifyEmail() {
				this.emailFormVisible = true;
			},
			// 获取邮箱激活码
			getActiveCode() {
				if(this.sendEmail) {
				 this.$notify({ type: 'warning', title: '提示', message: '激活邮件已发送，请注意查收' })
				 return;
				}
				let params = {
					email: this.userInfo.email
				}
				this.emailLoading = true;
				getEmailActiveCode(params).then(res => {
					this.emailLoading = false;
					if(res.data.code === '0001') {
						this.$message(`验证信息已发送到${this.userInfo.email}，请注意查收`)
						this.sendEmail = true;
					} else {
						this.$message.error(res.data.message)
					}
				}).catch(err => {
					this.emailLoading = false;
					console.log(err)
				})
			},
			// 邮箱激活码提交
			activeEmail() {
				this.$refs.emailForm.validateField('activeCode', (errMessage) => {
					if(errMessage) return;
					let data = {
						email: this.userInfo.email,
						activeCode: this.emailForm.activeCode
					}
					emailActive(data).then(res => {
						if(res.data.code === '0001') {
							this.$message.success(res.data.message)
							this.userInfo.emailVerified = 1;
							// this.getUserInfo()
						} else {
							this.$message.error(res.data.message)
						}
					}).catch(err => {
						console.log(err)
						this.$catchError(err)
					})
					this.emailForm.activeCode = '';
					this.emailFormVisible = false;
				})
			},
			bindMobile() {
				this.mobileFormTitle = '绑定手机号';
				this.mobileFormVisible = true; 
			},
			countDown() {
				let count = 60;
				let timer = null;
				this.disabled = true;
				timer = setInterval(() => {
					if(count > 0) {
						this.buttonText = `重新获取 ${count}s`
						count --;
					} else {
						this.buttonText = '重新获取';
						this.disabled = false;
						clearInterval(timer)
					}
				}, 1000)	
			},
			// 获取短信验证码
			getSmsCode() {
				this.$refs.mobileForm.validateField('mobile', (errMessage) => {
					if(errMessage) return;
					this.mobileForm.smsCode = '';
					let params = {
						mobile: this.mobileForm.mobile
					}
					getMobileSmsCode(params).then(res => {
						if(res.data.code === '0001') {
							this.countDown()
							this.$message('短信已发送，请注意查收')
						} else {
							this.$message.error(res.data.message)
						}
					}).catch(err => {
						console.log(err)
						this.$catchError(err)
					})
				})
			},
			// 绑定手机号提交
			submitMobile() {
				this.$refs.mobileForm.validate(valid => {
					if(!valid) return;
					let data = {
						mobile: this.mobileForm.mobile,
						smsCode: this.mobileForm.smsCode
					}
					bindMobile(data).then(res => {
						if(res.data.code === '0001') {
							this.$message(res.data.message)
							this.userInfo.mobile = data.mobile
						} else {
							this.$message.error(res.data.message)
						}
					}).catch(err => {
						console.log(err)
						this.$catchError(err)
					})
					this.mobileForm.smsCode = '';
					this.mobileFormVisible = false;
				})
			},
			modifyMobile() {
				this.mobileFormTitle = '修改手机号';
				this.mobileFormVisible = true;
			}
		},
		computed: {
			...mapGetters([
	  		'userInfo'
	  	]),
			mobile() {
				return this.userInfo.mobile && 
							 this.userInfo.mobile.replace(/(\d{3})\d{4}(\d{4})/, '$1****$2')
			}
		},
		mounted () {
			// this.getUserInfo()
		}
	}
</script>
<style scoped lang="scss">
	.account-group {
		.account-item {
			display: flex;
			padding: 15px 10px;
			line-height: 36px;
			border-bottom: 1px solid #ddd;
			&:last-child {
				border: 0
			}
			i {
				width: 50px;
				padding-top: 6px;
				font-size: 24px;
				&.el-icon-circle-check {
					color: #7ABD54
				}
				&.el-icon-warning {
					color: #FFCC01;
				}
			}
			label {
				width: 130px;
				font-size: 18px
			}
			> span {
				flex: 1;
				padding: 0 15px;
			}
		}
	}
</style>