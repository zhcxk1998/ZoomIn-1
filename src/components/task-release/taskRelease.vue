<template>
  <el-container style="height:100%">
    <left :taskList="taskList" :selectedIndex="selectedIndex" :changeTask="changeTask"></left>
    <el-container v-loading="loading" element-loading-text="任务加载中...">
      <el-main style="padding:0 5%;">
        <div style="margin-top:20px;">
        </div>
        <div class="task-title">
          <div class="task-title-name">
            {{newTaskModel.taskName}}
          </div>
          <div class="task-title-new">
            <el-button type="primary" icon="el-icon-picture" @click="newChartDialogVisible=true">新建图表</el-button>
            <el-button type="primary" icon="el-icon-picture" @click="generateReport">生成报告</el-button>
          </div>
          <div style="clear:both"></div>
        </div>

        <div>数据分析</div>
        <div class="table-container">
          <div v-for="element in analyzeArray" :key="element.id" style="width:350px;margin:10px">
            <el-card class="box-card">
              <div slot="header" class="clearfix">
                <span>图表标题</span>
                <div class="dropdown" style="float: right;">
                  <el-dropdown @command="extendClick">
                    <span class="el-dropdown-link">
                      <el-button style="float: right; padding: 3px 0" type="text"><i class="el-icon-more-outline"></i></el-button>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                      <el-dropdown-item :command="element.name">放大</el-dropdown-item>
                      <el-dropdown-item command="编辑">编辑</el-dropdown-item>
                      <el-dropdown-item>删除</el-dropdown-item>
                    </el-dropdown-menu>
                  </el-dropdown>
                </div>
              </div>
              <img :src="'http://120.79.146.91:8000'+element" width="100%" height="185px" class="image">
            </el-card>
          </div>
        </div>
        <div>数据挖掘（线性回归）</div>
        <div class="table-container">
          <div v-for="element in clusteringArray" :key="element.id" style="width:350px;margin:10px">
            <el-card class="box-card">
              <div slot="header" class="clearfix">
                <span>图表标题</span>
                <div class="dropdown" style="float: right;">
                  <el-dropdown @command="extendClick">
                    <span class="el-dropdown-link">
                      <el-button style="float: right; padding: 3px 0" type="text"><i class="el-icon-more-outline"></i></el-button>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                      <el-dropdown-item :command="element.name">放大</el-dropdown-item>
                      <el-dropdown-item command="编辑">编辑</el-dropdown-item>
                      <el-dropdown-item>删除</el-dropdown-item>
                    </el-dropdown-menu>
                  </el-dropdown>
                </div>
              </div>
              <img :src="'http://120.79.146.91:8000'+element" width="100%" height="185px" class="image">
            </el-card>
          </div>
        </div>
        <div>数据挖掘（非线性回归）</div>
        <div class="table-container">
          <div v-for="element in regressionArray" :key="element.id" style="width:350px;margin:10px">
            <el-card class="box-card">
              <div slot="header" class="clearfix">
                <span>图表标题</span>
                <div class="dropdown" style="float: right;">
                  <el-dropdown @command="extendClick">
                    <span class="el-dropdown-link">
                      <el-button style="float: right; padding: 3px 0" type="text"><i class="el-icon-more-outline"></i></el-button>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                      <el-dropdown-item :command="element.name">放大</el-dropdown-item>
                      <el-dropdown-item command="编辑">编辑</el-dropdown-item>
                      <el-dropdown-item>删除</el-dropdown-item>
                    </el-dropdown-menu>
                  </el-dropdown>
                </div>
              </div>
              <img :src="'http://120.79.146.91:8000'+element" width="100%" height="185px" class="image">
            </el-card>
          </div>
        </div>

        <el-dialog :visible.sync="picDialogVisible" width="50%" center>
          <img :src="picPath" style="width:100%" class="image">
        </el-dialog>


      </el-main>
    </el-container>

    <el-dialog title="新建图表-选择数据源" :visible.sync="newChartDialogVisible" width="30%">
      <el-form :model="newChartModel" ref="newChartModel" :rules="chartModelRules" status-icon label-width="80px" class="demo-ruleForm" label-position="left" @keyup.native="submitTask($event)">
        <el-form-item label="数据集名" prop="dataSetTitle">
          <el-input type="text" v-model="newChartModel.dataSetTitle"></el-input>
        </el-form-item>
        <el-form-item label="数据源" prop="IsChooseHistory">
          <el-radio-group v-model="newChartModel.IsChooseHistory" @change="watchChoose">
            <el-radio :label="true">历史数据集</el-radio>
            <el-radio :label="false">新建数据集</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item v-if="newChartModel.IsChooseHistory" label="数据集" prop="historyDataSet">
          <el-input v-model="newChartModel.historyDataSet" v-show="false"></el-input>
          <el-select v-model="newChartModel.historyDataSet" placeholder="请选择数据集">
            <el-option v-for="dataSet in dataSetList" :label="dataSet.title" :value="dataSet.id" :key="dataSet.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item v-if="!newChartModel.IsChooseHistory" label="数据集">
          <input type="file" ref="obj" @change="importFile()" />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="newChartDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="nextStep()">下一步</el-button>
      </span>
    </el-dialog>
    <el-dialog title="新建图表-任务设置" :visible.sync="newChartTaskDialogVisible" width="30%">
      <el-form :model="newTaskModel" status-icon label-width="80px" class="demo-ruleForm" label-position="left" @keyup.native="submitTask($event)">
        <el-form-item label="任务名" prop="taskName">
          <el-input type="text" v-model="newTaskModel.taskName"></el-input>
        </el-form-item>
        <el-form-item label="任务属性" prop="taskProperty">
          <el-input type="text" v-model="newTaskModel.taskProperty"></el-input>
        </el-form-item>
        <el-form-item label="创建者" prop="creator">
          <el-input type="text" v-model="newTaskModel.creator"></el-input>
        </el-form-item>
        <el-form-item label="任务描述" prop="taskDesc">
          <el-input type="textarea" v-model="newTaskModel.taskDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="newChartTaskDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="createNewDataSet">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 数据预览模态框 -->
    <el-dialog :visible.sync="tablePreviewVisable" width="50%" title="数据预览" top="5vh">
      <preview-table :json="tablejsons" v-on:setTitleIndex="setTitleIndex"></preview-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="tablePreviewVisable = false">取 消</el-button>
        <el-button type="primary" @click="tablePreviewVisable=false;newChartTaskDialogVisible=true">下一步</el-button>
      </span>
    </el-dialog>
    <el-dialog title="生成预览" :visible.sync="generateReportVisable" width="40%">
      <Report :taskInfo="newTaskModel" :data="report" :dataAnalysisPic="analyzeArray" :dataMiningPic="regressionArray" :changeWordLoading="changeWordLoading" ref="report"></Report>
      <span slot="footer" class="dialog-footer">
        <el-button @click="generateReportVisable = false">取 消</el-button>
        <el-button type="primary" @click="generateHtml()" style="">生成html链接</el-button>
        <el-button type="primary" @click="generateWord()" style="" v-loading="wordLoading" element-loading-text="文档生成中...">导出word</el-button>
      </span>
    </el-dialog>
  </el-container>
