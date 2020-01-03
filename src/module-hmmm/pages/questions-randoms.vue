<template>
  <div class="dashboard-container">
    <!-- <div class="app-container">组题列表</div> -->
    <el-card>
      <el-table :data="formData">
        <el-table-column label="序号" prop="id"></el-table-column>
        <el-table-column label="用户名" prop="userName"></el-table-column>
        <el-table-column label="试题" prop="questionIDs"></el-table-column>
        <el-table-column label="答题进度" prop="progressOfAnswer"></el-table-column>
        <el-table-column label="正确率" prop="accuracyRate"></el-table-column>
        <el-table-column label="答题总耗时" prop="totalSeconds"></el-table-column>
        <el-table-column label="组题类型/详情" prop="questionType"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scpoe">
            <el-button type="text" @click="removeRandoms(scpoe.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-row type="flex" justify="center" align="middle" style="height:80px">
        <el-pagination
          background
          @current-change="changePage"
          layout="prev, pager, next"
          :total="page.counts"
          :page-size="page.pagesize"
          :current-page="page.page"
        ></el-pagination>
      </el-row>
    </el-card>
  </div>
</template>

<script>
import { randoms as randomsQuestions, removeRandoms } from "../../api/hmmm/questions";
export default {
  name: "QuestionsRandoms",
  data() {
    return {
      formData: [],
      getDate: {
        
      },
      page: {
        page: 1,
        pagesize: 10,
        counts: 1
      }
    };
  },
  methods: {
    changePage(newPage) {
      this.page.page = newPage;
      console.log(this.page);
      this.getRandoms(this.page);
    },
    async getRandoms(obj) {
      let data = await randomsQuestions(obj);
      this.formData = data.data.items;
      this.page.page = data.data.page;
      this.page.pagesize = data.data.pagesize;
      this.page.counts = data.data.counts;
    },

    async removeRandoms(obj) {
      await this.$confirm("您确定要删除该组题吗？");
      let res = await removeRandoms(obj);
      this.$message({ message: '删除成功', type: 'success' });
      this.getRandoms();
    }
  },
  created() {
    this.getRandoms();
  }
};
</script>

<style scoped>
</style>
