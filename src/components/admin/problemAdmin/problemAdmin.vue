<template>
    <div>

        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 搜索与添加区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.key" clearable @clear="getProblemList"
                              @keyup.enter.native="getProblemList">
                        <el-button slot="append" icon="el-icon-search" @click="getProblemList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="jumpToCreate">添加题目</el-button>
                </el-col>
            </el-row>
            <br>

            <!-- 列表区域 stripe 斑马-->
            <el-table :data="problemlist" border stripe v-loading="loading"
                      :header-cell-style="{'text-align':'center'}"
                      :cell-style="{'text-align':'center'}">
                <!--                索引列-->
                <el-table-column label="ID" prop="id"  min-width="5%"></el-table-column>
                <el-table-column label="标题" prop="title"  min-width="10%"></el-table-column>
                <el-table-column label="创建时间" prop="createTime"   min-width="10%"></el-table-column>
                <el-table-column label="作者" prop="authorName"  min-width="5%"></el-table-column>

                <el-table-column label="可见性"  width="70px">
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.visible" @change="problemVisibleChanged(scope.row.id)">
                        </el-switch>
                    </template>
                </el-table-column>

                <el-table-column label="操作" width="270px">
                    <template slot-scope="scope">
                        <!-- 编辑按钮 -->
                        <!--                        enterable=false表示鼠标进入tooltip区域自动隐藏-->
                        <el-tooltip effect="dark" content="编辑" placement="top" :enterable="false">
<!--                            @click="editProblem(scope.row.id)"-->
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="jumpToEdit(scope.row.id)"></el-button>
                        </el-tooltip>
                      <!-- 上传按钮 -->
                      <el-tooltip effect="dark" content="上传测试用例" placement="top" :enterable="false">
                        <el-button type="warning" size="mini" @click="dialogVisible2 = true;
                        uploadId = scope.row.id;uploadTitle='为题目 '+scope.row.id+' 上传测试用例'" icon="el-icon-upload">
                        </el-button>
                      </el-tooltip>
                      <!-- 下载按钮 -->
                      <el-tooltip effect="dark" content="下载测试用例" placement="top" :enterable="false">
                        <el-button type="success" icon="el-icon-download" size="mini" @click="downloadTestCase(scope.row.id)"></el-button>
                      </el-tooltip>
                      <!-- 删除按钮 -->
                      <el-tooltip effect="dark" content="删除" placement="top" :enterable="false">
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeProblemById(scope.row.id)"></el-button>
                      </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

<!--            上传测试样例-->
            <el-dialog :title=uploadTitle :visible.sync="dialogVisible2" width="335px">
              <el-form :model="form" ref="formRef">
                <el-form-item label-width="0px"
                              ref="uploadElement">
                  <el-upload ref="upload"
                             action="http://localhost:8888/uploadTestCaseById"
                             accept="application/zip"
                             :limit=limitNum
                             :file-list="fileList"
                             :auto-upload="false"
                             :before-upload="handleBeforeUpload"
                             :on-remove="handleRemove"
                             :on-change="imgChange">
                    <el-button size="small" type="primary" >添加文件</el-button>
                  </el-upload>
                </el-form-item>
                <el-form-item>
                  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                  <el-button size="small"
                             type="primary"
                             @click="uploadFile">立即上传</el-button>
                  <el-button size="small"
                             @click="cancel">取消</el-button>
                </el-form-item>
              </el-form>
            </el-dialog>


            <br>
            <!-- 分页区域 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                           :current-page="queryInfo.page" :page-sizes="[5, 10]"
                           :page-size="queryInfo.pre"
                           layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>
    </div>
</template>

