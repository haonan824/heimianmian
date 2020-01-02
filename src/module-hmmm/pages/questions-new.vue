<template>
  <div class="dashboard-container">
    <!-- <div class="app-container">题库添加</div> -->
    <el-row class="boss">
      <el-form ref='checkout' :model='formDate' :rules="regulation"> 
        <el-form-item  prop='subjectID' style="width:80%;" label-width="100px" label="学课">
          <el-select v-model="formDate.subjectID" style="margin-left:10px"  placeholder="请选择">
            <el-option v-for="itme in subject" :key='itme.id' :label="itme.subjectName" :value="itme.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item style="width:80%;" prop='catalogID' label-width="100px" label="目录">
          <el-select v-model="formDate.catalogID" style="margin-left:10px" placeholder="请选择">
            <el-option v-for="itme in catalogue" :key='itme.id' :label='itme.directoryName' :value='itme.id'></el-option>
          </el-select>
        </el-form-item>
        <el-form-item style="width:80%;" prop='enterpriseID' label-width="100px" label="企业">
          <el-select v-model="formDate.enterpriseID" style="margin-left:10px" placeholder="请选择">
            <el-option v-for='itme in enterprise' :key="itme.id" :label='itme.shortName' :value='itme.id'></el-option>
          </el-select>
        </el-form-item>
        <el-form-item style="width:80%;" prop='province' label-width="100px" label="市级">
          <el-select v-model="formDate.province" style="margin-left:10px" placeholder="请选择" @change='cityss'>
            <el-option v-for="(itme,index) in citys" :key='index' :label='itme' :value='itme'></el-option>
          </el-select>
        </el-form-item>
        <el-form-item prop='city' style="width:80%;" label-width="100px" label="县级">
       <el-select v-model="formDate.city" style="margin-left:12px" placeholder="请先选择市级">
            <el-option v-for="(itme,index) in province" :key='index' :label='itme' :value='itme'></el-option>
          </el-select>
        </el-form-item>
        <el-form-item style="width:80%;" prop='direction' label-width="100px" label="方向">
          <el-select v-model="formDate.direction" style="margin-left:10px" placeholder="请选择">
            <el-option v-for="(itme,index) in direction" :key="index" :label='itme' :value='itme'></el-option>
          </el-select>
        </el-form-item>
        <el-form-item style="width:80%;" prop='pattern' label-width="100px" label="题型">
          <el-radio-group v-model="formDate.questionType">
            <el-radio label="1">单选</el-radio>
            <el-radio label="2">多选</el-radio>
            <el-radio label="3">简答</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item style="width:80%;" prop='difficulty' label-width="100px" label="难度">
          <el-radio-group v-model="formDate.difficulty">
            <el-radio label="1">简单</el-radio>
            <el-radio label="2">一般</el-radio>
            <el-radio label="3">困难</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item  style="width:80%;height:480px" prop='question' label-width="100px" label="题干">
          <quill-editor v-model="formDate.question" style="height:300px"  type="textarea" :rows="2" placeholder="请输入内容"></quill-editor>
        </el-form-item>
        <el-form-item style="width:80%;" label-width="100px" label="选项(以下选中的选项为正确答案)"></el-form-item>
        <el-form-item
        style="width:80%;" label-width="100px"
        prop='options'
          v-for="(time, index) in formDate.options"
          :label="xuanxiang[index]"
          :key="index"
        >
    <el-input v-model="time.title" style="width:300px; margin-right:10px"></el-input><el-button @click.prevent="removeDomain(time)">删除</el-button>
  </el-form-item>
  <el-button style="margin-left:100px;margin-bottom:10px" @click="addDomain" type="primary">+新增选项</el-button>
        <el-form-item  style="width:80%;" label-width="100px" label="解析视频" prop='videoURL'>
          <el-input  v-model='formDate.videoURL'  placeholder="请输入内容"></el-input>
        </el-form-item>
        <el-form-item style="width:80%;height:480px" prop='answer' label-width="100px" label="答案解析">
          <quill-editor style="height:300px"  v-model='formDate.answer'  type="textarea" :rows="2" placeholder="请输入内容"></quill-editor>
        </el-form-item>
        <el-form-item style="width:80%;" prop='remarks' label-width="100px" label="题目备注">
          <el-input  v-model='formDate.remarks' placeholder="请输入内容"></el-input>
        </el-form-item>
        <el-form-item style="width:80%;height:480px" prop='tags' label-width="100px" label="试题标签">
          <quill-editor style="height:300px"  v-model='formDate.tags' type="textarea" :rows="2" placeholder="请输入内容"></quill-editor>
        </el-form-item>
          <el-button type="primary" @click="increase">保存</el-button>
          <el-button @click="resetForm('checkout')">清空</el-button>
      </el-form>
    </el-row>
  </div>
