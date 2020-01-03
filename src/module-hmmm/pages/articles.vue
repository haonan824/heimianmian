<template>
  <el-card>
    <div slot="header">
      <el-button type="primary" @click="dialogFormVisible = true">新增面试技巧</el-button>
    </div>
    <el-form>
      <el-form-item label="关键字">
        <el-input v-model="keyword" placeholder="请输入内容" class="inputWidth"></el-input>
        <el-button @click="clear">清除</el-button>
        <el-button @click="search" type="primary">搜索</el-button>
      </el-form-item>
      <el-table :data="tableData">
        <el-table-column prop="id" label="序号"></el-table-column>
        <el-table-column prop="title" label="标题"></el-table-column>
        <el-table-column prop="reads" label="阅读数"></el-table-column>
        <el-table-column :formatter="formatterState" prop="state" label="状态"></el-table-column>
        <el-table-column prop="creator" label="录入人"></el-table-column>
        <el-table-column prop label="操作">
          <template slot-scope="obj">
            <el-button type="text" size="small">预览</el-button>
            <el-button type="text" size="small" @click="modify(obj.row)">修改</el-button>
            <el-button type="text" size="small" @click="delArticle(obj.row)">删除</el-button>
            <el-button
              type="text"
              size="small"
              @click="changeState(obj.row)"
            >{{ obj.row.state ? '禁用' : '启用'}}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-row type="flex" justify="end" align="middle" style="height:80px">
        <el-pagination
          background
          layout="prev, pager, next"
          :current-page="page.currentPage"
          :page-size="page.pageSize"
          :total="page.total"
          @current-change="changePage"
        ></el-pagination>
      </el-row>
    </el-form>
    <el-dialog :visible="dialogFormVisible" :show-close="false">
      <el-form ref="myForm" style="margin-left:40px" :model="formData" :rules="publishRules">
        <el-form-item label="学科名称" prop="title">
          <el-input style="width:80%" v-model="formData.title"></el-input>
        </el-form-item>
        <el-form-item label="内容" prop="articleBody">
          <quill-editor
            style="height:200px; width:600px;margin-left:67px;"
            v-model="formData.articleBody"
          ></quill-editor>
        </el-form-item>
        <el-form-item label="视频地址" prop="videoURL" style="margin-top:150px">
          <el-input style="width:80%" v-model="formData.videoURL"></el-input>
        </el-form-item>
      </el-form>
      <el-row slot="footer" type="flex" justify="end">
        <el-button @click="btnOK" type="primary">确定</el-button>
        <el-button @click="btnCancel">取消</el-button>
      </el-row>
    </el-dialog>
  </el-card>
</template>
<script>
import Vue from "vue";
import { quillEditor } from "vue-quill-editor"; // 引入了quill组件
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
import {
  list as getArticles,
  state as changeSa,
  remove as removeArticle,
  add as addArticle,
  update,
  detail
} from "../../api/hmmm/articles";
Vue.component("quill-editor", quillEditor);
export default {
  name: "ArticlesList",
  data() {
    return {
      formData: {
        title: "", // 标题
        articleBody: "", // 文章内容
        cover: "" //视频地址
      },
      tableData: [],
      dialogFormVisible: false, //弹层
      page: {
        // 专门放置分页数据
        total: 0, // 数据总条数
        pageSize: 10, // 默认每页10条
        currentPage: 1 // 当前页码 默认第一页
      },
      keyword: "", // 内容
      publishRules: {
        title: [
          { required: true, message: "学科名称不能为空", trigger: "blur" }
        ],
        articleBody: [
          { required: true, message: "文章内容不能为空", trigger: "blur" }
        ],
        videoURL: [{ required: true, message: "视频不能为空", trigger: "blur" }]
      }
    };
  },
  methods: {
    //文章修改
    modify(row) {
      detail({ id: row.id }).then(res => {
        this.dialogFormVisible = true;
        this.formData = res.data;
      });
    },
    //文章搜索
    search() {
      this.getList({ keyword: this.keyword });
      this.page.currentPage = 1;
    },
    //提交添加新文章
    btnOK() {
      this.$refs.myForm.validate((isOK) => {
        if (isOK) {
          this.formData.id
            ? update(this.formData).then(() => {
              this.getList(this.keyword)
                this.formData = {
                  title: "", // 标题
                  articleBody: "", // 文章内容
                  cover: "" //视频地址
                  
                },
                this.dialogFormVisible = false
              })
            : addArticle(this.formData).then(() => {
              this.getList(this.keyword)
                this.formData = {
                  title: "", // 标题
                  articleBody: "", // 文章内容
                  cover: "" //视频地址
                  
                },
                this.dialogFormVisible = false
              });
        }
      });
    },
    //关闭弹窗
    btnCancel() {
      this.formData = {
        title: "", // 标题
        articleBody: "", // 文章内容
        videoURL: "" //视频地址
      };
      this.dialogFormVisible = false;
    },
    //删除文章
    delArticle(row) {
      this.$confirm("您确定要删除吗").then(() => {
        removeArticle({ id: row.id }).then(() => {
          this.getList({ keyword: this.keyword });
        });
      });
    },
    //改变状态
    changeState(row) {
      this.$confirm("您确定要改变当前状态吗").then(() => {
        changeSa({ state: 1, id: row.id }).then(() => {
          this.getList({ keyword: this.keyword });
        });
      });
    },
    //清除搜索框
    clear() {
      // console.log(this.content);
      this.keyword = "";
      this.getList();
    },
    formatterState(row, column, cellValue, index) {
      // console.log(row.state);
      return row.state ? "启用" : "禁用";
    },
    changePage(newPage) {  //此事件有一个参数   返回值是改变的页数 
      this.page.currentPage = newPage
      console.log(this.page.currentPage)
      this.getList({page:this.page.currentPage});
    },
    //获取列表
    getList(data) {
      getArticles({
        page: this.page.currentPage,
        pagesize: this.page.pageSize,
        ...data
      }).then(res => {
        this.tableData = res.data.items;
        this.page.total = res.data.counts
      });
    }
  },
  created() {
    this.getList();
  }
};
</script>



<style scoped>
.inputWidth {
  width: 220px;
  margin-right: 10px;
}
</style>
