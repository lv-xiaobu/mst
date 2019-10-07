<template>
  <div class="dashboard-container">
    <el-row>
      <el-col :span="7">学科：
        <el-select></el-select>
      </el-col>
      <el-col :span="7">难度：
        <el-select></el-select>
      </el-col>
      <el-col :span="7">试题类型：
        <el-select></el-select>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  name: 'QuestionsList',
  data() {
    return {
      // 定义各个搜素表单域的数据展示成员
      // 定义搜索数据对象
      searchForm: {}
    }
  }
}
</script>

<style scoped>
</style>