</template>

<script>
import {list as subjectlist} from '../../api/hmmm/subjects'
import {list as cataloguelist} from '../../api/hmmm/directorys'
import {list as enterpriselist} from '../../api/hmmm/companys'
import {provinces as citysdatas,citys as cityslist} from '../../api/hmmm/citys'
import {direction} from '../../api/hmmm/constants'
import {add} from '../../api/hmmm/questions'
import quillEditor from 'vue-quill-editor'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
export default {
  data() {
    return {
      xuanxiang:['A','B','C','D','E','F','G','H','Y','J','K','L'],
      subject:[],//学科
      catalogue:[],//目录
      enterprise:[],//企业
      citys:[],//城市
      province:[],//省
      direction:[],//方向
      pattern:null,//题型
      formDate:{
        subjectID:null, //学科id
        catalogID:null, //目录id
        enterpriseID:null,//企业id
        direction:null,//方向
        city:null,//市
        province:null,//省份
        questionType:'1',//题型
        difficulty:'1',//难度
        question:null,//题干
        videoURL:null,//解析视频
        answer:null,//答案解析
        remarks:null,//题目备注
        tags:null,//式题标签
        options:[{ title: '',code:'',img:'',isRight:true}],
      },
      regulation: {
      options:[{ required: true, message: '内容不能为空'}],
      subjectID: [{ required: true, message: '学科不能为空' }],
      catalogID: [{ required: true, message: '目录不能为空' }],
      city: [{ required: true, message: '城市不能为空' }],
      province: [{ required: true, message: '省份不能为空' }],
      direction: [{ required: true, message: '方向不能为空' }],
      enterpriseID: [{ required: true, message: '企业不能为空' }],
      questionType: [{required: true, message: '题型不能为空'}],
      difficulty: [{required: true, message: '难度不能为空'}],
      question: [{required: true, message: '题干不能为空'},{min: 5, max: 30, message: '题干长度需要在5到30字符之间'}],
      videoURL:[{required: true, message: '解析视频不能为空'},{min: 5,max: 30, message: '解析视频长度需要在5到30字符之间'}],
      answer:[{required:true,message: '答案解析不能为空'},{min: 5,max: 30, message: '答案解析长度需要在5到30字符之间'}],
      remarks:[{required:true,message:'题目备注不能为空'},{min: 5,max: 30, message: '题目备注长度需要在5到30字符之间'}],
      tags:[{required:true,message:'式题标签不能为空'},{min: 5, max: 30, message: '式题标签长度需要在5到30字符之间'}]
      }
    };
  },
  components:('quill-editor', quillEditor),
  methods:{
    removeDomain(item) {
        var index = this.formDate.options.indexOf(item)
        if (index !== -1) {
          this.formDate.options.splice(index, 1)
        }
      },
      addDomain() {
        this.formDate.options.push({
          value: '',
          key: Date.now()
        });
      },
    resetForm(checkout) {
        this.$refs[checkout].resetFields();
      },
    cityss(value){ //点击获取省份
    this.province = cityslist(value)
    },
    increase(){
       this.$refs.checkout.validate((isOK)=>{
         console.log(this.formDate.options)
        add(this.formDate).then(res=>{
          console.log(res);
          this.$message({
           message: '添加成功',
            type: 'success'
          });
          // this.$router.push('/questions/list')
       })
      })
    }
  },
  watch:{ //监听省份的改变
    'formDate.citys': function( newValue ){
        this.formDate.province=''
    }
  },
  mounted(){  //文档被挂载后
    this.direction = direction
    this.citys =  citysdatas()
  },
  created(){
    subjectlist().then(res =>{
      this.subject = res.data.items
    }),
    cataloguelist().then(res =>{
      this.catalogue = res.data.items
    }),
    enterpriselist().then(res =>{
      this.enterprise = res.data.items
    })
  }
};
</script>

<style scoped>
.boss {
  margin: 20px;
  padding: 50px 100px;
  background: #fff;
  box-shadow: 0px 0px 20px #ccc;
}
</style>