<script>
    import qs from 'qs'
    import axios from "axios";
    export default {
        name:"problemAdmin",
        data() {
            return {
                loading: true,
                // 获取用户列表的参数对象
                queryInfo: {
                    // 当前的页数
                    key:"",
                    page: 1,
                    // 当前每页显示多少条数据
                    pre: 5,
                    token: ""
                },
                problemlist:[],
                total: 0,
                limitNum: 1,
                dialogImageUrl: '',
                dialogVisible2:false,
                hideUpload: false,
                myHeaders:{'content-type': 'application/x-www-form-urlencoded'},
                form: {},
                uploadId:0,
                uploadTitle:'',
                fileList: []
            }
        },
        created() {
            if(this.$route.query.page != null)
                this.queryInfo.page = this.$route.query.page - 0;
            this.getProblemList()
        },
        methods: {
            async getProblemList() {
                if(window.localStorage.getItem("token") != null)
                {
                    this.loading = true;
                    this.queryInfo.token = window.localStorage.getItem("token")
                    const { data: res } = await this.$http.get('getProblemListAdmin', {
                        params: this.queryInfo
                    })
                    this.loading = false
                    // console.log(res)
                    if (res.status !== 1) {
                        return this.$message.error('获取题目列表失败！')
                    }
                    this.problemlist = res.data
                    this.total = res.num

                }
                else
                {
                    this.$message.warning("请先登录")
                    this.$router.push('/')
                }
            },
            // 监听 pagesize 改变的事件
            handleSizeChange(newSize) {
                // console.log(newSize)
                this.queryInfo.pre = newSize
                this.getProblemList()
            },
            // 监听 页码值 改变的事件
            handleCurrentChange(newPage) {
                // console.log(newPage)
                this.queryInfo.page = newPage
                this.getProblemList()
            },
            // 监听 switch 开关状态的改变
            async problemVisibleChanged(id) {

                let result =  this.$axios({
                    method: 'put',
                    url: '/changeProblemVisible',
                    headers: { 'content-type': 'application/x-www-form-urlencoded'},
                    data: qs.stringify({
                        id: id,
                        token: window.localStorage.getItem("token")
                    })
                });
                result.then(res=>{
                    var error = res.data.error;
                    if(error === '0')
                    {
                        this.$message.success('更改可见性成功')
                    }
                    else
                        this.$message.warning('越权操作')
                    this.getProblemList()
                })
            },
            // 根据Id删除对应的用户信息
            async removeProblemById(id) {
                // 弹框询问用户是否删除数据
                const confirmResult = await this.$confirm(
                    '此操作将永久删除该题目, 是否继续?',
                    '提示',
                    {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                    }
                ).catch(err => err)

                // 如果用户确认删除，则返回值为字符串 confirm
                // 如果用户取消了删除，则返回值为字符串 cancel
                // console.log(confirmResult)
                if (confirmResult !== 'confirm') {
                    return this.$message.info('已取消删除')
                }

                const { data: res } = await this.$http.delete('deleteProblemById',
                    {params:{
                            id:id,
                            token:window.localStorage.getItem("token")
                        }})
                if (res.error !== "0") {
                    return this.$message.error('删除题目失败！')
                }
                this.$message.success('删除题目成功！')
                await this.getProblemList()
            },
            jumpToCreate()
            {
                this.$router.push({path:'/problemCreate'})
            },
            jumpToEdit(id)
            {
                this.$router.push({path:'/problemEdit',query:{id:id,page:this.queryInfo.page}})
            },
            goFile()
            {
              this.$refs.files.click();
            },
            handleBeforeUpload(file) {
              let flag = 0
              if (!(file.type === 'application/zip')) {
                flag = 1;
                this.$notify.warning({
                  title: '警告',
                  message: '请上传格式为zip的压缩包文件'
                })
                return false
              }
              let size = file.size / 1024 / 1024
              // console.log(size)
              if (size > 10) {
                flag = 1;
                this.$notify.warning({
                  title: '警告',
                  message: '压缩包大小必须小于10M'
                })
                return false
              }
              if(flag === 0) {
                let fd = new FormData();
                fd.append("id", this.uploadId);
                fd.append("file", file);
                fd.append("token",window.localStorage.getItem("token"))
                let result =  this.$axios({
                  method: 'put',
                  url: '/uploadTestCaseById',
                  headers: { 'content-type': 'application/x-www-form-urlencoded'},
                  data: fd
                });
                result.then(res=>{
                  // console.log(res)
                  if(res.data.error === "0")
                  {
                    this.dialogVisible2 = false
                    this.$refs.formRef.resetFields()
                    this.$message.success("上传成功")
                  }
                  else
                    this.$message.error("上传失败")
                })
              }
              return false
            },
            handleRemove(file, fileList) {
              this.hideUpload = fileList.length >= this.limitNum;
            },
            uploadFile() {
              this.$refs.upload.submit()
            },
            imgChange(files, fileList) {
              this.hideUpload = fileList.length >= this.limitNum;
              if (fileList) {
                this.$refs.uploadElement.clearValidate();
              }
            },
            cancel() {
              this.$refs.formRef.resetFields()
              this.dialogVisible2 = false
            },
            async downloadTestCase(id)
            {
              const {data:res} =  await this.$http.get('isTestCaseExistById',{params:{id:id,token:window.localStorage.getItem("token")}})
              if(res.error === "0")
              {
                //有样例
                let target = axios.defaults.baseURL +"downloadTestCaseById?token="+ window.localStorage.getItem("token") + "&id="+ id;
                window.location.href=target;
              }
              else if(res.error === "-1")
              {
                this.$message.warning('越权访问')
                return this.$router.push('/');
              }
              else
              {
                this.$message.warning('该题目暂未上传测试用例')
              }



            }
        }
    }
</script>

<style lang="less" scoped>
</style>
