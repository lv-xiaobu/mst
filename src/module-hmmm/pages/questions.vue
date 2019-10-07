<template>
  <div class="dashboard-container">
    <el-row>
      <el-col :span="6">学科：
        <el-select v-model="searchForm.subjectID">
          <el-option v-for="item in subjectIDList" :key="item.value" :value="item.value" :label="item.label"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">难度：
        <el-select v-model="searchForm.difficulty">
          <el-option v-for="item in difficultyList" :key="item.value" :value="item.value" :label="item.label"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">试题类型：
        <el-select v-model="searchForm.questionType" style="width:200px;">
          <el-option v-for="item in questionTypeList" :key="item.value" :value="item.value" :label="item.label"></el-option>
        </el-select>
      </el-col>
    </el-row>
  </div>
</template>

<script>
// 把api数据接口相对应的方法导入进来
import { simple } from '@/api/hmmm/subjects'
import { difficulty, questionType as questionTypeList } from '@/api/hmmm/constants' // 常量不需要axios请求
export default {
  name: 'QuestionsList',
  data() {
    return {
      // 定义各个搜素表单域的数据展示成员
      subjectIDList: [], // 学科
      difficultyList: difficulty, // 难度
      questionTypeList, // 试题类型 简易成员赋值 questionTypeList：questionTypeList

      // 定义搜索数据对象
      searchForm: {
        subjectID: '', // 学科
        difficulty: '', // 难度
        questionType: ''// 试题类型
      }
    }
  },
  methods: {
    // 获得 学科 下拉列表的数据
    async getSubjectIDList() {
      var rst = await simple()
      //  把获得到的 学科 信息赋值给subjectIDList成员
      this.subjectIDList = rst.data
    }
  },
  created() {
    // 学科
    this.getSubjectIDList()
  }
}
</script>

<style scoped>
</style>
