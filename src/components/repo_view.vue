<script>
import {computed} from 'vue'
import branchView from './branch_view.vue'
import issue_view from "@/components/issue_view.vue";
import pr_view from "@/components/pr_view.vue";
import topicSetting from "@/utils/topic-setting";

export default {
  name: "repoView",
  components: {
    branchView,
    issue_view,
    pr_view
  },
  data() {
    return {
      selectedRepo: 0,
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
      //     intro: "doc introduction"
      //   }
      // ]
    }
  },
  inject: {
    user: {default: null},
    proj: {default: null},
    bindRepos: {default: null},
    bindReposBusy: {default: null}
  },
  provide() {
    return {
      selectedRepo: computed(() => this.bindRepos[this.selectedRepo])
    }
  },
  methods: {
    getTopicColor: topicSetting.getColor,
    getDarkColor: topicSetting.getDarkColor,
    getRadialGradient: topicSetting.getRadialGradient
  }
}
</script>

<template>
  <v-col cols="12" class="px-1">
    <h2 v-if="bindReposBusy">代码存储库</h2>
    <h2 v-else-if="bindRepos.length > 0">代码存储库 &nbsp; &nbsp;{{ (bindRepos[selectedRepo].repo).replace(/_[0-9]+$/, '') }}</h2>
    <h2 v-else>代码存储库</h2>
    <h4>该仓库ssh:&nbsp;git@github.com:{{ bindRepos[selectedRepo].user}}/{{ bindRepos[selectedRepo].repo}}.git</h4>

    <v-skeleton-loader v-if="bindReposBusy" type="card"></v-skeleton-loader>
    <div v-else-if="bindRepos.length > 0" >
      <v-tabs :background-color="getTopicColor(user.topic)" center-active dark show-arrows v-model="selectedRepo">
        <v-tab class="" v-for="repository in bindRepos" :key="repository.id">{{ (repository.repo).replace(/_[0-9]+$/, '') }}</v-tab>
      </v-tabs>
      <v-tabs-items v-model="selectedRepo">
        <v-tab-item v-for="repository in bindRepos" :key="repository.id">

          <!--<h4>仓库ssh:&nbsp;git@github.com:{{ bindRepos[selectedRepo].user}}/{{ bindRepos[selectedRepo].repo}}.git</h4>-->
          <v-row>
            <v-col class="ma-1">
              <v-card :style="getRadialGradient(user.topic)" raised class="pa-2"  max-height="100%" >
                <branchView/>
              </v-card>
            </v-col>
          </v-row>
          <!--<v-row>-->
          <!--  <v-col class="ma-1">-->
          <!--    <v-card  :style="getRadialGradient(user.topic)" raised class="pa-2 overflow-y-auto">-->
          <!--      <v-card-title>事务</v-card-title>-->
          <!--      <v-card-text>-->
          <!--        <issue_view/>-->
          <!--      </v-card-text>-->
          <!--    </v-card>-->
          <!--  </v-col>-->
          <!--</v-row>-->
          <v-row>
            <v-col class="ma-1">
              <v-card  :style="getRadialGradient(user.topic)" raised class="pa-2 overflow-y-auto"  >
                <v-card-title>和并请求</v-card-title>
                <v-card-text >
                  <pr_view/>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-tab-item>
      </v-tabs-items>
    </div>
    <div v-else>
      <p>这个项目还没有绑定代码存储库哦</p>
    </div>
  </v-col>
</template>