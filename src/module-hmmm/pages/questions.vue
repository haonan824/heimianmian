<template>
  <el-card>
    <div class="choice">
      <el-button type="primary" @click="textPut">新增试题</el-button>
      <el-button type="primary">批量导入</el-button>
    </div>

    <el-form style="margin-top: 30px" :inline="true">
      <el-form-item label="学科">
        <el-select v-model="formData.subjectID">
          <el-option
            v-for="item in subjectList"
            :key="item.id"
            :value="item.id"
            :label="item.subjectName"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="难度">
        <el-select v-model="formData.difficulty">
          <el-option
            v-for="(item, index) in difficulty"
            :key="index"
            :value="item.value"
            :label="item.label"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="试题类型">
        <el-select v-model="formData.questionType">
          <el-option
            v-for="(item, index) in questionType"
            :key="index"
            :value="item.value"
            :label="item.label"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="标签">
        <el-select v-model="formData.tags">
          <el-option
            v-for="(item, index) in tagsList"
            :key="index"
            :value="item.id"
            :label="item.tagName"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="城市">
        <el-select v-model="formData.province" @change="optionChange">
          <el-option v-for="(item, index) in provinces" :key="index" :value="item" :label="item"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="区域">
        <el-select v-model="formData.city">
          <el-option v-for="(item, index) in nowcity" :key="index" :value="item" :label="item"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="关键字">
        <el-input></el-input>
      </el-form-item>
      <el-form-item label="题目备注">
        <el-input></el-input>
      </el-form-item>
      <el-form-item label="企业简称">
        <el-input></el-input>
      </el-form-item>
      <el-form-item label="方向">
        <el-select v-model="formData.direction">
          <el-option v-for="(item, index) in direction" :key="index" :value="item" :label="item"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="录入人">
        <el-select v-model="formData.creatorID">
          <el-option
            v-for="item in writemans"
            :key="item.id"
            :value="item.id"
            :label="item.username"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级目录">
        <el-select v-model="formData.catalogID">
          <el-option
            v-for="(item, index) in menuLists"
            :key="index"
            :value="item.value"
            :label="item.label"
          ></el-option>
        </el-select>
      </el-form-item>
      <!-- 做预览 -->
      <el-row style="margin-left: 50px; margin-top: 30px">
        <el-button @click="clearCondition">清除</el-button>
        <el-button type="primary">搜索</el-button>
      </el-row>
    </el-form>
    <el-table style="margin-top: 40px" :data="dataList">
      <el-table-column prop="id" label="序号" width="50px"></el-table-column>
      <el-table-column prop="number" label="试题编号"></el-table-column>
      <el-table-column prop="subjectID" label="学科" width="100"></el-table-column>
      <el-table-column prop="questionType" label="题型" width="110"></el-table-column>
      <el-table-column prop="question" label="题干"></el-table-column>
      <el-table-column prop="addDate" label="录入时间" width="200"></el-table-column>
      <el-table-column prop="difficulty" label="难度"></el-table-column>
      <el-table-column prop="creator" label="录入人"></el-table-column>
      <el-table-column prop label="操作" width="210" type="flex" justify-content="space-around">
        <template slot-scope="scope">
          <el-link type="primary" style="margin-left: 7px; font-size: 12px"  @click="textdetail(scope.row)">预览</el-link>
          <el-link type="primary" style="margin-left: 7px; font-size: 12px" @click="textPut(scope.row.id)">修改</el-link>
          <el-link type="primary" style="margin-left: 7px; font-size: 12px" @click="deletetext(scope.row)">删除</el-link>
          <el-link type="primary" style="margin-left: 7px; font-size: 12px" @click="joinChoice">加入精选</el-link>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="题目预览" :visible.sync="dialogVisible" width="75%" :before-close="handleClose" :show-close= 'false' >
      <span class="section">
        <span>[题库]：</span>
        <span>{{detailtext.question}}</span>
      </span>
      <span class="section">
        <span>[学科]：</span>
        <span>{{detailtext.subjectID}}</span>
      </span>
      <span class="section">
        <span>[难度]：</span>
        <span>{{detailtext.difficulty}}</span>
      </span>
      <span class="section">
        <span>[标签]：</span>
        <span>{{detailtext.tags}}</span>
      </span>
      <span class="section">
        <span>[题型]：</span>        
        <span>{{detailtext.questionType}}</span>
      </span>
      <span class="section">
        <span>[编号]：</span>        
        <span>{{detailtext.id}}</span>
      </span>
      <span class="section">
        <span>[方向]：</span>        
        <span>{{detailtext.direction}}</span>
      </span>
      <span class="section">
        <span>[城市]：</span>        
        <span>{{detailtext.city}}</span>
      </span>
      <span class="section">
        <span>[目录]：</span>        
        <span>{{detailtext.catalogID}}</span>
      </span>
      <span class="section">
        <span>[企业]：</span>        
        <span>{{detailtext.enterpriseID}}</span>
      </span>
    
      <span slot="footer" class="dialog-footer" >
        <el-button type="primary" @click="dialogVisible = false" style="margin-left: 85%">关闭</el-button>
      </span>
    </el-dialog>
    
    <el-row type="flex" style="height: 60px" justify="end" align="middle">
      <el-pagination
        background
        layout="prev, pager, next"
        :total="page.total"
        :page-size="page.pageSize"
        :current-page="page.currentPage"
        @current-change="changePage"
      ></el-pagination>
    </el-row>
  </el-card>
