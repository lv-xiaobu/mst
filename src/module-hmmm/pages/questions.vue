<template>
  <div class="dashboard-container">
    <div class="app-container">
      <!-- <el-form ref="searchFormRef" :model="searchForm"></el-form> 
            ref：用于获取当前的el-form元素
            :model: 是el-form的固定属性，用于获取整个form表单的各项表单域信息
      -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="$router.push('/questions/new')"> {{ $t('question.xinzeng') }}</el-button>
          <el-button type="danger"> {{ $t('question.pidao') }}</el-button>
        </el-col>
      </el-row>
      <el-row :gutter="10">

        <el-col :span="6">
          学科：
          <el-select v-model="searchForm.subjectID" clearable>
            <el-option v-for="item in subjectIDList" :key="item.value" :value="item.value" :label="item.label">
            </el-option>
          </el-select>
        </el-col>
        <el-col :span="6">
          难度：
          <el-select v-model="searchForm.difficulty" clearable>
            <el-option v-for="item in difficultyList" :key="item.value" :value="item.value" :label="item.label">
            </el-option>
          </el-select>
        </el-col>
        <el-col :span="6">
          试题类型：
          <el-select v-model="searchForm.questionType" style="width:140px;" clearable>
            <el-option v-for="item in questionTypeList" :key="item.value" :value="item.value" :label="item.label">
            </el-option>
          </el-select>
        </el-col>
        <el-col :span="6">标签:
          <el-select placeholder="请选择" v-model="searchForm.tags" style="width:135px" clearable>
            <el-option 
              v-for="item in tagsList" 
              :key="item.label" 
              :label="item.username" 
              :value="item.label">
            </el-option>
          </el-select>
        </el-col>

      </el-row>

       <el-row :gutter="20">
        <el-col :span="6">城市：
          <el-select placeholder="城市" v-model="searchForm.province" style="width:102px" @change="getCitys">
            <el-option v-for="item in provinces()" :key="item" :value="item" :label="item"></el-option>
          </el-select>
          <el-select placeholder="区县" v-model="searchForm.city" style="width:102px">
            <el-option v-for="item in cityList" :key="item" :value="item" :label="item"></el-option>
          </el-select>

        </el-col>
        <el-col :span="6">关键字：
          <el-input type="text" placeholder="请输入关键字" v-model="searchForm.keyword" style="width:190px"></el-input>
        </el-col>
        <el-col :span="6">题目备注：
          <el-input type="text" placeholder="请输入备注" v-model="searchForm.remarks" style="width:190px"></el-input>
        </el-col>
        <el-col :span="6">企业简称：
          <el-input type="text" placeholder="请输入简称" v-model="searchForm.shortName" style="width:190px"></el-input>
        </el-col>
      </el-row>

      <el-row :gutter="20">
        <el-col :span="6">方向：
          <el-select placeholder="请选择" v-model="searchForm.direction" style="width:135px">
            <el-option v-for="item in directionList" :key="item" :value="item" :label="item"></el-option>
          </el-select>
        </el-col>
        <el-col :span="6">录入人：
          <el-select placeholder="请选择" v-model="searchForm.creatorID" style="width:135px">
            <el-option v-for="item in creatorIDList" :key="item.id" :label="item.username" :value="item.id"></el-option>
          </el-select>
        </el-col>
        <el-col :span="6">二级目录：
          <el-select placeholder="请选择" v-model="searchForm.catalogID" style="width:135px">
            <el-option v-for="item in catalogDList" :key="item.value" :label="item.label" :value="item.value"></el-option>
          </el-select>
        </el-col>
        <el-col :span="6">
          <el-button size="mini">清除</el-button>
          <el-button type="primary" size="mini" @click="getQuestionsList()">搜索</el-button>
        </el-col>
      </el-row>
      
        <el-table :data="questionsList" style="width:100%">
          <el-table-column label="序号" type="index"></el-table-column>
          <el-table-column label="试题编号" prop="number"></el-table-column>
          <el-table-column label="学科" prop="subject"></el-table-column>
          <el-table-column label="题型" prop="questionType" :formatter="questionTypeFMT"></el-table-column>
          <el-table-column label="题干" prop="question"></el-table-column>
          <el-table-column label="录入时间" prop="addDate" width="170">
             <!--table表格column中获取数据信息，要使用 作用域插槽 形式-->
             <span slot-scope="stData">{{stData.row.addDate | parseTimeByString}}</span>
          </el-table-column>
          <el-table-column label="难度" prop="difficulty" :formatter="difficultyFMT"></el-table-column>
          <el-table-column label="录入人" prop="creator"></el-table-column>
          <el-table-column label="操作" width="200">
            <template slot-scope="stData">
              <a href="#">预览</a>
              <a href="#">修改</a>
              <!-- 用.prevent阻止a默认动作 -->
              <a href="#" @click.prevent="del(stData.row)">删除</a>
              <a href="#">加入精选</a>
            </template>
        </el-table-column>
      </el-table>
       <el-row type='flex' justify="center">
              <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="searchForm.page"
              :page-sizes="[4, 10, 15, 20]"
              :page-size="searchForm.pagesize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="tot"
            ></el-pagination>
       </el-row>
    </div>
  </div>
  
