<template>
  <div class="dashboard-container">
    <el-row>
      <el-button type="primary" size="mini" @click="$router.push('/questions/new')">
        {{ $t('question.xinzeng') }}
      </el-button> 
      <el-button type="danger" size="mini">
        {{ $t('question.pidao') }}
      </el-button> 
    </el-row>
    <el-row :gutter="10">
      <el-col :span="6">学 科：
        <el-select placeholder="请选择" v-model="searchForm.subjectID"  style="width:135px" clearable>
          <el-option v-for="item in subjectIDList" :key="item.value" :value="item.value" :label="item.label"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">难 度：
        <el-select placeholder="请选择" v-model="searchForm.difficulty"  style="width:135px" clearable>
          <el-option v-for="item in difficultyList" :key="item.value" :value="item.value" :label="item.label"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">试题类型：
        <el-select placeholder="请选择" v-model="searchForm.questionType" style="width:135px" clearable>
          <el-option v-for="item in questionTypeList" :key="item.value" :value="item.value" :label="item.label"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">标 签:
        <el-select placeholder="请选择" v-model="searchForm.tags" style="width:135px" clearable>
          <el-option v-for="item in tagsList" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
      </el-col>
    </el-row>

    <el-row :gutter="10">
      <el-col :span="6">城 市：
        <!-- 给**城市**设置change内容改变事件 -->
        <el-select placeholder="城市" v-model="searchForm.province" style="width:102px" clearable @change="getCitys">
          <el-option v-for="item in provinces()" :key="item" :label="item" :value="item"></el-option>
        </el-select>
        <el-select placeholder="区县" v-model="searchForm.city" style="width:102px" clearable>
          <el-option v-for="item in cityList" :key="item" :label="item" :value="item"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">关键字：
        <el-input type="text" placeholder="请输入关键字" v-model="searchForm.keyword" style="width:170px" clearable></el-input>
      </el-col>
      <el-col :span="6">题目备注：
        <el-input type="text" placeholder="请输入备注" v-model="searchForm.remarks" style="width:170px" clearable></el-input>
      </el-col>
      <el-col :span="6">企业简称：
        <el-input type="text" placeholder="请输入简称" v-model="searchForm.shortName" style="width:170px" clearable></el-input>
      </el-col>
    </el-row>

    <el-row :gutter="10">
      <el-col :span="6">方向：
        <el-select placeholder="请选择" v-model="searchForm.direction" style="width:135px" clearable>
          <el-option v-for="item in directionList" :key="item" :value="item" :label="item"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">录入人：
        <el-select placeholder="请选择" v-model="searchForm.creatorID" style="width:135px" clearable>
          <el-option v-for="item in creatorIDList" :key="item.id" :label="item.username" :value="item.id"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">二级目录：
        <el-select placeholder="请选择" v-model="searchForm.catalogID" style="width:135px" clearable>
          <el-option  v-for="item in catalogIDList"  :key="item.value"  :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-col>
      <el-col :span="6">
        <el-button size="mini">清除</el-button>
        <el-button type="primary" size="mini" @click="getQuestionList">搜索</el-button>
      </el-col>
    </el-row>
    <el-table style="text-align: center" :data="questionList" border>   
    <el-table-column label="序号" type="index"></el-table-column>
        <el-table-column label="试题编号" prop="number"></el-table-column>
        <el-table-column label="学科" prop="subject"></el-table-column>
        <el-table-column label="题型" :formatter="questionTypeFMT" prop="questionType"></el-table-column>
        <el-table-column label="题干" prop="question"></el-table-column>
        <el-table-column label="录入时间" width="170">
        <!--table表格column中获取数据信息，要使用 作用域插槽 形式-->
          <template slot-scope="stDate">{{stDate.row.addDate | parseTimeByString('{y}-{m}-{d} {h}:{i}:{s}')}}</template>
        </el-table-column>
        <el-table-column label="难度" :formatter="difficultyFMT" prop="difficulty" ></el-table-column>
        <el-table-column label="录入人" prop="creator"></el-table-column>
        <el-table-column label="操作" width="180">
          <template slot-scope='stData'>
            <a href="#">预览</a>
            <a href="#">修改</a>
            <!-- .prevent 阻止a标签的默认事件 -->
            <a href="#" @click.prevent="del(stData.row)">删除</a>
            <a href="#">加入精选</a>
          </template> 
        </el-table-column>
    </el-table>

    <!-- 分页 -->
    <el-row type='flex' justify='center' style="margin:20px 0">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="searchForm.page"
        :page-sizes="[4, 10, 15, 20]"
        :page-size="searchForm.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="searchForm.tot">
      </el-pagination>
    </el-row>
  </div>
</template>

