<template>
  <div class="dashboard-container">
    <div class="app-container">
      <!-- 用:model作用 可以使得el-form针对表单的全部信息进行收集，固定属性 -->
      <el-form ref="addFormRef" :model="addForm" label-width="120px">
        <el-form-item label="学科">
          <el-select v-model="addForm.subjectID">
              <el-option
              v-for="item in subjectIDList"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="目录">
          <el-select v-model="addForm.catalogID">
              <el-option
              v-for="item in catalogIDList"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="企业">
          <el-select v-model="addForm.enterpriseID">
                <el-option
                v-for="item in enterpriseIDList"
                :key="item.id"
                :label="item.company"
                :value="item.id"
              ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="城市">
          <el-select v-model="addForm.province" @change="getCitys">
            <el-option 
            v-for="item in provinces()" 
            :key="item" 
            :label="item" 
            :value="item">
            </el-option>
          </el-select>
          <el-select v-model="addForm.city">
            <el-option 
            v-for="item in citysList" 
            :key="item" 
            :label="item" 
            :value="item">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="方向">
          <el-select v-model="addForm.direction">
            <el-option 
            v-for="item in directionList" 
            :key="item" 
            :value="item" 
            :label="item">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="题型">
          <el-radio-group v-model="addForm.questionType">
            <el-radio v-for="item in questionTypeList" :key="item.value" :label="item.value+''">
              {{item.label}}
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="难度">
          <el-radio-group v-model="addForm.difficulty">
            <el-radio v-for="item in difficultyList" :key="item.value" :label="item.value+''">
              {{item.label}}
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="题干">
          <el-input type="textarea" v-model="addForm.question"></el-input>
        </el-form-item>
       <el-form-item label="选项：" v-if="allShow">
          <template v-if="radioShow">
            <el-radio v-model="singleSelect" :label="0">A:
              <el-input type="text" v-model="addForm.options[0]['title']"></el-input>
            </el-radio>
            <br>
            <el-radio v-model="singleSelect" :label="1">B:
              <el-input type="text" v-model="addForm.options[1]['title']"></el-input>
            </el-radio>
            <br>
            <el-radio v-model="singleSelect" :label="2">C:
              <el-input type="text" v-model="addForm.options[2]['title']"></el-input>
            </el-radio>
            <br>
            <el-radio v-model="singleSelect" :label="3">D:
              <el-input type="text" v-model="addForm.options[3]['title']"></el-input>
            </el-radio>
          </template>
          <template v-else>
            <el-checkbox v-model="addForm.options[0]['isRight']">A:
              <el-input type="text" v-model="addForm.options[0]['title']"></el-input>
            </el-checkbox>
            <br>
            <el-checkbox v-model="addForm.options[1]['isRight']">B:
              <el-input type="text" v-model="addForm.options[1]['title']"></el-input>
            </el-checkbox>
            <br>
            <el-checkbox v-model="addForm.options[2]['isRight']">C:
              <el-input type="text" v-model="addForm.options[2]['title']"></el-input>
            </el-checkbox>
            <br>
            <el-checkbox v-model="addForm.options[3]['isRight']">D:
              <el-input type="text" v-model="addForm.options[3]['title']"></el-input>
            </el-checkbox>
          </template>
        </el-form-item>
        <el-form-item label="答案">
          <el-input type="textarea" v-model="addForm.answer"></el-input>
        </el-form-item>
        <el-form-item label="备注">
          <el-input type="textarea" v-model="addForm.remarks"></el-input>
        </el-form-item>
        <el-form-item label="标签">
          <el-input type="text" v-model="addForm.tags"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="addQuestion()">提交</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
import { simple as subjectsSimple } from '@/api/hmmm/subjects' // 获取标签信息方法 
import { simple as directorysSimple } from '@/api/hmmm/directorys'// 获取二级目录方法
import { provinces, citys } from '@/api/hmmm/citys'// 获取城市 县区
import { difficulty as difficultyList, direction as directionList, questionType as questionTypeList } from '@/api/hmmm/constants'// 方向
import { list as companysList } from '@/api/hmmm/companys'// 企业
import { add } from '@/api/hmmm/questions' 

export default {
  name: 'QuestionsNew',
  data() {
    return {
      addForm: {
        subjectID: '', // 学科
        catalogID: '', // 目录
        enterpriseID: '', // 企业
        province: '', // 城市
        city: '', // 区县
        direction: '', // 方向
        questionType: '1', // 默认单选题型
        difficulty: '1', // 默认难度

        question: '', // 题干
        answer: '', // 答案
        remarks: '', // 备注
        tags: '', // 标签
        videoURL: 'http://www.xxx.com', // 解析视频
        // 选项表单数据对象部分
        options: [
          { code: 'A', title: '', img: '', isRight: false },
          { code: 'B', title: '', img: '', isRight: false },
          { code: 'C', title: '', img: '', isRight: false },
          { code: 'D', title: '', img: '', isRight: false }
        ]
      },
      radioShow: true, // 默认显示单选项目
      allShow: true, // 单选或多选默认显示一个
      singleSelect: '',
      subjectIDList: [], // 学科
      catalogIDList: [], // 二级目录
      citysList: [], // 县区信息
      directionList, // 方向
      enterpriseIDList: [], // 企业
      questionTypeList, // 题型 
      difficultyList // 难度 
    }
  },
  methods: {
    // 收集数据并添加
    async addQuestion() {
      await add(this.addForm)
      this.$message.success('添加数据成功')
      this.$router.push('/questions/list')
    },
    // 获得 企业
    async getEnterpriseIDList() {
      var rst = await companysList()
      this.enterpriseIDList = rst.data.items
    },
    // 获得 省份信息
    provinces,
    // 获得城市
    getCitys(pname) {
      this.addForm.city = ''
      this.citysList = citys(pname)
    },
    // 获取二级目录
    async getCatalogIDList () {
      var rst = await directorysSimple()
      this.catalogIDList = rst.data
    },
    // 获得学科列表信息
    async getSubjectIDList() {
      var rst = await subjectsSimple()
      this.subjectIDList = rst.data
    }
  },
  // 监听
  watch: {
    // 监听选项中要把isRight的值做设置操作
    singleSelect(newval) {
      for (let i = 0; i < 4; i++) {
        this.addForm.options[i].isRight = false
        
      }
      this.addForm.options[newval].isRight = true
    },
    // 对题型进行监听
    'addForm.questionType': function(newval) {
      // console.log(newval)
      if (Number(newval) === 1) {
        // newval=1 单选
        this.radioShow = true
        this.allShow = true
      } else if (Number(newval) === 2) {
        // newval=2 多选
        this.radioShow = false
        this.allShow = true
      } else {
        // newval=3 简答
        this.allShow = false
      }
    }

  },
  created() {
    this.getEnterpriseIDList()// 获得企业
    this.getCatalogIDList()// 获得目录
    this.getSubjectIDList()// 获得学科
  }
}
</script>

<style scoped>
</style>
