<template>
  <div class="dashboard-container">
    <!-- <div class="app-container">题库添加</div> -->
    <el-row class="boss">
      <el-form>
        <el-form-item style="width:50%;" label-width="60px" label="学课">
          <el-select style="margin-left:10px" placeholder="请选择">
            <el-option></el-option>
          </el-select>
        </el-form-item>
        <el-form-item style="width:50%;" label-width="60px" label="目录">
          <el-select style="margin-left:10px" placeholder="请选择">
            <el-option></el-option>
          </el-select>
        </el-form-item>
        <el-form-item style="width:50%;" label-width="60px" label="企业">
          <el-select style="margin-left:10px" placeholder="请选择">
            <el-option></el-option>
          </el-select>
        </el-form-item>
        <el-form-item  label-width="60px" label="城市">
          <el-select  style="width:11%;margin-left:10px" placeholder="请选择">
            <el-option></el-option>
          </el-select>
           <el-select style="width:11%;margin-left:10px" placeholder="请选择">
            <el-option></el-option>
          </el-select>
        </el-form-item>
      </el-form>
    </el-row>
  </div>
</template>

<script>
export default {
  name: "QuestionsNew",
  data() {
    return {};
  }
};
</script>

<style scoped>
.dashboard-container {
}
.boss {
  margin: 20px;
  padding: 20px;
  background: #fff;
  box-shadow: 0px 0px 20px #ccc;
}
</style>