<script>
import { simple } from '@/api/hmmm/subjects'// 把api数据接口相对应的方法导入进来
import {
  difficulty,
  questionType as questionTypeList,
  direction as directionList // 获取 方向 下拉列表数据部分
} from '@/api/hmmm/constants' // 常量不需要axios请求
import { simple as tagsSimple } from '@/api/hmmm/tags' // 获取标签信息方法导入
import {provinces, citys} from '@/api/hmmm/citys' // 获取 省份/城市 信息方法导入 常量不需要axios请求
import { simple as usersSimple } from '@/api/base/users' // 获取录入人信息方法导入
import {simple as directorysSimple} from '@/api/hmmm/directorys' // 获取二级目录信息方法导入
import {list, remove} from '@/api/hmmm/questions' // 基础题库相关api导入

export default {
  name: 'QuestionsList',
  data() {
    return {
      // 定义各个搜素表单域的数据展示成员
      subjectIDList: [], // 学科
      difficultyList: difficulty, // 难度
      questionTypeList, // 试题类型 简易成员赋值 questionTypeList：questionTypeList
      tagsList: [], // 标签
      cityList: [], // 城市信息
      directionList, // 方向
      creatorIDList: [], // 录入人
      catalogIDList: [], // 二级目录
      questionList: [], // 基础题库列表信息

      // 定义搜索数据对象
      searchForm: {
        page: 1, // 默认获取第一页信息
        pagesize: 4, // 每页显示4条信息
        tot: 0, // 总的记录条数
        subjectID: '', // 学科
        difficulty: '', // 难度
        questionType: '', // 试题类型
        tags: '', // 标签
        province: '', // 城市
        city: '', // 区县
        keyword: '', // 关键字
        remarks: '', // 备注
        shortname: '', // 企业简称
        direction: '', // 方向
        creatorID: '', // 录入人
        catalogID: '' // 二级目录
      }
    }
  },
  methods: {
    // ===== 每页条数变化的回调处理
    handleSizeChange(newsize) {
      this.searchForm.pagesize = newsize
      this.getQuestionList()
    },
    // ===== 当前页码变化的回调处理
    handleCurrentChange(newpage) {
      this.searchForm.page = newpage
      this.getQuestionList()
    },
    // ===== 获得 学科 下拉列表的数据
    async getSubjectIDList() {
      var rst = await simple()
      //  把获得到的 学科 信息赋值给subjectIDList成员
      this.subjectIDList = rst.data
    },
    // ===== 获得 标签 列表数据
    async getTagsList() {
      var rst = await tagsSimple()
      this.tagsList = rst.data
    },
    // ===== 获得 录入人 列表数据
    async getCreatorIDList() {
      var rst = await usersSimple()
      this.creatorIDList = rst.data
    },
    // ===== 获得 二级目录 列表数据
    async getCatalogIDList() {
      var rst = await directorysSimple()
      this.catalogIDList = rst.data
    },
    // ===== 获得 城市 信息，简易成员赋值
    provinces, 
    // ===== 获得 区县 信息
    // getCitys方法在模板中声明的时候，没有设置()
    // 这个pname形参就代表被选中的城市信息
    getCitys(pname) {
      this.searchForm.city = '' // 清除之前选取好的城市
      this.cityList = citys(pname)
    },
    // ===== 获得基础题库列表信息
    async getQuestionList() {
      var rst = await list(this.searchForm)
      // 把获得好的题库列表信息赋予给 questionList 成员
      this.questionList = rst.data.items
      this.searchForm.tot = rst.data.counts
    },
    // ===== 题型数字转汉字
    // row：代表一行记录信息  column：代表列的信息  cellValue：当前正要处理的域的信息
    questionTypeFMT(row, column, cellValue) {
      return this.questionTypeList[cellValue - 1]['label']
    },
    // ===== 难度数字转汉字
    difficultyFMT(row, column, cellValue) {
      return this.difficultyList[cellValue - 1]['label']
    },
    // ===== 删除题库
    del(info) {
      // 确认框
      this.$confirm('确认要删除该题库么?', '删除', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // remove(info) // 删除数据
          remove({id: info.id}) // 删除数据
          this.getQuestionList() // 更新数据
        }).catch(() => {
        })
    }
  },
  created() {
    // 学科
    this.getSubjectIDList()
    // 获得标签信息
    this.getTagsList()
    // 获得录入人信息
    this.getCreatorIDList()
    // 获得 二级目录 信息
    this.getCatalogIDList()
    // 获得 基础题库 列表信息
    this.getQuestionList()
  }
}
</script>

<style scoped>
 div {
  overflow: hidden;
  vertical-align: middle;
}
.el-row {
  margin-top:10px;
  padding: 0 10px;
}
.el-table {
  margin:10px;
  padding-right: 10px;
}
</style>
