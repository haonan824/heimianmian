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
        <template v-if="formDate.questionType!='3'">
        <el-form-item  label-width="100px" label="选项(以下选中的选项为正确答案)"></el-form-item>
        <el-form-item style="width:80%;" label-width="50px" prop='options' v-for="(item, index) in formDate.options"
        :label="xuanxiang[index]" :key="index">
        <el-radio v-if="formDate.questionType == '1'" v-model="item.isRight" :label="1" @change="danxuan(index)">{{item.img}}</el-radio>
        <el-checkbox style="margin-right:10px" v-if="formDate.questionType == '2'"
        v-model="item.isRight"
        :true-label="1"
        :false-label="0"
        ></el-checkbox>
        <el-input v-model="item.title" style="width:280px; margin-right:10px"></el-input><el-button @click.prevent="removeDomain(item)">删除</el-button>
        </el-form-item>
        <el-button style="margin-left:100px;margin-bottom:10px" @click="addDomain" type="primary">+新增选项</el-button>
        </template>
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
import {list as subjectlist} from '../../api/hmmm/subjects' //学科
import {list as cataloguelist} from '../../api/hmmm/directorys' //目录
import {list as enterpriselist} from '../../api/hmmm/companys' //企业
import {provinces as citysdatas,citys as cityslist} from '../../api/hmmm/citys' //城市 区
import {direction} from '../../api/hmmm/constants' //方向
import {add} from '../../api/hmmm/questions'//增加
import quillEditor from 'vue-quill-editor'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
export default {
  data() {
    return {
      a:0,
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
        options:[],//选项
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
      question: [{required: true, message: '题干不能为空'}],
      videoURL:[{required: true, message: '解析视频不能为空'}],
      answer:[{required:true,message: '答案解析不能为空'}],
      remarks:[{required:true,message:'题目备注不能为空'}],
      tags:[{required:true,message:'式题标签不能为空'}]
      }
    };
  },
  components:('quill-editor', quillEditor),
  methods:{
    danxuan(index){
        let options = this.formDate.options.map((item, i) => {
        i === index ? (item.isRight = true) : (item.isRight = false);
        return item;
      });
      // console.log(options);
    },
    removeDomain(item) { //删除选项
        var index = this.formDate.options.indexOf(item)
        if (index !== -1) {
          this.formDate.options.splice(index, 1)
        }
      },
      addDomain() { //增加选项
        this.formDate.options.push({
           title:'',
           code:'',
           img:'',
           isRight: false
        });
        debugger
      },
    resetForm(checkout) { //重置
        this.$refs[checkout].resetFields();
      },
    cityss(value){ //点击获取省份
    this.province = cityslist(value)
    },
    increase(){  //增加面试题
       this.$refs.checkout.validate((isOK)=>{
         console.log(this.formDate.options)
        add(this.formDate).then(res=>{
          console.log(res.config.data);
          this.$message({
           message: '添加成功',
            type: 'success'
          });
          this.$router.push('/questions/list')
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
    this.citys =  citysdatas() //获取城市
  },
  created(){
    subjectlist().then(res =>{ //获取学科
      this.subject = res.data.items
    }),
    cataloguelist().then(res =>{ //获取目录
      this.catalogue = res.data.items
    }),
    enterpriselist().then(res =>{ //获取企业
      this.enterprise = res.data.items
    })
  }
}
</script>

<style scoped>
.boss {
  margin: 20px;
  padding: 50px 100px;
  background: #fff;
  box-shadow: 0px 0px 20px #ccc;
}
</style>
