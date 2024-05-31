<script>
import axios from "axios";
import topicSetting from "@/utils/topic-setting";
import {completeCreateNewPR, completeTask, getDiffString2, getPrDesc, watchMyTask} from "@/api/user";
import Cookies from "js-cookie";

export default {
  name: "commit_view",
  methods: {
    getAiPr(){
      getDiffString2({
        user_id: this.user.id,
        remote_path: this.selectedRepo.user+'/'+this.selectedRepo.repo,
        project_id: this.proj.id,
        source_branch: this.selectedBranch.name,
      }).then((res) =>{
        console.log(res.data.diff_output + "11111111111")
        this.newPRForm.msg = "please wait ..."
        getPrDesc({
          diff: res.data.diff_output,
        }).then((res)=>{
          this.newPRForm.msg = res.data.data.content
          console.log(res)
        })
      })

    },
    updateCommitHistory() {
      this.commitHistoryBusy = true
      axios.post('/api/develop/getCommitHistory', {
        userId: this.user.id,
        projectId: this.proj.id,
        repoId: this.selectedRepo.id,
        branchName: this.selectedBranch.name
      }).then((res) => {
        if (res.data.errcode === 0) {

          let today = new Date()

          // get past 10 days
          let statsPerDay = {}
          for (let i = 0; i < 10; i++) {
            let str = (today.getMonth() + 1) + '.' + today.getDate()
            statsPerDay[str] = 0
            today.setDate(today.getDate() - 1)
          }

          this.commitHistory = res.data.data.map((cur, index) => {
            let time = new Date(cur.commitTime)
            let str = (time.getMonth() + 1) + '.' + time.getDate()
            if (str in statsPerDay)
              statsPerDay[str] += 1
            return {
              id: index,
              committer: cur.author,
              committerEmail: cur.authorEmail,
              hash: cur.commithash,
              message: cur.commitMessage,
              time: cur.commitTime
            }
          })
          this.commitHistoryBusy = false

          this.statsPerDay = {
            label: Object.keys(statsPerDay).reverse(),
            value: Object.values(statsPerDay).reverse()
          }
        } else {
          console.log(res)
          alert('/api/develop/getCommitHistory error with not 0 err code (' + res.data.errcode + ') ' + res.data.message)
          this.commitHistoryBusy = false
        }
      }).catch((err) => {
        alert('/api/develop/getBindRepos error' + err)
        this.commitHistoryBusy = false
      })
    },
    getTopicColor: topicSetting.getColor,
    getDarkColor: topicSetting.getDarkColor,
    createNewPullRequest () {
      var taskIdList = [];
      for (let i = 0; i < this.newPRForm.taskList.length; i++) {
        taskIdList.push(
            this.taskIds[(this.taskNames.indexOf(this.newPRForm.taskList[i]))]);
      }
      console.log("hhh c" + taskIdList);


      completeCreateNewPR({
        title:this.newPRForm.title,
        description: this.newPRForm.msg,
        user_id: JSON.parse(Cookies.get("user")).id,
        project_id: JSON.parse(Cookies.get("proj")).projectId,
        source_branch_name: this.selectedBranch.name,
        repo_id:this.selectedRepo.id,
        tasks: taskIdList,
      }).then(
          res => {
            if (res.data.errcode === 2) {
              this.$message({
                type: 'fail',
                message: '提交失败!'
              })
            } else if (res.data.errcode === 0) {
              this.$message({
                type: 'success',
                message: '提交成功!'
              });
            } else if (res.data.errcode === 4) {
              this.$message({
                type: 'fail',
                message: '您无法对项目进行提交！'
              })
            }
          }
      )
      this.createNewPR = false;
    },
    handleClose(done) {
      this.$confirm('确认关闭？')
          .then(() => {
            done();
          })
          .catch(() => {
          });
    },
    getTaskNameList() {
      console.log("get name list!");
      for (let i = 0; i < this.tasks.length; i++) {
        var tmp_taskName = this.tasks[i].taskName;
        // this.taskNames.push(this.tasks[i].taskName);
        for (let j = 0; j < this.tasks[i].subTaskList.length; j++) {
          this.taskNames.push(tmp_taskName + "--" + this.tasks[i].subTaskList[j].subTaskName);
          this.taskIds.push(this.tasks[i].subTaskList[j].subTaskId);
        }
      }
      console.log("hhh" + this.taskNames);
      console.log("hhh" + this.taskIds);
    },
    checkMyTask() {
      //checkMyTask
      console.log(this.checkMyFlag);
      // console.log("cby checkMyTask  " + user.id + "  " + proj.projectId);
      this.checkMyFlag = true;

      watchMyTask({
        userId: JSON.parse(Cookies.get("user")).id,
        projectId: JSON.parse(Cookies.get("proj")).projectId
      }).then(
          res => {
            console.log(res);
            this.tasks = res['data']['data'];
            console.log("tasks:   " + this.tasks);
            this.getTaskNameList();
          }
      );

    },


  },
  data() {

    return {
      commitHistoryBusy: true,
      commitHistory: [
        {}
      ],
      statsPerDay: {},
      createNewPR: false,
      newPRForm: {
        title:'',
        msg: '',
        taskList: [],
      },
      checkMyFlag: false,
      tasks: [],
      taskNames: [],
      taskIds:[],

    }
  }, watch: {
    selectedBranch() {
      this.updateCommitHistory()
    }
  }, created() {
    this.updateCommitHistory();
    this.checkMyTask();
  }, inject: {
    user: {default: null},
    proj: {default: null},
    selectedRepo: {default: null},
    selectedBranch: {default: null}
  }
}
</script>

