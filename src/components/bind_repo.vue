<script>
import axios from 'axios'
import topicSetting from "@/utils/topic-setting";
import {createRepo, initRepo} from "@/api/user";

export default {
  name: "bindGithubRepo",
  data() {
    return {
      gh_username: '',
      gh_reponame: '',
      git_url: '',
      seperate: true,
      bindingInProgress: false,
      createRepoInProgress: false,
    }
  },
  inject: {
    user: {default: null},
    proj: {default: null},
    updateBindRepos: {default: null}
  },
  methods: {
    bindSplit() {
      this.bindingInProgress = true
      axios.post('/api/userBindRepo',
          {
            userId: this.user.id,
            projectId: this.proj.id,
            repoRemotePath: this.gh_username + '/' + this.gh_reponame +  '_' + this.proj.id,
          }
      )
          .then((res) => {
            this.bindingInProgress = false;
            this.gh_reponame = '';
            this.updateBindRepos()
          })
          .catch((err) => {
            alert('哦不，好像绑定失败了！');
            this.bindingInProgress = false;
          })
    },
    createRepo() {
      this.createRepoInProgress = true;
      axios.post('/api/develop/CreateRepo',{
        name: this.gh_reponame,
        user_id: this.user.id,
        project_id: this.proj.id,
        remote_path: 'https://github.com/LZhengg324/',
      }).then((res) => {
        this.createRepoInProgress = false;
        this.gh_username = 'LZhengg324';
        // this.gh_reponame = this.gh_reponame + '_' + this.proj.id;
        this.$message({
          showClose: true,
          message: "您创建的仓库ssh为：" + res['data']['ssh'],
          type: "success",
          duration: 0,
        });
        this.bindSplit();
      }).catch((err) => {
        alert('哦不，创建失败了！');
        this.createRepoInProgress = false;
      })
    },
    bindWhole() {
      this.bindingInProgress = true
      axios.post('/api/userBindRepo',
          {
            userId: this.user.id,
            projectId: this.proj.id,
            repoRemotePath: this.git_url.replace(/https:\/\/github.com\//, '')
          }
      )
          .then((res) => {
            if (res.data.errcode !== 0) throw new Error()
            this.bindingInProgress = false;
            this.updateBindRepos()
          }).catch((err) => {
        this.$message.error('哦不，好像绑定失败了！');
        this.bindingInProgress = false;
      })
    },
    getTopicColor: topicSetting.getColor,
    getDarkColor: topicSetting.getDarkColor,
  }, watch: {}
}
</script>

<template>
  <div>
    <v-card>
      <v-card-title>输入一个仓库名以创建您的仓库</v-card-title>
      <v-container fluid>
        <v-form v-if="seperate">
          <v-row>

            <v-col cols="12">
              <v-text-field :color="getDarkColor(user.topic)" v-model="gh_reponame"
                            label="Reponame"></v-text-field>
            </v-col>
          </v-row>

          <v-card-actions>
            <v-btn block :color="getTopicColor(user.topic)" @click="createRepo()"
                   :disabled="createRepoInProgress || gh_reponame === ''">
              <v-icon>mdi-link</v-icon>
              为“{{ proj.name }}”创建仓库
              {{ gh_reponame === '' ? '?' : gh_reponame }}
            </v-btn>
          </v-card-actions>
          <!--        <span v-if="bindingInProgress">binding, please wait.</span>-->
        </v-form>
        <v-form v-else>
          <v-row>
            <v-col cols="3" class="text-center my-3">
              <v-btn block @click="seperate = !seperate">https://github.com/</v-btn>
            </v-col>

            <v-col cols="9">
              <v-text-field v-model="git_url" label="github url"></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="4"></v-col>
            <v-col cols="4">
              <v-btn @click="bindWhole()"
                     :disabled="bindingInProgress || git_url === '' || !/[a-zA-Z0-9-_]+\/[a-zA-Z0-9-_]+/.test(git_url.replace(/https:\/\/github.com\//, ''))">
                绑定
                {{ git_url === '' ? '?' : git_url.replace(/https:\/\/github.com\//, '') }}
                到“{{ proj.name }}”
              </v-btn>
            </v-col>
            <v-col cols="4"></v-col>
          </v-row>
          <!--        <span v-if="bindingInProgress">binding, please wait.</span>-->
        </v-form>
      </v-container>
    </v-card>
    <v-divider></v-divider>
    <!--<v-card>-->
    <!--  <v-card-title>请提供GitHub代码存储库的用户/组织名和代码存储库名</v-card-title>-->
    <!--  <v-container fluid>-->
    <!--    <v-form v-if="seperate">-->
    <!--      <v-row>-->
    <!--        <v-col cols="6">-->
    <!--          <v-text-field :color="getDarkColor(user.topic)" v-model="gh_username"-->
    <!--                        label="GitHub Username"></v-text-field>-->
    <!--        </v-col>-->
    <!--        <v-col cols="6">-->
    <!--          <v-text-field :color="getDarkColor(user.topic)" v-model="gh_reponame"-->
    <!--                        label="GitHub Reponame"></v-text-field>-->
    <!--        </v-col>-->
    <!--      </v-row>-->

    <!--      <v-expand-transition>-->
    <!--        <v-card-text v-if="(!(gh_reponame === '' || gh_username === ''))">-->
    <!--          通过点击绑定按钮，您同意JiHub读取代码存储库的文件、提交记录、分支、问题、和并请求等公开可见的信息。<br>-->
    <!--          JiHub<strong>不会</strong>对您的代码存储库做任何修改，也<strong>不会</strong>向您的代码存储库中添加任何文件。<br>-->
    <!--          您可能需要将GitHub代码存储库可见权限设置为公开（public）才能成功绑定。<br>-->
    <!--          （beta测试期间）如果您不愿公开您的仓库，可以添加用户TrickEye为需要绑定的仓库的Collaborator。-->
    <!--        </v-card-text>-->
    <!--      </v-expand-transition>-->

    <!--      <v-card-actions>-->
    <!--        <v-btn block :color="getTopicColor(user.topic)" :href="`https://github.com/${gh_username}/${gh_reponame}`"-->
    <!--               target="_blank" :disabled="bindingInProgress || gh_reponame === '' || gh_username === ''">-->
    <!--          <v-icon>mdi-github</v-icon>-->
    <!--          拿不太准？去GitHub看一眼-->
    <!--        </v-btn>-->
    <!--      </v-card-actions>-->
    <!--      <v-card-actions>-->
    <!--        <v-btn block :color="getTopicColor(user.topic)" @click="bindSplit()"-->
    <!--               :disabled="bindingInProgress || gh_reponame === '' || gh_username === ''">-->
    <!--          <v-icon>mdi-link</v-icon>-->
    <!--          绑定-->
    <!--          {{ gh_username === '' ? '?' : gh_username }} /-->
    <!--          {{ gh_reponame === '' ? '?' : gh_reponame }}-->
    <!--          到“{{ proj.name }}”-->
    <!--        </v-btn>-->
    <!--      </v-card-actions>-->
    <!--      &lt;!&ndash;        <span v-if="bindingInProgress">binding, please wait.</span>&ndash;&gt;-->
    <!--    </v-form>-->
    <!--    <v-form v-else>-->
    <!--      <v-row>-->
    <!--        <v-col cols="3" class="text-center my-3">-->
    <!--          <v-btn block @click="seperate = !seperate">https://github.com/</v-btn>-->
    <!--        </v-col>-->

    <!--        <v-col cols="9">-->
    <!--          <v-text-field v-model="git_url" label="github url"></v-text-field>-->
    <!--        </v-col>-->
    <!--      </v-row>-->
    <!--      <v-row>-->
    <!--        <v-col cols="4"></v-col>-->
    <!--        <v-col cols="4">-->
    <!--          <v-btn @click="bindWhole()"-->
    <!--                 :disabled="bindingInProgress || git_url === '' || !/[a-zA-Z0-9-_]+\/[a-zA-Z0-9-_]+/.test(git_url.replace(/https:\/\/github.com\//, ''))">-->
    <!--            绑定-->
    <!--            {{ git_url === '' ? '?' : git_url.replace(/https:\/\/github.com\//, '') }}-->
    <!--            到“{{ proj.name }}”-->
    <!--          </v-btn>-->
    <!--        </v-col>-->
    <!--        <v-col cols="4"></v-col>-->
    <!--      </v-row>-->
    <!--      &lt;!&ndash;        <span v-if="bindingInProgress">binding, please wait.</span>&ndash;&gt;-->
    <!--    </v-form>-->
    <!--  </v-container>-->
    <!--</v-card>-->

  </div>
</template>
