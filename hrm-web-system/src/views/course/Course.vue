<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :model="filters" :inline="true">
                <el-form-item>
                    <el-input v-model="filters.keywords" placeholder="关键字" ></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getCourses">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="addHandler" >新增</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="success" @click="onLineCourse" >上线</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="danger" @click="offLineCourse" >下线</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表v-loading="listLoading"-->
        <el-table @selection-change="selsChange" @row-click="rowClick" :data="courses" v-loading="listLoading" highlight-current-row  style="width: 100%;">
            <!--多选框-->
            <el-table-column type="selection" width="55">
            </el-table-column>
            <!--索引值,为什么不用id,id不序号-->
            <el-table-column type="index" width="60">
            </el-table-column>
            <!--其他都设置值,只有一个不设置值就自动适应了-->
            <el-table-column prop="name" label="课程名称">
            </el-table-column>
            <el-table-column prop="courseType.name" label="类型">
            </el-table-column>
            <el-table-column prop="gradeName" label="等级">
            </el-table-column>
            <el-table-column prop="status" label="状态" :formatter="statusFormatter">
            </el-table-column>
            <el-table-column prop="users" label="适用人群">
            </el-table-column>
            <el-table-column prop="tenantName" label="所属机构">
            </el-table-column>
            <el-table-column prop="userName" label="创建用户">
            </el-table-column>
            <el-table-column prop="startTime" label="开课时间">
            </el-table-column>
            <el-table-column prop="endTime" label="结课时间">
            </el-table-column>
            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small"  @click="edit(scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="del(scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange"  :page-size="10" :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--新增界面-->
        <el-dialog title="新增" :visible.sync="addFormVisible"  :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px"  ref="addForm">
                <el-form-item label="课程名称" prop="name">
                    <el-input v-model="addForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="适用人群" prop="users">
                    <el-input v-model="addForm.users" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="课程等级" prop="gradeId">
                    <el-radio-group v-model="addForm.gradeId">
                        <el-radio v-for="grade in grades" :label="grade.id">{{grade.name}}</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="收费规则" prop="gradeId">
                    <el-radio-group v-model="addForm.chargeId">
                        <el-radio @change="changeCharge" v-for="charge in charges" :label="charge.id">{{charge.name}}</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="课程价格" prop="price">
                    <el-input :disabled="priceDisabled" v-model="addForm.price" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="咨询QQ" prop="qq">
                    <el-input v-model="addForm.qq" auto-complete="off"></el-input>
                </el-form-item>

                <el-form-item label="课程类型" prop="coursetTypId">
                    <el-cascader
                            :props="courseTypeProps"
                            v-model="addForm.courseTypeId"
                            placeholder="试试搜索：指南"
                            :options="courseTypes"
                            expand-trigger="hover"
                            :show-all-levels="false"
                            filterable
                            change-on-select
                    ></el-cascader>
                </el-form-item>

                <el-form-item prop="logo" label="课程封面">
                    <!--<el-input type="text" v-model="employee.logo" auto-complete="off" placeholder="请输入logo！"></el-input>-->
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:1030/services/common/fastDfs/"
                            :on-preview="handlePreview"
                            :on-remove="handleRemove"
                            :on-success="handleSuccess"
                            :file-list="fileList"
                            list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form-item>


                <el-form-item label="课程简介" prop="description">
                    <el-input
                            type="textarea"
                            :rows="2"
                            placeholder="请输入内容"
                            v-model="addForm.description">
                    </el-input>
                </el-form-item>

                <el-form-item label="课程详情" prop="intro">
                    <div class="edit_container">
                        <quill-editor v-model="addForm.intro" ref="myQuillEditor" class="editer" :options="editorOption" @ready="onEditorReady($event)">
                        </quill-editor>
                    </div>
                </el-form-item>


            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" >提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    import { quillEditor } from "vue-quill-editor"; //调用编辑器
    import "quill/dist/quill.core.css"
    import "quill/dist/quill.snow.css"
    import "quill/dist/quill.bubble.css"

    export default {
        computed: {
            editor() {
                return this.$refs.myQuillEditor.quill
            }
        },
        components: {//使用编辑器
            quillEditor
        },
        data() {
            return {
                row:"",
                courseTypeProps:{
                    value:"id",
                    label:"name"
                },
                priceDisabled:true,
                editorOption: {},//富文本编辑框配置
                fileList:[],
                grades:[
                    {"id":1 , "name":"低级"},
                    {"id":2 , "name":"中级"},
                    {"id":3 , "name":"高级"}
                ],
                charges:[
                    {"id":1 , "name":"免费"},
                    {"id":2 , "name":"收费"}
                ],
                courseTypes:[{
                    value: '4',
                    label: '指南',
                    children: [
                        {
                            value: '1',
                            label: '设计原则',
                            children: [{
                                value: '2',
                                label: '一致'
                            }, {
                                value: '3',
                                label: '反馈'
                            }]
                        }]
                }],
                addFormVisible:false,
                addForm:{
                    name:'',
                    users:'',
                    gradeId:'',
                    courseTypeId:'',
                    description:'',
                    intro:'',
                    chargeId:'',
                    price:'',
                    qq:'',
                    pic:''
                },
                listLoading:false,
                //查询对象
                filters:{
                    keywords:''
                },
                page:1,//当前页,要传递到后台的
                total:0, //分页总数
                courses:[], //当前页数据
            }
        },
        methods: {
            selsChange: function (sels) {
                this.sels = sels;
            },
            handleSuccess(response, file, fileList){
                console.log("===========")
                console.log(response);
                console.log(file);
                console.log(fileList);
                this.addForm.pic = response.resultObj;
            },
            handleRemove(file, fileList) {
                var filePath =file.response.resultObj;
                this.$http.delete("/common/fastDfs/?path="+filePath)
                    .then(res=>{
                        if(res.data.success){
                            this.$message({
                                message: '删除成功!',
                                type: 'success'
                            });
                        }else{
                            this.$message({
                                message: '删除失败!',
                                type: 'error'
                            });
                        }
                    })
            },
            handlePreview(file) {
                console.log(file);
            },
            addSubmit(){
                this.addForm.courseTypeId = this.addForm.courseTypeId[this.addForm.courseTypeId.length - 1];
                /**

                 chargeId: 2
                 description: "1111"
                 gradeId: 1
                 intro: "<p>111<img src="data:image/jpeg;base64,/9j/4AAQSkZ"
                 price: "1111"
                 qq: "111"
                 users: "1"
                 **/

                var course ={
                    //课程基本信息
                    courseTypeId:this.addForm.courseTypeId,
                    name:this.addForm.name,
                    users:this.addForm.users,
                    grade:this.addForm.gradeId,
                    pic:this.addForm.pic,
                    //课程详情
                    courseDetail:{
                        description:this.addForm.description,
                        intro:this.addForm.intro
                    },
                    //课程市场信心
                    courseMarket:{
                        charge:this.addForm.chargeId,
                        qq:this.addForm.qq,
                        price:this.addForm.price
                    }
                };
                this.$http.put("/course/course",course).then(res=>{
                    var ajaxResult = res.data;
                    if(ajaxResult.success){
                        this.$message({
                            message: '保存成功!',
                            type: 'success'
                        });
                        this.addFormVisible = false;
                        this.getCourses();
                    }else{
                        this.$message({
                            message: '保存失败!',
                            type: 'error'
                        });
                    }
                });
            },
            edit(row){
                //this.getEmployees();
                //回显
                let addFormTmp = Object.assign({}, row); //解决对话框改值后列表会被改值.
                this.addForm = addFormTmp; //里面本来就有id,相当于回显了id
                //显示
                this.addFormVisible =true;
            },
            del(row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    var id = row.id;
                    this.listLoading = true;
                    this.$http.delete("/course/course/" + id)
                        .then(result => {
                            this.listLoading = false;
                            //做提示
                            if (result.data.success) {
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                            } else {
                                this.$message({
                                    message: result.data.message,
                                    type: 'error'
                                });
                            }
                            //刷新数据
                            this.getCourses();
                        })
                });

            },
            getCourseTypes(){
                this.$http.get("/course/courseType/treeData").then(result=>{
                    this.courseTypes = result.data;
                });
            },
            changeCharge(chargeId){
                console.log(chargeId);
                if(chargeId === 1){
                    this.priceDisabled = true;
                }else{
                    this.priceDisabled = false;
                }
            },
            onEditorReady(editor) {
                //当富文本编辑框初始化好执行
            },
            addHandler(){
                this.addFormVisible = true;
            },
            handleCurrentChange(curentPage){
                this.page = curentPage;
                this.getCourses();
            },
            getCourses(){
                //发送Ajax请求后台获取数据  axios
                //添加分页条件及高级查询条件
                let para = {
                    "page":this.page,
                    "keyword":this.filters.keywords
                };
                this.listLoading = true; //显示加载圈
                //分页查询
                this.$http.post("/course/course/list",para) //$.Post(.....)
                    .then(result=>{
                        this.total = result.data.total;
                        this.courses = result.data.rows;
                        this.listLoading = false;  //关闭加载圈
                    });
            },
            onLineCourse(){
                //获取选中的行
                if(!this.sels || this.sels.length  <1){
                    this.$message({ message: '老铁，你不选中数据，臣妾下不了啊....',type: 'error'});
                    return;
                }
                //[{id:1,name:zs},{id:2,name:zs}] 把数组里面的某个属性获取出来作为一个数组
                var ids = this.sels.map(item => item.id);
                this.$http.post("/course/course/onLine",ids).then(res=>{
                    var ajaxResult = res.data;
                    if(ajaxResult.success){
                        this.$message({ message: '老铁，上线成功.',type: 'success'});
                        this.getCourses();
                    }else{
                        this.$message({ message: ajaxResult.message,type: 'error'});
                    }
                })
            },
            offLineCourse(){

                //获取选中的行
                if(!this.sels || this.sels.length  <1){
                    this.$message({ message: '老铁，你不选中数据，臣妾下不了啊....',type: 'error'});
                    return;
                }
                //[{id:1,name:zs},{id:2,name:zs}] 把数组里面的某个属性获取出来作为一个数组
                var ids = this.sels.map(item => item.id);
                this.$http.post("/course/course/offLine",ids).then(res=>{
                    var ajaxResult = res.data;
                    if(ajaxResult.success){
                        this.$message({ message: '老铁，下线成功.',type: 'success'});
                        this.getCourses();
                    }else{
                        this.$message({ message: ajaxResult.message,type: 'error'});
                    }
                })
            },
            rowClick(row){
                this.row = row;
                console.log(this.row);
            },
            //性别显示转换
            statusFormatter: function (row, column) {
                return row.status == 1 ? '已上线' : '未上线';
            },
        },
        mounted() {
            this.getCourses();
            this.getCourseTypes();
        }
    }

</script>

<style scoped>

</style>