<template>
  <div>
    <v-card-title>
      分支
      <span class="need-mono">
        “{{ selectedBranch.name }}”
      </span>上的提交记录
    </v-card-title>

    <p v-if="!commitHistoryBusy">分支<span class="need-mono"> “{{ selectedBranch.name }}” </span>中有
      {{ commitHistory.length }} 条提交记录。最新的提交记录：</p>
    <p v-else>正在与服务器同步分支{{ selectedBranch.name }}上的最新提交记录...</p>
    <div v-if="!commitHistoryBusy">
      <v-simple-table dense height="200px" >
        <thead>
        <tr>
          <th class="committer">committer</th>
          <th class="message">message</th>
          <th class="hash">hash</th>
          <th class="time">time</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="commit in commitHistory" :key="commit.id">
          <td class="need-mono">{{ commit.committer }}</td>
          <td>{{ commit.message }}</td>
          <td class="need-mono">
            <v-tooltip bottom>
              <template v-slot:activator="{on, attrs}">
                <span v-bind="attrs" v-on="on">{{ commit.hash.slice(0, 6) }}</span>
              </template>
              <span>{{ commit.hash }}</span>
            </v-tooltip>
          </td>
          <td>{{ new Date(commit.time).toLocaleString() }}</td>
        </tr>
        </tbody>
      </v-simple-table>
      <br>
      <v-divider></v-divider>
      <br>
      <v-form>
        <v-row>
          <v-col cols="12" class="mb-3">
            <v-btn :color="getTopicColor(user.topic)"
                   class="white--text"
                   block
                   @click="createNewPR = true">
              创建PullRequest
            </v-btn>
          </v-col>
        </v-row>
      </v-form>
      <br>
      <v-divider></v-divider>
      <br>
      <v-card-title>过去10天的提交记录数量</v-card-title>
      <v-sparkline
          :labels="statsPerDay.label"
          :value="statsPerDay.value"
          :color="getDarkColor(user.topic)"
          auto-line-width
          smooth
          padding="20"
          stroke-linecap="round"
          show-labels
          auto-draw
      ></v-sparkline>
      <br>
      <v-divider></v-divider>
      <br>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn width="10rem" :color="getTopicColor(user.topic)" link
               :href="'https://github.com/' + selectedRepo.user + '/' + selectedRepo.repo + '/tree/' + selectedBranch.name"
               target="_blank">
          <v-icon>mdi-github</v-icon>
          在GitHub查看
        </v-btn>
        <v-btn width="10rem" :color="getTopicColor(user.topic)" link
               :to="'/dev/' + proj.id + '/' + selectedRepo.id + '/' + selectedBranch.name + '/'" target="_blank">
          <v-icon>mdi-send</v-icon>
          浏览详情
        </v-btn>
      </v-card-actions>
    </div>
    <v-skeleton-loader v-else type="table" class="mx-auto"/>


    <el-dialog
        title="创建PullRequest"
        :visible.sync="createNewPR"
        width="50%"
        :before-close="handleClose">
      <el-form label-position="labelPosition"
               label-iwdth="80px"
               :model="newPRForm"
               ref="newPRForm">
        <el-form-item label="PR title">
          <el-input  v-model="newPRForm.title"></el-input>
        </el-form-item>
        <el-form-item label="PR message">
          <v-row>
            <v-col cols="6">
              <!--<v-btn block :color="getTopicColor(user.topic)" :href="`https://github.com/`"-->
              <!--       target="_blank" >-->
              <!--  <v-icon>mdi-message</v-icon>-->
              <v-btn block :color="getTopicColor(user.topic)"
                     @click="getAiPr()">
                <v-icon>mdi-message</v-icon>
                不想写了，帮我写一下
              </v-btn>
            </v-col>
          </v-row>
          <el-input type="textarea" v-model="newPRForm.msg"></el-input>
        </el-form-item>
        <p style="top:5%">对应任务</p>
        <v-select
            outlined
            chips
            multiple
            v-model="newPRForm.taskList"
            :items="taskNames"
        >
          <template v-slot:item="{ item }">
            <div style="position: relative;background-color: aliceblue; ">
              <v-avatar size="25" color="indigo">
                <v-icon class="mr-2" :color="getTopicColor(user.topic)">
                  mdi-bullseye-arrow
                </v-icon>
              </v-avatar>
              <span style="position:absolute;left: 120%; width: 500px">
              {{ item }}
            </span>
            </div>
          </template>
        </v-select>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="createNewPR = false">取 消</el-button>
    <el-button  type="primary" @click="createNewPullRequest">确 定</el-button>
    </span>
    </el-dialog>

  </div>
</template>


<style scoped>
.need-mono {
  font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>