</template>

<script>
import { list as basis, detail as textdetail, update, remove} from "../../api/hmmm/questions";
import { detail as question } from "../../api/hmmm/questions";
import { list as subjectList } from "./../../api/hmmm/subjects";
import { difficulty, questionType, direction } from "../../api/hmmm/constants";
import { list as tagsList } from "../../api/hmmm/tags";
import { provinces, citys } from "../../api/hmmm/citys";
import { simple } from "../../api/base/users";
import { simple as menuList } from "../../api/hmmm/directorys";

export default {
  data() {
    return {
      detailtext: {},
      dialogVisible: false,
      menuLists: [],
      writemans: [],
      direction,
      nowcity: [],
      citys,
      provinces: [],
      tagsList: [],
      questionType,
      difficulty,
      subjectList: [],
      dataList: [],
      page: {
        total: 0,
        pageSize: 4,
        currentPage: 1
      },
      formData: {
        subjectID: "",
        difficulty: "",
        questionType: "",
        tags: "",
        province: "",
        city: "",
        direction: "",
        creatorID: "",
        catalogID: "",
        keyword: ""
      }
    };
  },

  methods: {

    handleClose(done) {
        this.$confirm('确认关闭？')
          .then(_ => {
            done();
          })
      },
   
    optionChange(newValue) {
      this.nowcitys(newValue);
      this.formData.city = ''
    },
    changePage(newPage) {
      this.page.currentPage = newPage;
      this.getDataList();
    },
    async getDataList() {
      let res = await basis({
        page: this.page.currentPage,
        pagesize: this.page.pageSize
      });
      this.dataList = res.data.items;
      this.page.total = res.data.counts;
      console.log(res);
    },
    async getSubjectList() {
      let res = await subjectList();
      this.subjectList = res.data.items;
    },
    async getTagsList() {
      let res = await tagsList();
      this.tagsList = res.data.items;
    },
    async getProvince() {
      this.provinces = await provinces();
    },
    async nowcitys(params) {
      this.nowcity = await citys(params);
    },
    async writeman() {
      let res = await simple();
      this.writemans = res.data;
    },
    async listmulu() {
      let res = await menuList();
      this.menuLists = res.data;
    },
    async textdetail(id) {
      this.dialogVisible = true
      let res = await textdetail(id)
      this.detailtext = res.data
      console.log(res.data)
    },
    async textPut(id) {
      this.$router.push(`new?id=${id}`)
    },
    async deletetext(data) {
      await this.$confirm('你确定要删除吗？')
      let res = await remove(data)
      this.getDataList()
    },
    joinChoice() {
      this.$confirm('你确定要加入精选吗？').then(() => {
        this.$message({
          message: '加入精选成功',
          type: 'success'
        })
      })
    },
    clearCondition() {
      this.formData = {
        subjectID: "",
        difficulty: "",
        questionType: "",
        tags: "",
        province: "",
        city: "",
        direction: "",
        creatorID: "",
        catalogID: "",
        keyword: ""
      }
    }
  },
  created() {
    this.getDataList();
    this.getSubjectList();
    this.getTagsList();
    this.getProvince();
    this.writeman();
    this.listmulu();
  }
};
</script>

<style lang="scss">
.choice {
  height: 60px;
  border-bottom: 1px solid #ccc;
}
.el-form-item__label {
  margin-left: 50px;
}
.section {
 
    display: inline-block;
    width: 23%;
}
</style>