</template>

<script>
// 把api数据接口相关方法导入进来
import { simple } from '@/api/hmmm/subjects' // 学科
import { simple as tagsSimple } from '@/api/hmmm/tags' // 标签
import { simple as usersSimple } from '@/api/base/users' // 录入人
import { simple as directorysSimple } from '@/api/hmmm/directorys'// 二级目录
import { provinces, citys } from '@/api/hmmm/citys'// 城市
import {list, remove} from '@/api/hmmm/questions'// 基础题库信息列表

// as给导入的成员设置别名
import {
  difficulty as difficultyList, 
  questionType as questionTypeList,
  direction as directionList
  } 
  from '@/api/hmmm/constants' // 常量数据
import { format } from 'util'

export default {
  name: 'QuestionsList',
  data() {
    return {
      // 定义各个搜索表单域的数据展示成员
      subjectIDList: [],
      difficultyList, // 简易成员赋值(difficultyList:difficultyList)
      questionTypeList,
      tagsList: [], // 标签
      creatorIDList: [], // 录入人
      catalogDList: [], // 二级目录
      directionList, // 方向
      cityList: [], // 区县
      questionsList: [], // 基础题库列表

      // 定义搜索数据对象
      searchForm: {
        subjectID: '', // 科学
        difficulty: '', // 难度
        questionType: '', // 试题类型
        tags: '', // 标签
        province: '', // 省份
        city: '', // 城市
        keyword: '', // 关键字
        remarks: '', // 备注
        shortname: '', // 企业简称
        direction: '', // 方向
        creatorID: '', // 录入人
        catalogID: '', // 二级目录
        tot: 0, // 总条数
        page: 1, // 默认获取第一页
        pagesize: 4// 默认每页四条
      }
    }
  },
  created() {
    // 学科
    this.getSubjectIDList()
    // 标签
    this.getTagsList()
    // 录入人
    this.getCreatorIDList()
    // 二级目录
    this.getcatalogDList()
    // 基础题库
    this.getQuestionsList()
  },

  methods: {
    // 每页条数变化
    handleSizeChange(val) {
      this.searchForm.pagesize = val
      this.getQuestionsList()
    },
    // 当前页变化
    handleCurrentChange(val) {
      this.searchForm.page = val
      this.getQuestionsList()
    },
    // 删除题库
    del(info) {
      // 提示
      this.$confirm('确认要删除该题库吗？', '删除', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).then(() => {
        remove(info)
        this.getQuestionsList()
      }).catch(() => {})
    },
    // 获得 学科 下拉列表数据
    async getSubjectIDList() {
      var rst = await simple()
      // console.log(rst)
      // 把获得到的学科信息赋予给 subjectIDList 成员
      this.subjectIDList = rst.data
    },
    // 获得 标签 列表数据
    async getTagsList() {
      var rst = await tagsSimple()
      this.tagsList = rst.data
    },
    // 获得 录入人 数据列表
    async getCreatorIDList() {
      var rst = await usersSimple()
      this.creatorIDList = rst.data
    },
    // 获得 二级目录 数据列表
    async getcatalogDList() {
      var rst = await directorysSimple()
      this.catalogDList = rst.data
    },
    // 城市
    provinces,
    // 区县
    getCitys(pname) {
      this.searchForm.city = ''// 清空上次选择的城市
      this.cityList = citys(pname)
    },
    // 获得基础题库列表
    async getQuestionsList() {
      var rst = await list(this.searchForm)
      this.questionsList = rst.data.items
      // 分页总条数赋值
      this.tot = rst.data.counts
    },
    // 题型数字转汉字 三个参数分别代表行信息 列信息 当前域信息
    questionTypeFMT(row, column, cellValue) {
      return this.questionTypeList[cellValue - 1]['label'] // 按下标获取所有要-1
    },
    // 难度数字转汉字
    difficultyFMT(row, column, cellValue) {
      return this.difficultyList[cellValue - 1]['label']
    }
  }
}
</script>

<style scoped>
.el-row{margin-top:10px;}
.el-table{margin-top:10px;}
</style>
