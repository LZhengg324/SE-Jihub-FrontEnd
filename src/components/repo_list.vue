<script>

import axios from "axios";
import topicSetting from "@/utils/topic-setting";
import {checkIsCollaborator, doAuthenticate} from "@/api/user";

export default {
    name: "bindedGithubRepos",
    inject: {
        user: { default: null },
        proj: { default: null },
        bindRepos: {default: null},
        bindReposBusy: {default: null},
        updateBindRepos: {default: () => {}} // default is a function that returns nothing
    },
    data() {
        return {
          // bondedRepos: [
          //   {
          //     id: 1,
          //     user: "opencv",
          //     repo: "opencv",
          //     intro: "opencv introduction. A Bonded repo of #" + this.proj.id + " called " + this.proj.name
          //   }, {
          //     id: 2,
          //     user: "buaa2023sw",
          //     repo: "doc",
          //     intro: "doc introduction. A Bonded repo of #" + this.proj.id + " called " + this.proj.name
          //   }
          // ]
          repoTodo:{},
          authenticating: false,
          aythenticForm:{
            githubUsername:'',
            githubMail:'',
          }
        }
    },
    methods: {

        unbind(repo) {
            axios.post('/api/develop/userUnbindRepo',
                {
                    userId: this.user.id,
                    projectId: this.proj.id,
                    repoId: repo.id
                }).then((res) => { this.updateBindRepos() })
                .catch((err) => { alert('哦不，好像解绑失败了！'); this.bindingInProgress = false; })
        },
        authenticate(){
          checkIsCollaborator({
            // repo_id = kwargs['repo_id']
            // username = kwargs['username']
            repo_id: this.repoTodo.id,
            username: this.aythenticForm.githubUsername,
          }).then((res)=>{
            if(res.data.errcode === 0)
              this.$message({
                type:'error',
                message: "您输入的github账户已经在该仓库中啦！",
                duration: 0,
                showClose: true
              })
            else if(res.data.errcode === 1)
            {
              doAuthenticate({
                // githubUsername: this.aythenticForm.githubUsername,
                // githubMail: this.aythenticForm.githubMail
                repo_id: this.repoTodo.id,
                username: this.aythenticForm.githubUsername,

              }).then((res)=>
                  {
                    this.authenticating = false;
                    if(res.data.errcode === 0)
                      this.$message({
                        type:'success',
                        message: "已向您的发送邀请，请注意查收！",
                        duration: 0,
                        showClose: true
                      })
                    else
                      this.$message({
                        type:'error',
                        message: "认证失败！",
                        duration: 0,
                        showClose: true
                      })
                    // else if(res.data.errcode === 1)
                    //   this.$message({
                    //     type:'success',
                    //     message: "您已拥有权限啦 无需再次认证！",
                    //     duration: 0,
                    //     showClose: true
                    //   })
                    // else if(res.data.errcode === 2)
                    //   this.$message({
                    //     type:'error',
                    //     message: "认证失败！",
                    //     duration: 0,
                    //     showClose: true
                    //   })
                  }
              )
            }
            else {
              this.$message({
                type:'error',
                message: "认证失败，请重新认证！",
                duration: 0,
                showClose: true
              })
            }

          })

        },
        getTopicColor: topicSetting.getColor,
    }
}

</script>

<template>
    <div >
        <h2>已绑定的代码存储库</h2>
        <p v-if="!bindReposBusy">项目“{{ proj.name }}”已绑定 {{ bindRepos.length }} 个代码存储库。</p>
        <p v-else>正在加载项目“{{ proj.name }}”已绑定的代码存储库列表...</p>
        <v-divider></v-divider>
        <br/>
        <v-skeleton-loader v-if="bindReposBusy"
                type="list-item, list-item, list-item"
        ></v-skeleton-loader>

        <v-expansion-panels v-else-if="bindRepos.length > 0" >
            <v-expansion-panel
                    v-for="repo in bindRepos"
                    :key="repo.id"
            >
                <v-expansion-panel-header>
                    代码仓库名:&nbsp;{{ repo.repo.replace(/_[0-9]+$/, '') }}
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <div>
                    <h4>仓库ssh:&nbsp;git@github.com:{{repo.user}}/{{repo.repo}}.git</h4>
                  </div>
                        <!--<div class="description" v-if="repo.intro !== ''">{{ repo.intro }}</div>-->
                        <!--<div class="description" v-else>存储库的简介空空如也呢</div>-->
                  <br>
                  <v-row>
                    <v-col cols="4"  class="text-center"><v-btn :color="getTopicColor(user.topic)" link :href="'https://github.com/' + repo.user + '/' + repo.repo" target="_blank"><v-icon>mdi-github</v-icon>GitHub</v-btn></v-col>
                    <v-col cols="4"  class="text-center"><v-btn :color="getTopicColor(user.topic)" @click="authenticating = true; repoTodo = repo"><v-icon>mdi-account-arrow-up</v-icon>认证</v-btn></v-col>
                    <v-col cols="4"  class="text-center"><v-btn :color="getTopicColor(user.topic)" @click="unbind(repo)"><v-icon>mdi-link-variant-off</v-icon>Unbind</v-btn></v-col>
                  </v-row>
                </v-expansion-panel-content>
            </v-expansion-panel>
        </v-expansion-panels>

        <v-card v-else >
            <v-card-title>
                <v-icon>mdi-github</v-icon>
                <span class="headline ml-3">还没有绑定代码存储库</span>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-subtitle>
                <span class="subtitle-1 ml-4">唔，快去绑定一个代码存储库开始记录你的工作吧！</span>
            </v-card-subtitle>
        </v-card>


      <el-dialog
          title="输入GitHub账户名和邮箱以向您发送协作邀请"
          :visible.sync="authenticating"
          width="50%">
        <el-form label-position="labelPosition"
                 label-iwdth="80px"
                 :model="aythenticForm"
                 ref="aythenticForm">
          <el-form-item label="您的github账户名">
            <el-input  v-model="aythenticForm.githubUsername"></el-input>
          </el-form-item>
          <el-form-item label="您的github邮箱">
            <el-input v-model="aythenticForm.githubMail"></el-input>
          </el-form-item>
          <v-row>
            <v-col cols="12">
              <v-btn block :color="getTopicColor(user.topic)" :href="`https://github.com/${aythenticForm.githubUsername}`"
                     target="_blank" :disabled="aythenticForm.githubUsername === '' || aythenticForm.githubMail === ''">
                <v-icon>mdi-github</v-icon>
                拿不太准？去GitHub看一眼
              </v-btn>
            </v-col>
          </v-row>
        </el-form>
        <span slot="footer" class="dialog-footer">

          <v-row>
            <el-button @click="authenticating = false">取 消</el-button>
            <el-button  type="primary" @click="authenticate()">确 定</el-button>
          </v-row>
        </span>
      </el-dialog>

    </div>
</template>

<style scoped>
.description {
    word-break:normal;
    width:auto;
    display:block;
    white-space:pre-wrap;
    word-wrap : break-word ;
    overflow: hidden ;
    line-height: 1.5rem;
}




</style>