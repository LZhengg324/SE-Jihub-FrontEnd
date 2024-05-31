<script>
import Cookies from "js-cookie";
import {approvePullRequest, closePullRequest, getCommentsList, makeComment, markTask} from "@/api/user";
import getIdenticon from "@/utils/identicon";
import topicSetting from "@/utils/topic-setting";

export default {
  name: "audit_view",
  data() {
    return {

      form: {
        name: "",
        comment: '',
      },
      PrToAudit: {},
      selectedRepo: {},
      PrToAudit_ReadOnly: false,
      prtitle: "",
      prdescription: "",

      events: [],
      input: null,
      nonce: 0,

      uuser: {},
      allData: [],
      comments: [
        {
          "comment id": 6,
          "user id": 11,
          "user name": "LeeZheng",
          "content": "1这个实现的不错，但是希望可以hhhhh",
          "created at": "2024-05-22T17:37:32.570Z"
        },
        {
          "comment id": 6,
          "user id": 11,
          "user name": "LeeZheng",
          "content": "2这个实现的不错，但是希望可以hhhhh",
          "created at": "2024-05-22T17:37:32.570Z"
        },
        {
          "comment id": 6,
          "user id": 11,
          "user name": "LeeZheng",
          "content": "3这个实现的不错，但是希望可以hhhhh",
          "created at": "2024-05-22T17:37:32.570Z"
        },
        {
          "comment id": 6,
          "user id": 11,
          "user name": "LeeZheng",
          "content": "4这个实现的不错，但是希望可以hhhhh",
          "created at": "2024-05-22T17:37:32.570Z"
        },
        {
          "comment id": 6,
          "user id": 11,
          "user name": "LeeZheng",
          "content": "5这个实现的不错，但是希望可以hhhhh",
          "created at": "2024-05-22T17:37:32.570Z"
        },
      ],
    }
  },
  inject: {
    user: {default: null},
    proj: {default: null},

  },

  computed: {
    timeline() {
      return this.events.slice().reverse()
    },
    items() {
      return Array.from({length: this.length}, (k, v) => v + 1)
    },
    length() {
      return 7000
    },

  },
  mounted() {
    this.PrToAudit_ReadOnly = this.user.id !== this.proj.managerId || !JSON.parse(Cookies.get("PrToAudit")).isOpen;
  },
  created() {

    this.form.comment = localStorage.getItem('comment');
    this.prtitle = localStorage.getItem('title');
    this.prdescription = localStorage.getItem("description");
    if (!this.PrToAudit_ReadOnly)
      this.PrToAudit = JSON.parse(Cookies.get("PrToAudit"));
    this.selectedRepo = JSON.parse(Cookies.get("selectedRepo"));

    this.uuser = JSON.parse(Cookies.get('user'))
    console.log("hhhh" + this.uuser.name)

    getCommentsList({
      ghpr_id: this.PrToAudit.id,
      project_id: this.proj.projectId,
    }).then((res) => {
      this.allData = res.data.data;
      for (let i = 0; i < this.allData.length; i++) {
        this.$set(this.allData[i], "input", "");
        this.$set(this.allData[i], "extend", false);
      }
    })
  },
  methods: {
    getIdenticon,
    markTaskSolved(taskTo){
      markTask({
        comment_id: taskTo,
      }).then(
          getCommentsList({
            ghpr_id: this.PrToAudit.id,
            project_id: this.proj.projectId,
          }).then((res) => {
            this.allData = res.data.data;
            for (let i = 0; i < this.allData.length; i++) {
              this.$set(this.allData[i], "input", "");
              this.$set(this.allData[i], "extend", false);
              // if(this.allData[i].task_id === taskTo){
              //   this.allData[i].extend = true;
              // }
            }
          })
      );
    },
    comment(taskTo, msg) {
      // const time = (new Date()).toTimeString()
      // this.events.push({
      //   id: this.nonce++,
      //   text: this.input,
      //   time: time.replace(/:\d{2}\sGMT-\d{4}\s\((.*)\)/, (match, contents, offset) => {
      //     return ` ${contents.split(' ').map(v => v.charAt(0)).join('')}`
      //   }),
      // })
      //
      // this.input = null
      makeComment({
        user_id : this.user.id,
        project_id: this.proj.projectId,
        ghpr_id: this.PrToAudit.id,
        parent_comment: taskTo,
        content:msg,
      }).then((res)=>{
        getCommentsList({
          ghpr_id: this.PrToAudit.id,
          project_id: this.proj.projectId,
        }).then((res) => {
          this.allData = res.data.data;
          for (let i = 0; i < this.allData.length; i++) {
            this.$set(this.allData[i], "input", "");
            this.$set(this.allData[i], "extend", false);
            if(this.allData[i].task_id === taskTo){
              this.allData[i].extend = true;
            }
          }
        })
      })
    },
    sendDataToIframe() {
      const diffString = localStorage.getItem('diffString');
      // const diffString = Cookies.get('diffString');
      console.log("cbycby" + diffString);
      this.$refs.showDiffHtml.contentWindow.postMessage(
          diffString);
    },
    turnBack() {
      this.$router.back();
    },
    createNewComment() {
      closePullRequest({
        ghpr_id: this.PrToAudit.id,
        repo_id: this.selectedRepo.id,
        comment: this.form.comment,
      }).then((res) => {
        alert('您拒绝了一个PullRequest并提出修改意见！');
      }).catch((err) => {
        alert('哦不，好像失败了！');
      })
    },
    approvePr() {
      approvePullRequest({
        ghpr_id: this.PrToAudit.id,
        repo_id: this.selectedRepo.id,
        comment: this.form.comment,
      }).then((res) => {
        alert('您同意了一个PullRequest');
      }).catch((err) => {
        alert('哦不，好像失败了！');
      })
    },
    getTopicColor: topicSetting.getColor,

  }
}
</script>

