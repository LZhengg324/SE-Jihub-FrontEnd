
<script>
import axios from "axios";
import topicSetting from "@/utils/topic-setting";

import Cookies from "js-cookie";
import {getDiffString} from "@/api/user";

export default {
  name: "pr_view",
  data() {
    return {
      prs: [
        {
          id: 1,
          author: 'TrickEye',
          title: 'PR title1',
          date: '20230403',
          fromBranchId: 2,
          toBranchId: 1,
          isOpen: true
        }, {
          id: 2,
          author: 'TrickEyeee',
          title: 'PR title2',
          date: '20230402',
          fromBranchId: 3,
          toBranchId: 1,
          isOpen: true
        }
      ],
      prsBusy: true,
      atAudit:false,

    }
  }, methods: {
      updatePR() {
          this.prsBusy = true
          axios.post('/api/develop/getPrList', {
              userId: this.user.id,
              projectId: this.proj.id,
              repoId: this.selectedRepo.id
          }).then((res) => {
              if (res.data.errcode === 0) {
                  let prs = res.data.data.map((cur, index, arr) => {
                      return {
                          id: cur.prId,
                          author: cur.prIssuer,
                          title: cur.prTitle,
                          date: cur.prTime,
                          isOpen: cur.isOpen,
                          ghLink: cur.ghLink,
                          fromBranchName: cur.fromBranchName,
                          toBranchName: cur.toBranchName
                      }
                  })
                  this.prs = prs
                  this.prsBusy = false
              } else {
                  console.log('get pr failure with not 0 err code + {' + res.data.errcode + ')' + res.data.message)
                  this.prsBusy = false
              }
          }).catch((err) => {
              console.log('get pr failure with err: ' + err)
              this.prsBusy = false
          })
      },
    handleClose(done) {
      this.$confirm('确认关闭？')
          .then(() => {
            done();
          })
          .catch(() => {
          });
    },
    changeToAuditView(PrToAudit){
      Cookies.set("PrToAudit_ReadOnly", false);
      getDiffString({
        user_id: this.user.id,
        remote_path: this.selectedRepo.user+'/'+this.selectedRepo.repo,
        project_id: this.proj.id,
        source_branch: PrToAudit.fromBranchName,
        ghpr_id:PrToAudit.id,
      }).then(
          res =>{
            console.log(res);
            this.$message({
              type: 'success',
              message: '正在进入！'
            });
            console.log("cbycby2" + res['data']['diff_output']);
            localStorage.setItem("diffString", res['data']['diff_output']);
            localStorage.setItem("comment", res['data']['comment']);
            localStorage.setItem("title", res['data']['title']);
            localStorage.setItem("description",res['data']['description']);
            // Cookies.set("diffString",JSON.stringify(res['data']['diff_output']));
            console.log("cbycby3" + res['data']['diff_output']);
            Cookies.set("selectedRepo", JSON.stringify(this.selectedRepo));
            Cookies.set("PrToAudit", JSON.stringify(PrToAudit));
            Cookies.set("PrToAudit_ReadOnly", false);

            this.$router.push({ name: 'audit' });
          }
      )
    },
    getTopicColor: topicSetting.getColor
  }, inject: {
        user: {default: null},
        proj: {default: null},
        selectedRepo: {default: null}
  }, created() {
      this.updatePR()
  }, provide(){
    return {
      selectedRepo: this.selectedRepo,
    }
  }
}
</script>

<template>
<div>
<!--  <p>This is pr view</p>-->

  <v-skeleton-loader v-if="this.prsBusy" type="button, table" />
  <v-row v-else-if="this.prs.length > 0">
    <v-col>
      <v-simple-table dense max-height="500px">
        <tbody>
        <tr v-for="pr in prs" :key="pr.id">
          <td>#{{pr.id}} ({{pr.isOpen ? '开启' : '已关闭'}})</td>
          <td>{{pr.author}}</td>
          <td>{{pr.title}}</td>
          <td>从分支“{{pr.fromBranchName}}”合并到“{{pr.toBranchName}}”</td>
          <td>
            <v-form class="justify-center">
              <v-row>
                <v-col>
                  <v-btn :color="getTopicColor(user.topic)"
                         class="white--text"
                         block
                         @click="changeToAuditView(pr)">
                    Pr审核
                  </v-btn>
                </v-col>
              </v-row>
            </v-form>
          </td>
        </tr>
        </tbody>
      </v-simple-table>
    </v-col>
  </v-row>
  <v-row v-else>
  <v-col>
    <p>合并请求似乎空空如也？现在就去GitHub上发一个吧！</p>
  </v-col>
  </v-row>

  <v-card-actions>
    <v-spacer></v-spacer>
    <v-btn :color="getTopicColor(user.topic)" link :href="'https://github.com/' + selectedRepo.user + '/' + selectedRepo.repo + '/pulls'" target="_blank">
      <v-icon>mdi-github</v-icon>在GitHub浏览
    </v-btn>
  </v-card-actions>

  <!--<el-dialog-->
  <!--    title="审核代码"-->
  <!--    :visible.sync="atAudit"-->
  <!--    width="100%"-->
  <!--    height="100%"-->
  <!--    :before-close="handleClose"-->
  <!--class="fullHeight">-->

    <!--<iframe src="/diff2html_ui.html"-->
    <!--        class="justify-center"-->
    <!--        width="100%"-->
    <!--        height="100%"-->
    <!--&gt;</iframe>-->
    <!--<div id="diff2htmlElement"></div>-->


  <!--  <span slot="footer" class="dialog-footer">-->
  <!--  <el-button @click="atAudit = false">取 消</el-button>-->
  <!--  <el-button type="primary" @click="createNewBranch">确 定</el-button>-->
  <!--  </span>-->
  <!--</el-dialog>-->

</div>
</template>


<style scoped>

.fullHeight /deep/ .el-dialog__body {
  height: 90vh;
  overflow: auto;
}



</style>