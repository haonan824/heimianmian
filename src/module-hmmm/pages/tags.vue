<template>
  <el-card>
    <el-form>
      <el-form-item style="border-bottom:1px solid #ccc;padding-bottom:20px">
        <el-button type="primary" @click="openDialog">新增标签</el-button>
        <el-button type="primary" @click="toSubject">返回学科</el-button>
      </el-form-item>
      <el-form-item label="标签名称">
        <el-input style="width:20%;margin-right:10px" v-model="searchForm"></el-input>
        <el-button @click="clear">清除</el-button>
        <el-button type="primary" @click="searchTags">搜索</el-button>
      </el-form-item>
    </el-form>
     <el-table
      :data="list"
      style="width: 100%">
      <el-table-column
        prop="id"
        label="序号"
        width="180">
      </el-table-column>
      <el-table-column
        prop="tagName"
        label="标签名称"
        width="180">
      </el-table-column>
      <el-table-column
        prop="creatorID"
        label="创建者">
      </el-table-column>
      <el-table-column
        prop="totals"
        label="面试题数量">
      </el-table-column>
      <el-table-column
        prop="state"
        label="状态" :formatter='formatterStatus'>
      </el-table-column>
      <el-table-column
        label="操作">
        <template slot-scope="obj">
          <!-- {{obj.row}} -->
          <el-button type="text" style="font-size:12px" @click="modify(obj.row.id)">修改</el-button>
        <el-button type="text" style="font-size:12px" @click="changeStatus(obj.row)">{{obj.row.state===1?'启用':'禁用'}}</el-button>
        <el-button type="text" style="font-size:12px" prop="id" @click="delTag(obj.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-row type="flex" justify="end" align="middle" style="height:60px">
         <span style="font-size:14px">共{{page.counts}}条</span>
         <el-pagination
           background
           layout="prev, pager, next"
           :total="page.counts"
           :current-page="page.pages"
           :page-size="page.pagesize"
           @current-change='changePage'>
         </el-pagination>
    </el-row>
    <el-dialog :visible='visible' @close='closeDialog'>
           <el-form label-width="100px" ref="myForm" :model="formDate" :rules="TagsRules">
          <el-form-item label="学科名称" prop="subjectName">
            <el-input style="width:90%" v-model="formDate.subjectName"></el-input>
          </el-form-item>
          <el-form-item label="学科" prop="list_id">
            <el-select v-model="formDate.list_id" placeholder="请选择">
              <el-option
                v-for="item in sublist"
                :key="item.id"
                :label="item.subjectName"
                :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-row type="flex" justify="end">
          <el-form-item>
            <el-button @click="closeDialog">取消</el-button>
            <el-button type="primary" @click="getForm">确定</el-button>
          </el-form-item>
          </el-row>
    </el-form>
    </el-dialog>
  </el-card>
</template>

<script>
import {status} from '../../api/hmmm/constants'
import {list as subjectList} from '../../api/hmmm/subjects'
import {list as TagsList,remove as TagsRemove,add as TagsAdd,removeState,update as updateTags,detail as detailTags} from '../../api/hmmm/tags'
export default {
  name: 'TagsList',
  data() {
    return {
      visible:false,
      list:[],
      page:{
        pages:1,//当前页码
        pagesize:10,//每页几条数据
        counts:0//总条数
      },
       sublist:[],
      formDate:{
        subjectName:'',
        list_id:null,
        id:''
      },
      TagsRules:{
        subjectName:[{required:true,message:'不能为空',trigger:blur}],
        list_id:[{validator:function(rules,value,callback){
        if (value) {
          callback()
        }else{
          callback(new Error('学科不能为空'))
        }
        },trigger:blur}]
      },
      status,//状态
      searchForm:'',//搜索内容
    }
  },
  methods:{
    //获取标签列表
   async getTags(data){
     let result=await TagsList( {
        page: this.page.pages,
        pagesize: this.page.pagesize,
        tagName:data
      })
        // console.log(result);
        this.list=result.data.items
        this.page.counts=result.data.counts
    },
    // 改变页码
    changePage(newPage){
      this.page.pages=newPage
      this.getTags()
    },
    // 打开弹层
    openDialog(){
      this.visible=true
    },
    // 关闭弹层
    closeDialog(){
      this.visible=false
    },
    // 删除标签
    async delTag(id){
      await this.$confirm('确认删除标签吗')
         await TagsRemove({id:id})
           //  alert(id)
           this.$message({
             type:'success',
             message:'删除成功'
           })
           this.getTags()
    },
    // 获取学科
    async getSubject(){
     let result= await subjectList()
        // console.log(result);
        this.sublist=result.data.items
    },
    //增加标签
   async getForm(){
     let value=await this.$refs.myForm.validate();
        if (value) {
          this.formDate.id?await updateTags({id:this.formDate.id,subjectID:this.formDate.list_id,tagName:this.formDate.subjectName})
          :await TagsAdd({subjectID:this.formDate.list_id,tagName:this.formDate.subjectName})
             this.$message({
               type:'success',
               message:'新增成功'
             })
             this.formDate.subjectName='',
             this.formDate.list_id=null
             this.closeDialog()
             this.getTags()
        }
    },
    closeDialog(){
      this.visible=false
    },
    // 状态
    formatterStatus(row, column, cellValue, index){
      let result=this.status.filter(item => item.value === cellValue);
      return result.length?result[0].label:'未知状态'
    },
    // 清除
    clear(){
      this.searchForm=''
    },
    //修改状态
    async changeStatus(row){
        await this.$confirm('是否更改状态')
        await removeState({id:row.id,state:row.state===1?0:1})
        this.getTags()
    },
    // 返回学科
    toSubject(){
      this.$router.push('list')
    },
    // 搜索
    searchTags(){
      //搜索时回到第一页
      this.page.pages=1,
      this.getTags(this.searchForm)
      this.searchForm=''
    },
    // 修改
    async modify(id){
      let result= await detailTags({id:id})
        //  console.log(result);
         this.formDate.subjectName=result.data.tagName
         this.formDate.list_id=result.data.subjectID
         this.formDate.id=result.data.id
       this.openDialog()
    }
  },
  created(){
    this.getTags()
    this.getSubject()
  }
}
</script>

<style scoped>
</style>