<template>
  <v-container>
    <div style="display: flex; flex-direction: row; height: 100vh; ">
      <v-row>
        <v-col cols="7">
          <div style="height: 100vh; flex: 7">
            <iframe
                ref="showDiffHtml"
                src="/diff2html_ui.html"
                :style="{ 'min-width': '100%', 'min-height': '100%' }"
                @load="sendDataToIframe"
            ></iframe>
          </div>
        </v-col>
        <v-divider vertical></v-divider>

        <v-col cols="5">
          <br>
          <v-row v-for="dataDetail in allData" :key="dataDetail.task_id">
            <v-card width="800" class="mx-auto">
              <v-app-bar flat :color="getTopicColor(user.topic)" height="60">
                <v-btn :color="getTopicColor(user.topic)" @click="markTaskSolved(dataDetail.task_id)"
                            :disabled="dataDetail.solved || PrToAudit_ReadOnly">
                  <v-icon>mdi-check-underline </v-icon>
                  确认完成
                </v-btn>
                &nbsp;
                &nbsp;
                &nbsp;
                <v-toolbar-title class="text-h6 black--text pl-0">
                  任务：{{ dataDetail.task_name }}
                </v-toolbar-title>
                <v-spacer></v-spacer>

                <v-btn  color="black" icon @click="dataDetail.extend=!dataDetail.extend">
                  <v-icon v-if="dataDetail.extend">mdi-chevron-down</v-icon>
                  <v-icon v-else >mdi-chevron-left</v-icon>
                </v-btn>
              </v-app-bar>
              <v-container v-if="dataDetail.extend">



              <v-virtual-scroll :items="dataDetail.comments" height="500" item-height="100">
                <template v-slot:default="{ item }">
                  <v-row>
                    <v-col cols="2">
                      <v-avatar active-class="deep-purple--text text--accent-4" color="indigo">
                        <v-img :src="getIdenticon(item.user_name, 20, 'user')" sizes="20"></v-img>
                      </v-avatar>
                    </v-col>
                    <v-col cols="10">
                      <v-row>
                        <v-col cols="4">
                          <v-chip :color="getTopicColor(user.topic)">
                            {{item.user_name}}
                          </v-chip>

                        </v-col>
                        <v-col cols="8">
                          {{item.created_at}}
                        </v-col>
                      </v-row>
                      <v-row>
                        <v-col cols="1">&nbsp;</v-col>
                        <v-col cols="10">
                          <h2>{{item.content}}</h2>

                        </v-col>
                        <v-col cols="1">&nbsp;</v-col>
                      </v-row>
                      <!--<v-col cols="1">-->
                      <!--  {{item.user_name}}-->
                      <!--</v-col>-->
                      <!--<v-col cols="6">-->
                      <!--  {{ item.content }}-->
                      <!--</v-col>-->
                      <!--<v-col cols="3">-->
                      <!--  {{ item.created_at }}-->
                      <!--</v-col>-->
                    </v-col>
                  </v-row>

                  <v-divider></v-divider>
                </template>
              </v-virtual-scroll>
              <v-text-field v-if="!dataDetail.solved"  v-model="dataDetail.input" hide-details flat label="Press Enter to Send Msg..." solo
                            @keydown.enter="comment(dataDetail.task_id, dataDetail.input); dataDetail.input = null">
                <template v-slot:append>
                  <v-btn class="mx-0" depressed @click="comment(dataDetail.task_id, dataDetail.input); dataDetail.input = null">
                    Post
                  </v-btn>
                </template>
              </v-text-field>
              </v-container>
            </v-card>
          </v-row>
          <v-divider></v-divider>
          <br><br>
          <h1>
            最终审核意见：
          </h1>
          <br>

          <v-row width="800" >
            <el-form :model="form" ref="form" >
              <el-form-item style="width: 250%">
                <el-input v-model="form.comment" type="textarea" :disabled="PrToAudit_ReadOnly"></el-input>
              </el-form-item>
            </el-form>
          </v-row>
          <v-row>
            <el-button v-if="!PrToAudit_ReadOnly" @click="turnBack">取 消</el-button>
            <el-button v-if="!PrToAudit_ReadOnly" type="primary" @click="createNewComment()">不通过</el-button>
            <el-button v-if="!PrToAudit_ReadOnly" type="primary" @click="approvePr()">通过</el-button>
          </v-row>
          <!--</div>-->
        </v-col>

      </v-row>
    </div>
  </v-container>

</template>


<style scoped>

</style>
