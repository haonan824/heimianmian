<template>
  <div class="dashboard-container">
    <el-card>
      <template slot="header">
        <el-button type="primary" @click="newQuestions">新增试题</el-button>
        <el-button type="primary">批量导入</el-button>
      </template>
      <!-- 查询表单 -->
      <el-form :model="formData.params" label-width="80px" :inline="true">
        <el-form-item label="学科">
          <el-select v-model="formData.params.subjectID" placeholder="请选择">
            <el-option
              :label="item.label"
              :value="item.value"
              v-for="item in subjectsList"
              :key="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model="formData.params.chkState" placeholder="请选择">
            <el-option label="待审核" :value="0"></el-option>
            <el-option label="通过" :value="1"></el-option>
            <el-option label="拒绝" :value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="难度">
          <el-select v-model="formData.params.difficulty" placeholder="请选择">
            <el-option :label="item.label" :value="item.value" v-for="item in difficulty" :key="item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="试题类型">
          <el-select v-model="formData.params.questionType" placeholder="请选择">
            <el-option :label="item.label" :value="item.value" v-for="item in questionType" :key="item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="标签">
          <el-select v-model="formData.params.tags" placeholder="请选择">
            <el-option
              :label="item.label"
              :value="item.value"
              v-for="item in tags"
              :key="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="关键字">
          <el-input v-model.lazy="formData.params.keyword"></el-input>
        </el-form-item>
        <el-form-item label="题目备注">
          <el-input v-model.lazy="formData.params.remarks"></el-input>
        </el-form-item>
        <el-form-item label="企业简称">
          <el-input v-model.lazy="formData.params.shortName"></el-input>
        </el-form-item>
        <el-form-item label="录入人">
          <el-select v-model="formData.params.creatorID" placeholder="请选择">
            <el-option label="超级管理员" value="1"></el-option>
            <el-option label="编辑" value="2"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="二级目录">
          <el-select v-model="formData.params.catalogID" placeholder="请选择">
            <el-option :label="item.label" :value="item.value" v-for="item in catalog" :key="item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="城市">
          <el-select v-model="formData.params.province" @change="switchCity" placeholder="请选择">
            <el-option :label="item" :value="item" v-for="(item,index) in provinces" :key="index"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="区域">
          <el-select v-model="formData.params.city" placeholder="请选择">
            <el-option :label="item" :value="item" v-for="(item,index) in citys" :key="index"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="方向">
          <el-select v-model="formData.params.direction" placeholder="请选择">
            <el-option :label="item" :value="index" v-for="(item,index) in direction" :key="index"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <el-row type="flex" justify="center">
        <el-button @click="empty">清除</el-button>
        <el-button type="primary" @click="search">搜索</el-button>
      </el-row>
      <!-- 发布状态 -->
      <el-row>
        <el-radio-group v-model="publishState" style="margin-bottom: 30px;" @change="publishStates">
          <el-radio-button :label="-1">全部</el-radio-button>
          <el-radio-button :label="0">待发布</el-radio-button>
          <el-radio-button :label="1">已发布</el-radio-button>
        </el-radio-group>
      </el-row>
      <!-- 表格数据 -->
      <el-table :data="itemsList">
        <el-table-column label="序号" prop="id"></el-table-column>
        <el-table-column label="试题编号" prop="number" width="140px"></el-table-column>
        <el-table-column label="学科" prop="subjectID">
          <template slot-scope="scpoe">
            <span v-if="scpoe.row.subjectID == 1">java</span>
            <span v-else-if="scpoe.row.subjectID == 2">javascript</span>
            <span v-else-if="scpoe.row.subjectID == 3">C++</span>
            <span v-else>其他</span>
          </template>
        </el-table-column>
        <el-table-column label="题型" prop="questionType">
          <template slot-scope="scpoe">
            <span v-if="scpoe.row.questionType == 1">单选</span>
            <span v-else-if="scpoe.row.questionType == 2">多选</span>
            <span v-else-if="scpoe.row.questionType == 3">简答</span>
            <span v-else>其他</span>
          </template>
        </el-table-column>
        <el-table-column label="题干" prop="question"></el-table-column>
        <el-table-column label="录入时间" prop="addDate" width="160px">
          <template slot-scope="scpoe">
            <span>{{ scpoe.row.addDate | parseTime }}</span>
          </template>
        </el-table-column>
        <el-table-column label="录入人" prop="creator" width="100px"></el-table-column>
        <el-table-column label="难度" prop="difficulty">
          <template slot-scope="scope">
            <span v-if="scope.row.difficulty == 0">简单</span>
            <span v-else-if="scope.row.difficulty == 1">一般</span>
            <span v-else-if="scope.row.difficulty == 2">困难</span>
            <span v-else-if="scope.row.difficulty == 3">烧脑</span>
            <span v-else>其他</span>
          </template>
        </el-table-column>
        <el-table-column label="审核状态" prop="chkState">
          <template slot-scope="scope">
            <!-- <span>{{ scope.row.chkState }}</span> -->
            <span v-if="scope.row.chkState == 0">待审核</span>
            <span v-else-if="scope.row.chkState == 1">通过</span>
            <span v-else-if="scope.row.chkState == 2">拒绝</span>
          </template>
        </el-table-column>
        <el-table-column label="审核意见" prop="chkRemarks"></el-table-column>
        <el-table-column label="审核人" prop="chkUser"></el-table-column>
        <el-table-column label="发布状态" prop="publishState">
          <template slot-scope="scope">
            <span v-if="scope.row.publishState == 0">待发布</span>
            <span v-else-if="scope.row.publishState == 1">已发布</span>
            <span v-if="scope.row.publishState == 2">已下架</span>
          </template>
        </el-table-column>
        <el-table-column label="操作" prop="st" width="200px">
          <template>
            <span>审核</span>
            <span>预览</span>
            <span>上架</span>
            <span>修改</span>
            <span>删除</span>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-row type="flex" justify="center" align="middle" style="height:80px">
        <span style="font-size:12px;letter-spacing:7px">共{{ page.counts }}条</span>
        <el-pagination
          background
          layout="prev, pager, next"
          @current-change="pageChange"
          :total="page.counts"
          :page-size="formData.params.pageSize"
          :current-page="formData.params.page"
        ></el-pagination>
        {{page.page}}
      </el-row>
    </el-card>
  </div>
