<template>
  <el-card>
    <!-- 头部插槽 -->
    <el-row slot="header">
      <!-- 改变按钮颜色 但是需要按照elementui上面得规则改 -->
      <el-button type="primary" @click="dialogVisible = true">新建目录</el-button>
      <el-button type="primary" @click="callback">返回学科</el-button>
    </el-row>
    <!-- 表单  el-card主体  -->
    <!-- 全写:inline="true" 简写 inline 表单默认是纵向  inline改为横向 -->
    <el-form inline>
      <el-form-item label="目录">
        <!-- 绑定目录名称 -->
        <el-input v-model="searchForm.directoryName"></el-input>
      </el-form-item>
      <el-form-item label="状态">
        <el-select v-model="searchForm.state">
          <!-- 循环生成选项   数据来自data 或者计算属性-->
          <el-option
            v-for="item in status"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="clear">清除</el-button>
        <el-button type="primary" @click="search">搜索</el-button>
      </el-form-item>
    </el-form>
    <!-- 放置表格  data绑定数据-->
    <el-table :data="list">
      <!-- 列绑定字段 用prop属性  主要内容看接口文档-->
      <el-table-column label="序号" prop="id"></el-table-column>
      <el-table-column label="目录名称" prop="directoryName"></el-table-column>
      <el-table-column label="创建者" prop="creator"></el-table-column>
      <!-- 处理日期的方法 :formatter="绑定一个方法"  方法里面出来日期格式 或者 作用域插槽 然后在使用过滤器 过滤器在src filter中有定义 且在main中已经全局注册 所有不需要 引入  fliter中写了所有的过滤器 直接调用其就可以了-->
      <el-table-column label="创建日期" prop="addDate">
        <template slot-scope="obj">{{obj.row.addDate | parseTimeByString}}</template>
      </el-table-column>
      <el-table-column label="面试题数量" prop="totals"></el-table-column>
      <el-table-column label="状态" prop="state" :formatter="formatterStatus"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="obj">
            <!-- 修改方法需要传入id -->
          <el-button type="text" @click="modify(obj.row.id)">修改</el-button>
          <!-- 传过整条数据的方法 -->
          <el-button
            type="text"
            @click="changeState(obj.row)"
          >{{ obj.row.state === 1 ? "禁用" : "启用"}}</el-button>
          <el-button type="text" @click="delItem(obj.row.id )">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 放置分页组件 -->
    <el-row type="flex" justify="end" style="margin:40px ">
      <el-pagination
        background
        layout="prev,pager,next"
        :total="page.total"
        :page-size="page.pageSize"
        :current-page="page.currentPage"
      ></el-pagination>
    </el-row>
    <!-- 弹层 该标签有个属性visible 接受布尔值 true为显示 -->
    <el-dialog :show-close="false " :visible="dialogVisible">
      <el-form ref="myForm" :model="publishForm" :rules="rules" label-width="80px">
        <el-form-item label="目录名称" prop="directoryName">
          <!-- 输入内容 -->
          <el-input v-model="publishForm.directoryName" style="width:40%"></el-input>
        </el-form-item>
        <el-form-item label="学科" prop="subjectID">
          <!-- 选择下拉学科 -->
          <el-select v-model="publishForm.subjectID" style="width:40%">
            <!-- 学科下拉内容 -->
            <el-option
              v-for="item in subjects"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <el-button @click="btnOK" type="primary">确定</el-button>
      <el-button @click="btnCancel">取消</el-button>
    </el-dialog>
  </el-card>
</template>

<script>
// export const status = [
//   {
//     value: 1,
//     label: '启用'
//   },
//   {
//     value: 0,
//     label: '禁用'
//   }
// ]
import { status } from "../../api/hmmm/constants";
import { simple as simpleSubjects, detail } from "../../api/hmmm/subjects"; //学科简单列表
import {
  list as DirectorysList, //查询
  remove as removeDirectory, //删除
  removeState, //修改状态
  add as addDirectory ,
  detail as queryDirectory,
  update as updateDirectory
} from "../../api/hmmm/directorys"; // 因为有一些会重名 所有as一个别名 放置重名 获取不到数据
import { log } from "util";
import { async } from "q";

