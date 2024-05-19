<script>
import {computed} from "vue";
import commit_view from "@/components/commit_view.vue";
import axios from "axios";
import topicSetting from "@/utils/topic-setting";
import {completeCreateNewBranch} from "@/api/user";

export default {
  name: "branchView",
  components: {
    commit_view
  },
  methods: {
    updateBranches() {
        this.branchBusy = true
        axios.post('/api/develop/getRepoBranches', {
            userId: this.user.id,
            repoId: this.selectedRepo.id,
            projectId: this.proj.id
        }).then((res) => {
            if (res.data.errcode === 0) {
                this.branches = res.data.data.map((cur, index) => {
                    return {
                        id: index,
                        name: cur.branchName,
                        lastCommit: {
                            sha: cur.lastCommit.sha,
                            authorName: cur.lastCommit.authorName,
                            authorEmail: cur.lastCommit.authorEmail,
                            commitDate: cur.lastCommit.commitDate,
                            commitMessage: cur.lastCommit.commitMessage
                        }
                    }
                })
                this.sortBranches()
                this.branchBusy = false
            } else {
                console.log(res);
                alert('/api/develop/getRepoBranches error with not 0 err code (' + res.data.errcode + ') ' + res.data.message)
                this.branchBusy = false
            }
        }).catch((err) => {
            alert('/api/develop/getBindRepos error' + err)
            this.branchBusy = false
        })
    },
    sortBranches() {
      // 按照时间排序
      this.branches.sort((a, b) => {
        return a.lastCommit.commitDate > b.lastCommit.commitDate ? -1 : 1
      })
      console.log(this.branches)
    },
    handleClose(done) {
      this.$confirm('确认关闭？')
          .then(() => {
            done();
          })
          .catch(() => {
          });
    },
    createNewBranch(){
      for(let i=0;i<this.branches.length;i++){
        if(this.branches[i].name === this.newBranchForm.branchName){
          this.$message({
            type: "error",
            message: '已存在同名Branch'
          })
          return;
        }
      }
      console.log("cbycby " + this.proj.id + "cbycbycby")
      axios.post("/api/develop/createBranch",{
        name: this.newBranchForm.branchName,
        project_id:this.proj.id,
        remote_path:this.selectedRepo.user+'/'+this.selectedRepo.repo,
        user_id: this.user.id
      }).then(
          res => {
            this.$message({
              type: 'success',
              message: '提交成功!'
            });
          }
      )

      this.creatingNewBranch = false;
    },
    getTopicColor: topicSetting.getColor,
  },
  data() {
    return {
      selectedBranchIndex: 0,
      branches: [{
          id: 0,
          name: 'master',
          lastCommit: {}
      }],
      branchBusy: true,
      creatingNewBranch:false,
      newBranchForm:{
        branchName:"",

      }
    }
  }, inject: {
    user: {default: null},
    proj: { default: null },
    selectedRepo: { default: null }
  }, provide() {
    return {
      selectedBranch: computed(() => this.branches[this.selectedBranchIndex])
    }
  }, created() {
      this.updateBranches()
  }
}
</script>

<template>
<div>
<!--  <p>I am branch view, I am aware that my proj = {{ proj }}, and that my selected repo = {{ selectedRepo }}</p>-->
<v-row>
  <v-col cols="4">
    <v-card-title>
      <v-form>
        <v-row class="justify-center">分支列表</v-row>
        <!--<v-row class="justify-center">-->
        <!--  <v-col  class="mb-3">-->
        <!--    <v-btn :color="getTopicColor(user.topic)"-->
        <!--           class="white&#45;&#45;text"-->
        <!--           block-->
        <!--           @click="creatingNewBranch = true">-->
        <!--      创建新的分支-->
        <!--    </v-btn>-->
        <!--  </v-col>-->
        <!--</v-row>-->
      </v-form>
    </v-card-title>
    <div v-if="branchBusy">
        <v-card-title><v-progress-circular indeterminate></v-progress-circular>正在与服务器同步分支</v-card-title>
    </div>
    <v-list outlined shaped v-if="!branchBusy" class="overflow-y-auto" max-height="750px"  >
      <v-list-item-group v-model="selectedBranchIndex" mandatory >
        <v-list-item v-for="branch in branches" :key="branch.id" two-line>
          <v-list-item-content>
            <v-list-item-title>
              <div style="display: inline-block;" class="need-mono">{{ branch.name }}</div>
              <div class="subtitle-2" style="display: inline-block; float: right">更新于{{new Date(branch.lastCommit.commitDate).toLocaleString() }}</div>
            </v-list-item-title>
            <v-list-item-subtitle>{{ branch.lastCommit.commitMessage }}</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
      </v-list-item-group>
    </v-list>
      <v-skeleton-loader v-else type="list-item-three-line"></v-skeleton-loader>
<!--      <v-progress-circular v-else indeterminate></v-progress-circular>-->
  </v-col>

  <v-divider vertical></v-divider>

  <v-col cols="8">
    <commit_view v-if="!branchBusy"/>
    <v-skeleton-loader v-else type="text@3, table"></v-skeleton-loader>
  </v-col>
</v-row>

  <!--<el-dialog-->
  <!--    title="创建新的分支"-->
  <!--    :visible.sync="creatingNewBranch"-->
  <!--    width="50%"-->
  <!--    :before-close="handleClose">-->
  <!--  <el-form label-position="labelPosition"-->
  <!--           label-iwdth="80px"-->
  <!--           :model="newBranchForm"-->
  <!--           ref="newBranchForm">-->
  <!--    <el-form-item label="Branch name">-->
  <!--      <el-input type="textarea" v-model="newBranchForm.branchName" ></el-input>-->
  <!--    </el-form-item>-->
  <!--  </el-form>-->
  <!--  <span slot="footer" class="dialog-footer">-->
  <!--  <el-button @click="creatingNewBranch = false">取 消</el-button>-->
  <!--  <el-button type="primary" @click="createNewBranch">确 定</el-button>-->
  <!--  </span>-->
  <!--</el-dialog>-->
</div>
</template>



<style scoped>
.need-mono {
  font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>