</template>

<script>
import { choice as ChoiceQuestions } from "../../api/hmmm/questions";
import { provinces, citys } from "../../api/hmmm/citys";
import { simple as subjectsList } from "../../api/hmmm/subjects";
import { parseTime } from "../../filters";
import { simple as simpleTags } from "../../api/hmmm/tags";
import {
  difficulty,
  questionType,
  direction,
  chkType,
  publishType
} from "../../api/hmmm/constants";
import { simple as simpleDirectorys } from "../../api/hmmm/directorys";
export default {
  // name: "QuestionsChoice",
  data() {
    return {
      provinces: provinces(), // 城市
      citys: [], // 城区
      subjectsList: [], // 学科
      tags: [], // 标签
      difficulty, // 难度
      questionType, // 题型
      direction, // 方向
      chkType, // 状态
      publishType, // 发布状态
      catalog: [], // 二级目录
      publishState: -1,
      // form表单数据
      formData: {
        params: {
          page: 1,
          pageSize: 10,
          subjectID: null,
          chkState: null,
          difficulty: null,
          questionType: null,
          tags: null,
          remarks: null,
          keyword: null,
          shortName: null,
          creatorID: null,
          catalogID: null,
          province: null,
          city: null,
          direction: null
        }
      },

      page: {
        counts: 1, // 总记录数
        pagesize: 10, // 页大小
        pages: 1, // 总页
        page: 1 // 当前页
      },

      itemsList: [] //内容
    };
  },
  methods: {
    async screening() {
      let tgasData = await simpleTags();
      this.tags = tgasData.data;
      let catalog = await simpleDirectorys();
      this.catalog = catalog.data
      
    },
    // 发布状态
    async publishStates() {
      await this.GetChoice();
      let list = this.itemsList.filter(item => {
        if (
          item.publishState === this.publishState &&
          this.publishState != -1
        ) {
          return item;
        } else if (this.publishState === -1) {
          return item;
        }
      });
      this.itemsList = list;
    },

    newQuestions() {
      this.$router.push("new");
    },
    // 翻页
    pageChange(newPage) {
      console.log(newPage);
      this.formData.params.page = newPage;
      this.GetChoice(this.formData.params);
    },
    search() {
      this.formData.params.page = 1;

      this.GetChoice(this.formData.params);
    },
    empty() {
      this.formData.params = {};
      this.GetChoice();
      this.switchCity();
    },
    // 内容
    async GetChoice(data) {
      let choice = await ChoiceQuestions(data);
      this.itemsList = choice.data.items;
      this.page = choice.data;
    },
    // 学科列表获取
    async GetSubjectsList() {
      let subjects = await subjectsList();
      this.subjectsList = subjects.data;
    },
    // 省市下级市区
    switchCity(city) {
      this.citys = citys(city);
    }
  },

  created() {
    this.GetChoice();
    this.GetSubjectsList();
    this.screening();
  }
};
</script>

<style scoped lang="scss">
// 1. 如果在此处 scoped 当前组件下生效样式
// 2. 在style中的所有选择器 编译的时候会自动带上属性选择器
// 3. .box{} ===> .box[data-v-2c9827a4]{} 交集选择器
// 4. 在当前组件下暴露的标签都会加上 data-v-2c9827a4 属性
// 5. 但是如果是组件，其他组件内部的标签是不会加上这个属性 意味写组件内部的样式是不会生效的
// 6. 其他组件的样式都不会生效
</style>