</template>
<script>
import Left from "../common/ReleastLeft.vue";
import MyChart from "./mychart.vue";
import PreviewTable from "../data-import/PrviewTable.vue";
import Report from "./Report.vue";
import Papa from "papaparse";
import {
  converterTwoDimArrayToObjectArray,
  converterFileToJson,
  importf
} from "@/utils/fileToJson.js";
import validateObj from "@/utils/validate.js";
import draggable from "vuedraggable"; /*CJW 引入拖拽 */
import { rejects } from "assert";
import { resolve } from "url";
export default {
  components: {
    Left,
    MyChart,
    PreviewTable,
    Report,
    draggable /*CJW 引入拖拽 */
  },
  data() {
    return {
      wordLoading:false,
      analyzeArray:[],
      clusteringArray:[],
      regressionArray:[],
      loading:true,
      selectedIndex:0,
      myArray: [],
      picPath: "",
      picDialogVisible: false,
      taskId: "",
      dataSetId:'',
      newChartDialogVisible: false,
      newChartTaskDialogVisible: false,
      tablePreviewVisable: false,
      generateReportVisable: false,
      newChartModel: {
        dataSetTitle: "",
        IsChooseHistory: true,
        historyDataSet: ""
      },
      newTaskModel: {
        taskName: "",
        taskProperty: "",
        creator: "",
        taskDesc: ""
      },
      tablejsons: "",
      titleIndex: 0,
      taskList: [],
      dataSetList: [],
      chartModelRules: {
        dataSetTitle: [
          { validator: validateObj.validateTitle, trigger: "change" }
        ],
        historyDataSet: [
          {
            validator: validateObj.validateNoNullOrUndefined,
            trigger: "change"
          }
        ]
      },
      report: Object,
      dataAnalysisPicUrl: [],
      dataMiningPicUrl: [],
      dataAnalysisPic: [],
      dataMiningPic: []
    };
  },
  created: function() {
    this.taskId=this.$route.params.taskId;
    this.dataSetId=this.$route.params.dataSetId;

    this.$store.commit("changeIndex", { index: "taskRelease" });

    let query = this.fetchAllTaskInfo();
    query.then(req => {
      this.taskList = req;
      if (this.taskId == undefined || this.taskId == "") {
        this.$router.push("/home/task-release/" + this.taskList[0].id);
        this.$post("/publish/", {
        task_id: this.taskList[0].id
      }).then(response => {
        this.report = response;
        this.dealAllReportUrl();
      });
      this.selectedIndex = 0;
      } else {
        this.$router.push("/home/task-release/" + this.taskId);
        this.fetchTaskInfo();
        this.fetchReport();
        this.$post("/publish/", {
        task_id: this.taskId
      }).then(response => {
        this.report = response;
        this.dealAllReportUrl();
      });
      this.selectedIndex = this.taskList.findIndex(item=>item.id===this.taskId)
      }
    });
    this.fetchAllDataSet();
    
  },
  mounted(){
    
  },
  methods: {
    changeTask:function(index){
      this.selectedIndex=index;
      this.taskId = this.taskList[index].id;
      this.$router.push("/home/task-release/"+this.taskList[index].id);
    },
    fetchTaskInfo: function() {
      this.loading=true;
      this.$get("/taskinfo/" + this.taskId).then(response => {
        this.newTaskModel.taskName = response.task_name;
        this.newTaskModel.taskDesc = response.task_desc;
      }).then(()=>{
        this.loading=false;
      });
    },
    fetchAllTaskInfo: function() {
      return this.$get("/taskinfo/");
    },
    fetchAllDataSet: function() {
      this.$get("/dataSet/").then(response => {
        console.log(response);
        this.dataSetList = response;
      });
    },
    handleClick(tab, event) {
      console.log(tab, event);
    },
    nextStep: function() {
      this.$refs.newChartModel.validate(vaild => {
        if (vaild) {
          if (
            !this.newChartModel.IsChooseHistory &&
            this.$refs.obj.value == ""
          ) {
            this.$message({
              message: "请选择数据源！",
              type: "warning",
              duration: 1500
            });
          } else {
            this.newChartDialogVisible = false;
            if (this.newChartModel.IsChooseHistory) {
              this.newChartTaskDialogVisible = true;
            } else {
              this.tablePreviewVisable = true;
            }
          }
        }
      });
    },
    importFile: function() {
      let obj = this.$refs.obj;
      let filetype = obj.value.substring(
        obj.value.lastIndexOf("."),
        obj.value.length
      );
      let _this = this;
      if (filetype == ".csv") {
        var file = obj.files[0];
        Papa.parse(file, {
          complete: function(results) {
            _this.tablejsons = converterTwoDimArrayToObjectArray(results.data);
            console.log(_this.tablejsons);
          }
        });
      } else {
        let jsonPromise = converterFileToJson(obj);
        jsonPromise.then(data => {
          this.tablejsons = data;
        });
      }
    },
    //设置表头
    setTitleIndex: function(index) {
      this.titleIndex = index;
    },
    createDataSet: function() {},
    watchChoose: function() {
      if (!this.newChartModel.IsChooseHistory) {
        this.$refs.newChartModel.clearValidate();
      }
    },
    createNewDataSet: function() {
      alert("createNewDataSet");
      if (!this.IsChooseHistory) {
        this.$post("/dataSet/", {
          task: this.taskId,
          step1: "1",
          step2: "2",
          step3: "3",
          stepX1: "x1",
          title: this.newChartModel.dataSetTitle,
          row_num: (this.titleIndex - 1).toString(),
          data_set: this.tablejsons
        })
          .then(response => {
            var dataSetId = response.data.id;
            //创建完成之后，跳转到数据处理页面，传任务ID
            this.$router.push(`/home/data-processing/${dataSetId}`);
          })
          .catch(err => {
            console.log(err);
          });
      }
    },
    converterUrlToBase64: function(urlList) {
      return new Promise((resolve, rejects) => {
        this.$post("/operation/image2base64", {
          image_url: urlList
        }).then(response => {
          resolve(response);
        });
      });
    },
    dealAllReportUrl() {
      console.log(this.report.data_set.length);
      for (let i = 0; i < this.report.data_set.length; i++) {
        this.dataAnalysisPicUrl = this.dataAnalysisPicUrl.concat(
          this.report.data_set[i].data_analyze
        );
        this.dataMiningPicUrl = this.dataMiningPicUrl.concat(
          this.report.data_set[i].data_mining_clustering
        );
        this.dataMiningPicUrl = this.dataMiningPicUrl.concat(
          this.report.data_set[i].data_mining_regression
        );
      }
      this.converterUrlToBase64(this.dataAnalysisPicUrl)
        .then(response => {
          this.dataAnalysisPic = response.message;
        })
        .then(() => {
          this.converterUrlToBase64(this.dataMiningPicUrl).then(response => {
            this.dataMiningPic = response.message;
          });
        });
    },
    fetchReport: function() {
      this.$post("/publish/", {
        task_id: this.taskId
      }).then(response => {
        this.report = response;
        const myArray=response.data_set[0];
        this.analyzeArray=myArray.data_analyze;
        this.clusteringArray=myArray.data_mining_clustering;
        this.regressionArray=myArray.data_mining_regression;
        this.dealAllReportUrl();
      });
    },
    postSummary: function() {
      this.$refs.report.postSummary(this.taskId);
    },
    getSummary: function() {
      return new Promise((resolve, rejects) => {
        this.$get("/summary/", {}).then(response => {
          resolve(response);
        });
      });
    },
    generateWord: function() {
      this.postSummary();
      this.$refs.report.generateWord();

      // this.converterUrlToBase64(this.dataAnalysisPicUrl)
      //   .then(response => {
      //     this.dataAnalysisPic = response.message;
      //     this.converterUrlToBase64(this.dataMiningPicUrl).then(response => {
      //       this.dataMiningPic = response.message;
      //       this.$refs.report.generateWord();
      //     });
      //   });
    },
    generateHtml: function() {
      this.postSummary();
      this.$refs.report.generateHtml();
      // this.converterUrlToBase64(this.dataAnalysisPicUrl)
      //   .then(response => {
      //     this.dataAnalysisPic = response.message;
      //     this.converterUrlToBase64(this.dataMiningPicUrl).then(response => {
      //       this.dataMiningPic = response.message;
      //       this.$refs.report.generateHtml();
      //     });
      //   })
    },
    generateReport:function(){
      this.generateReportVisable = true;
      if(typeof this.report == "function"){
        this.fetchReport();
      }
    },
    extendClick(path) {
      if (path == "编辑") {
        this.$router.push({
          name: "data-mining",
          params: {}
        });
      } else {
        this.picPath = path;
        this.picDialogVisible = true;
      }
    },
    changeWordLoading(status){
      this.wordLoading=status;
    }
  },
  watch: {
    $route(to, from) {
      // 对路由变化作出响应...
      if (this.taskId == undefined) {
        this.$router.push("/home/task-release/" + this.taskList[0].id);
      }
      this.fetchTaskInfo();
      this.fetchReport();
    }
  }
};
</script>

<style>
.table-container{
  display: flex;
  flex-wrap: wrap;
}

.task-title {
  border-bottom: 2px solid #ccc;
  padding-bottom: 5px;
  margin-bottom: 20px;
}
.task-title-new {
  float: right;
}
.task-title-name {
  float: left;
  line-height: 42px;
  font-size: 20px;
  font-weight: 550;
  color: #1f2f3d;
  font-family: "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
}
</style>

