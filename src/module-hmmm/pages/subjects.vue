<template>
<el-card>
 <el-form class="subjectAdd">
   <el-form-item>
     <el-button type="primary" @click="openDialog">新增学科</el-button>
   </el-form-item>
   <el-form-item label="学科名称">
     <el-input style="width:18.5%" v-model="subjectName"></el-input>
     <el-button style="margin-left:10px" @click="clear()">清除</el-button>
     <el-button type="primary" @click="serach()">搜索</el-button>
   </el-form-item>
 </el-form>
    <el-table
      :data="list"
      style="width: 100%">
      <el-table-column width="90"
        prop="id"
        label="序号"
      >
      </el-table-column>
      <el-table-column width="90"
        prop="subjectName"
        label="学科名称"
      >
      </el-table-column>
      <el-table-column
        prop="" width="80"
        label="创建者">
      </el-table-column>
      <el-table-column
        prop="addDate"
        label="创建日期">
        <template slot-scope="obj">{{obj.row.addDate|parseTimeByString}}</template>
      </el-table-column>
      <el-table-column
        prop="isFrontDisplay"
        width="120"
        label="前台是否显示" :formatter='formatterStatus'>
      </el-table-column>
      <el-table-column
        prop="tags"
        width="80"
        label="二级目录">
      </el-table-column>
      <el-table-column
        prop="totals"
        width="60"
        label="标签">
      </el-table-column>
      <el-table-column
       width="80"
        prop="twoLevelDirectory"
        label="题目数量">
      </el-table-column>
      <el-table-column
        prop="username"
        label="操作">
        <template slot-scope="obj">
        <el-button type="text" >学科分类</el-button>
        <el-button type="text" >学科标签</el-button>
        <!-- {{obj.row}} -->
        <el-button type="text"  @click="toModify(obj.row)">修改</el-button>
        <el-button type="text" @click="DeleteSubject(obj.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  <el-row type="flex" justify="end" style="height:80px" align="middle">
    <span>共{{page.total}}条</span>
  <el-pagination
  background
  layout="prev, pager, next"
  :total="page.total"
       :page-size="page.pageSize"
       :current-page="page.currentPage"
       @current-change='changePage'
  >
</el-pagination>
</el-row>
<el-dialog :visible="dialogVisible" @close="closeDialog">
  <el-form label-width="150px" :model="formData" ref="form" :rules="rules">
    <el-form-item prop="subjectName" label="学科名称">
       <el-input v-model="formData.subjectName" style="width:80%"></el-input>
    </el-form-item>
    <el-form-item label="是否前台显示">
<el-switch
  v-model="value"
  active-color="skyblue"
  inactive-color="#ccc">
</el-switch>
    </el-form-item>
    <el-row type="flex" align="middle" justify="end">
<el-button @click="AddSubjectName" type="primary">确定</el-button>
     <el-button @click="cancel">取消</el-button>
    </el-row>
  </el-form>
</el-dialog>
 </el-card>
</template>
<script>
import {list as SubjectsList,remove as SubjectLremove,add as subjectAdd,detail,update} from'../../api/hmmm/subjects'
export default {
  name: 'SubjectsList',
  data() {
    return {
      value:false,
      list:[],
      dialogVisible: false,
      page: {
        total: 0,
        pageSize: 10,
        currentPage: 1
      },
      formData:{
subjectName:'',
isFrontDisplay:true
      },
      subjectName:'',//搜索
      rules:{
subjectName:[{required:true,message:'学科名称不能为空',trigger:blur}],
      }
    }
  },
methods:{
  formatterStatus(cellValue){
    // console.log(cellValue);
    if (cellValue.isFrontDisplay===1) {
      cellValue.isFrontDisplay='显示'
    }else{
       cellValue.isFrontDisplay='__'
    }
    
  },
  toModify(row){
    this.openDialog()
    detail({id:row.id}).then((result)=>{
      this.formData=result.data
    })
        // update({id:row.id,subjectName:row.subjectName,isFrontDisplay:row.isFrontDisplay}).then(()=>{
        //     //  alert(123)
        // //发布完学科名称清空
        // this.formData={
        //    subjectName:"",
        // }
        // // 发布完弹框自动关闭，数据重新获取，列表重新刷新
        // // this.closeDialog()
        // this.getComment()
        // })
  },
  AddSubjectName(){
    this.$refs.form.validate((isOK)=>{
    if (isOK) {
        if (this.formData.id) {
          update(this.formData).then(()=>{
        // alert(123)
        //发布完学科名称清空
        this.formData={
           subjectName:"",
        }
        // 发布完弹框自动关闭，数据重新获取,列表重新刷新
        this.closeDialog()
        this.getComment()
      })
        }else{
          subjectAdd(this.formData).then(()=>{
        // alert(123)
        //发布完学科名称清空
        this.formData={
           subjectName:"",
        }
        // 发布完弹框自动关闭，数据重新获取,列表重新刷新
        this.closeDialog()
        this.getComment()
      })
        }
      
      }
    })
  },
  cancel(){
    // 点击取消，学科名称清空，是否前台显示初始值，弹框关闭
    this.formData.subjectName='',
    this.formData.isFrontDisplay=true
    this.closeDialog()
  },
  clear(){
  this.subjectName=""
  },
  serach(){
this.page.currentPage=1;
    this.getComment(this.subjectName)
  this.subjectName=""
  },
closeDialog(){
this.dialogVisible=false
},
  openDialog(){
      this.dialogVisible = true//打开弹层
  },
  DeleteSubject(id){
this.$confirm('您是否要删除这个学科?').then(()=>{
  SubjectLremove({id:id}).then(()=>{
this.$message({message:'删除学科成功',
type:'success'})
})
this.getComment()
})
  },
   changePage (page) {
    //  debugger
      this.page.currentPage = page,
      this.getComment()
    },
  getComment(data){
  SubjectsList({page: this.page.currentPage,
          per_page: this.page.pageSize,subjectName:data}).then(result=>{
  console.log(result);
  this.list=result.data.items
  this.page.total = result.data.counts
})
  }
},created(){
  this.getComment()
}

}
</script>

<style lang:'less' scoped>

</style>