export default {
  data() {
    return {
      dialogVisible: false, //默认不显示弹窗
      searchForm: {
        directoryName: "", //目录名称
        state: null //状态
      },
      status, //相当于把status数据给 data中得变量 若不定义 直接引入之后使用 会报错 这样在上面写个插值表达式就可以获取到了
      list: [], //用来绑定表格数据
      page: {
        //分页数据
        total: 0, //总条数
        pageSize: 8, //每页条数
        currentPage: 1 //当前页码 默认第一页
      },
      subjects: [], //存放学科的简单列表
      // 表单数据对象
      publishForm: {
        subjectID: null, //学科id  在api接口文档中看
        directoryName: "" // 目录名称
      },
      rules: {
        // 校验规则
        // trigger  是校验的触发方式 默认是change 可以改变 我们选择失去焦点时进行校验 blur 失去焦点时触发
        subjectID: [
          { required: true, message: "学科不能为空", trigger: "blur" }
        ],
        directoryName: [
          { required: true, message: "目录名称不能为空", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    // 点击返回 返回到学科
    callback(){
      this.$router.push("list")
    },
    // 修改方法
    async modify(id){
      // 先获取要修改的数据
       let result = await queryDirectory({ id});  //拿到当前要修改的数据
       this.publishForm = result.data;  //弹窗的表单回直接 接受 要查询的数据
       this.dialogVisible = true //打开弹窗
    },
    // 确定
    btnOK() {
      // 首先校验不能为空
      console.log(1);

      this.$refs.myForm.validate(async isOK => {
        if (isOK) {
          // 如果校验是true的话  就调用接口  
          // 因为添加和修改用的是同一个 为了区分 需要判断一下 若返回里面有id就是修改 每页就是添加 
          this.publishForm.id ? await updateDirectory(this.publishForm) : await addDirectory(this.publishForm);
          // 成功之后 关闭模态框 但是在关闭之前应该先清空 当前form里面的内容
          this.publishForm = {
            subjectID: null,
            directoryName: ""
          };
          this.dialogVisible = false;
          // 关闭之后在重新加载页面
          this.getDirectory(this.searchForm);
        }
      });
      this.dialogVisible = false;
    },
    // 取消
    btnCancel() {
      this.publishForm = {
        subjectID: null,  //学科id
        directoryName: ""  //目录名称
      };
      this.dialogVisible = false;
    },
    // 修改状态
    async changeState(row) {
      await this.$confirm("你确实要删除此目录吗？"),
        await removeState({ id: row.id, state: row.state === 1 ? 0 : 1 }); //状态是反着的
      this.$message({ type: "success", message: "修改状态成功" });
      // 删除成功之后在重新加载数据
      this.getDirectory(this.searchForm);
    },
    // 定义一个删除的方法/
    async delItem(id) {
      await this.$confirm("你确实要删除吗？");
      // 如果确定 调用删除的接口
      await removeDirectory({ id }); //此步只是删除了后台的数据
      this.$message({ type: "success", message: "删除目录成功" });
      // 删除成功之后在重新加载数据
      this.getDirectory(this.searchForm);
    },
    // 搜索  带条件搜索 页码应该回到第一页
    search() {
      this.page.currentPage = 1;
      this.getDirectory(this.searchForm); //重新加载的时候条件丢了不合适
    },
    // 清空
    clear() {
      this.searchForm = {
        directives: "", //目录名称
        state: null //状态
      };
    },
    // 表格的列属性
    formatterStatus(row, column, cellValue, index) {
      // 通过cellValue参数可以  在 status 中找到匹配的value 返回对应的label
      let result = this.status.filter(item => item.value === cellValue); //得到一个筛选后的数组
      return result.length ? result[0].label : "未知状态";
    },
    async getDirectory(data) {
      // 获取目录数据  通过调用列表接口获取 接口在 api-hmmm-directory 中可以看到
      let result = await DirectorysList({
        page: this.page.currentPage, //默认请求第一页
        pageSize: this.page.pageSize, //请求的每页多少条
        // 使用es7里面的同步异步事件  来代替.then
        ...data //使用扩展字符串来接受参数 （将所有的条件合在一起）
      });
      this.page.total = result.data.counts; //总记录数
      this.list = result.data.items; //表格数据
    },
    // 页码改变事件
    changePage(newPage) {
      (this.page.currentPage = newPage), this.getComment();
    },
    getComment() {},
    async getSimpleSubject() {
      let result = await simpleSubjects();
      this.subjects = result.data; //获取简单列表
    }
  },
  // 钩子函数
  created() {
    this.getDirectory(); //获取目录数据
    this.getSimpleSubject(); //获取目录数据
  }
};
</script>

<style scoped>
</style>
