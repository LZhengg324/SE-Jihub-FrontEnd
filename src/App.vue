<template>
  <div>


  <v-app id="main_page">
    <v-app-bar app fixed clipped-left ref="appBar" color="white" dark extension-height="36"
               :src=topic>
      <v-toolbar-title style="font-weight: bold">JiHub<span v-if="existManager()"> - Admin Override</span>
      </v-toolbar-title>

      <v-spacer></v-spacer>


      <v-icon id="v-step-remind" v-if="!existUnreadNote" style="right: 1%" @click="checkNote">mdi-bell</v-icon>
      <v-icon id="v-step-remind" v-else style="right: 1%" @click="checkNote">mdi-bell-badge</v-icon>

      <v-icon v-if="existUser()" @click="checkClock">mdi-clock-outline</v-icon>
      <v-tooltip bottom>
        <template v-slot:activator="{ on, attrs }">
          <v-btn link href="/allProject/" v-if="existUser()" icon color="white" v-bind="attrs" v-on="on">
            <v-icon>mdi-home</v-icon>
          </v-btn>
        </template>
        <span>主页</span>
      </v-tooltip>
      <v-tooltip bottom>
        <template v-slot:activator="{ on, attrs }">
          <v-btn v-if="existManager()" icon color="white" v-bind="attrs" v-on="on" @click="gotoManagerPage">
            <v-icon>mdi-link-variant</v-icon>
          </v-btn>
        </template>
        <span>管理端</span>
      </v-tooltip>

      <v-menu offset-y :close-on-content-click="true">
        <template v-slot:activator="{ on, attrs }">

          <v-chip v-if="user" outlined v-bind="attrs" v-on="on">
            <v-icon v-if="user" v-bind="attrs" v-on="on">mdi-account</v-icon>
            <v-icon v-else v-bind="attrs" v-on="on">mdi-account-remove</v-icon>
            {{user.name}}
          </v-chip>
          </template>
        <v-card v-if="user" min-width="300px"  >
          <v-img
              gradient="transparent 0%, rgba(255, 255, 255, 80%) 80%, white 100%"
              :src="getIdenticon(user.name, 300, 'user')"
              class="shades--text black--text align-end"
          >
            <v-card-title style="font-weight: bold; font-size: xx-large;">欢迎, {{ user.name }}</v-card-title>
          </v-img>

          <v-list>
            <v-list-item link to="/profile">
              <v-list-item-title class="menu_list">
                <v-icon>mdi-smart-card-outline</v-icon>
                个人信息
              </v-list-item-title>
            </v-list-item>
            <v-list-item v-if="showLabel()" @click="startTour()">
              <v-list-item-title  class="menu_list">
                <v-icon>mdi-map-search-outline</v-icon>
                打开新手指南
              </v-list-item-title>
<!--              <v-switch v-model="this.showTour" :label="'新手指南'"></v-switch>-->
            </v-list-item>
            <v-list-item link to="/topic">
              <v-list-item-title  class="menu_list">
                <v-icon>mdi-looks</v-icon>
                主题设置
              </v-list-item-title>
            </v-list-item>
            <v-divider></v-divider>
            <v-list-item link @click="logoff()">
              <v-list-item-title  class="menu_list">
                <v-icon>mdi-logout</v-icon>
                退出登录
              </v-list-item-title>
            </v-list-item>
          </v-list>
        </v-card>

        <v-card v-else min-width="200px" link to="/login">
          <v-card-title>请登录</v-card-title>
        </v-card>
      </v-menu>

      <template v-if="showLabel()" v-slot:extension>
        <v-tab link to="/allProject/">
          <v-icon middle>mdi-home-outline</v-icon>
          主页
        </v-tab>
        <v-menu
            :open-on-hover="true"
            :close-on-click="false"
            :close-on-content-click="false"
            transition="scroll-y-transition"
            v-if="user.projects.length"
            offset-y
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
                text
                v-bind="attrs"
                v-on="on"
            >
              项目
              <v-icon v-if="attrs['aria-expanded'] === 'false'" right>
                mdi-menu-down
              </v-icon>
              <v-icon v-else>
                mdi-menu-up
              </v-icon>
            </v-btn>
          </template>

          <v-card min-width="200px">
            <v-list nav rounded class="grey lighten-3">
              <v-subheader>最近项目</v-subheader>
              <v-list-item-group color="primary">
                <v-list-item
                    two-line
                    v-for="item in user.projects.slice(0, 5)"
                    :key="item.id"
                    @click="getProj(item)"
                >
                  <v-list-item-avatar>
                    <v-avatar size="40" color="indigo">
                      <!--                        <span class="white&#45;&#45;text text-h5">{{ item.projectName[0] }}</span>-->
                      <v-img :src="getIdenticon(item.projectName, 40, 'proj')"></v-img>
                    </v-avatar>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title style="font-weight: bold">{{ item.projectName }}</v-list-item-title>
                    <v-list-item-subtitle> {{
                        item.projectIntro === '' ? '暂无简介' : item.projectIntro
                      }}
                    </v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list>

            <v-divider></v-divider>

            <v-list class="grey lighten-3">
              <v-list-item-group
                  color="primary"
                  v-model="whatisclicked"
              >

                <router-link :to="{path: '/allProject/'}" custom v-slot="{ navigate }">
                  <v-list-item @click="navigate" @keypress.enter="navigate" link>
                    查看所有项目
                  </v-list-item>
                </router-link>
                <v-list-item @click="setupDialog = true; whatisclicked = null" link>
                  新建项目
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </v-card>

        </v-menu>
        <!-- <v-tabs v-model="routeSelect"> -->
        <!-- <v-tab link to="/allProject/">项目</v-tab> -->
        <!-- <v-tab link to="/allProject/allTask/">Plan</v-tab>
        <v-tab link to="/dev/">Dev</v-tab>
        <v-tab link to="/allProject/allPerson">Team</v-tab> -->
        <!--          <v-tab-->
        <!--                  v-for="project in user.projects"-->
        <!--                  :key="project.id"-->
        <!--                  link :to="'/' + ['proj', 'plan', 'dev'][drawerSelect] + '/' + project.id"-->
        <!--                  @click="selectedProj=project.id"-->
        <!--          >{{ project.name }}</v-tab>-->
        <!--          <v-tab v-for="project in user.projects" :key="project.id" link :to="'/proj/' + project.id">{{ project.name }}</v-tab>-->

        <!--          <v-tab link to="/newproj"><v-icon class="px-1">mdi-plus-circle</v-icon> new project</v-tab>-->
        <!-- </v-tabs> -->
      </template>
    </v-app-bar>

    <v-navigation-drawer
      v-model="drawer"
      app
      clipped
      permanent
      v-if="((user && proj && showLabel()) || (user && user.status === 'C') || (user && user.status === 'D'))"
    >
      <!-- <div style="background-color: aqua;width: 100%;">

      </div> -->
      <v-list v-if="user.status !== 'C' & user.status !== 'D'">
        <v-list-item id="v-step-quickStart" :style="getLinearGradient(user.topic)" two-line class="px-2">
          <v-list-item-avatar size="40" color="indigo">
            <!--            <span class="white&#45;&#45;text text-h5">{{ this.proj.projectName[0] }}</span>-->
            <v-img :src="getIdenticon(this.proj.projectName, 40, 'proj')"></v-img>
          </v-list-item-avatar>
          <v-list-item-content class="px-3">
            <!-- <v-img src="https://randomuser.me/api/portraits/women/85.jpg"></v-img> -->
            <v-list-item-title class="text-h5">
              <strong :style="'color: ' + getDarkColor(user.topic)">{{ this.proj.projectName }}</strong>
            </v-list-item-title>
            <v-list-item-subtitle>
              {{ this.proj.projectIntro === '' ? '暂无简介' : this.proj.projectIntro }}
            </v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>

        <v-list-item id="v-step-done" link to="/profile" :style="getLinearGradient(user.topic)" two-line>
          <v-list-item-content>
            <v-list-item-title class="text-h5">
              <strong :style="'color: ' + getDarkColor(user.topic)">{{ this.user.name }}</strong>
            </v-list-item-title>
            <v-list-item-subtitle>{{ this.user.email }}</v-list-item-subtitle>
          </v-list-item-content>
          <v-list-item-avatar size="40">
            <v-img :src="getIdenticon(this.user.name, 40, 'user')"></v-img>
          </v-list-item-avatar>
        </v-list-item>
      </v-list>

       <v-list subheader v-if="user.status !== 'C' & user.status !== 'D'">

          <v-subheader inset title="查看项目涉及的任务、人员和统计图">
            规划
          </v-subheader>

        <v-list-item :style="'color: ' + getDarkColor(user.topic)" link :to="'/allTask'" title="查看或管理项目涉及的任务" >
          <v-list-item-avatar>
            <v-icon :color="getDarkColor(user.topic)">mdi-ballot-outline</v-icon>
          </v-list-item-avatar>

          <v-list-item-content>
            <v-list-item-title id="v-step-createTask" >任务列表</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item :style="'color: ' + getDarkColor(user.topic)" link :to="'/allPerson'" title="查看或管理项目涉及的人员">
          <v-list-item-avatar>
            <v-icon :color="getDarkColor(user.topic)">mdi-account-outline</v-icon>
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title id="v-step-inviteMember" >人员列表</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item :style="'color: ' + getDarkColor(user.topic)" @click="gotoPic" title="查看项目统计图">
          <v-list-item-avatar>
            <v-icon :color="getDarkColor(user.topic)">mdi-align-vertical-bottom</v-icon>
          </v-list-item-avatar>

          <v-list-item-content>
            <v-list-item-title id="v-step-showInsights">图表展示</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <v-subheader inset title="项目开发涉及功能">开发</v-subheader>
        <v-list-item :style="'color: ' + getDarkColor(user.topic)" link :to="'/dev'" title="管理项目涉及的代码，提交pr并完成任务">
          <v-list-item-avatar>
            <v-icon :color="getDarkColor(user.topic)">mdi-align-vertical-center</v-icon>
          </v-list-item-avatar>

          <v-list-item-content>
            <v-list-item-title id="v-step-createRepo">代码</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
<!--        <v-list-item :style="'color: ' + getDarkColor(user.topic)" link :to="'/user/ai/diagnosis'">-->
<!--          <v-list-item-avatar>-->
<!--            <v-icon :color="getDarkColor(user.topic)">mdi-atom-variant</v-icon>-->
<!--          </v-list-item-avatar>-->

<!--          <v-list-item-content>-->
<!--            <v-list-item-title id="v-step-diagnosis">代码诊断</v-list-item-title>-->
<!--          </v-list-item-content>-->
<!--        </v-list-item>-->
<!--        <v-list-item :style="'color: ' + getDarkColor(user.topic)" link :to="'/user/ai/testdata'">-->
<!--          <v-list-item-avatar>-->
<!--            <v-icon :color="getDarkColor(user.topic)">mdi-palette-outline</v-icon>-->
<!--          </v-list-item-avatar>-->

<!--          <v-list-item-content>-->
<!--            <v-list-item-title id="v-step-testData">生成测试数据</v-list-item-title>-->
<!--          </v-list-item-content>-->
<!--        </v-list-item>-->

         <v-list-item :style="'color: ' + getDarkColor(user.topic)" link :to="'/user/ai'">
           <v-list-item-avatar>
             <v-icon :color="getDarkColor(user.topic)">mdi-palette-swatch-variant</v-icon>
           </v-list-item-avatar>

           <v-list-item-content>
             <v-list-item-title id="v-step-AI">AI 小助手</v-list-item-title>
           </v-list-item-content>
         </v-list-item>


         <v-list-item :style="'color: ' + getDarkColor(user.topic)" link :to="'/user/database'">
          <v-list-item-avatar>
            <v-icon :color="getDarkColor(user.topic)">mdi-database</v-icon>
          </v-list-item-avatar>

          <v-list-item-content>
            <v-list-item-title id="v-step-database">团队数据库</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-subheader inset id="v-step-communicate">沟通</v-subheader>
        <v-dialog
            width="1300"
            v-model="dialog"
            fullscreen
            transition="dialog-bottom-transition"
            hide-overlay
        >
          <template v-slot:activator="{on, attrs}">
            <v-list-item :style="'color: ' + getDarkColor(user.topic)">
              <v-list-item-avatar>
                <v-icon :color="getDarkColor(user.topic)">mdi-file-document-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-content>
                <v-list-item-title v-bind="attrs" v-on="on">共享文档</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </template>
          <AllFile @close="closeDocument" @open="openDocument"></AllFile>
        </v-dialog>
        <v-list-item :style="'color: ' + getDarkColor(user.topic)" link :to="'/user/chat'">
          <v-list-item-avatar>
            <v-icon :color="getDarkColor(user.topic)">mdi-account-group-outline</v-icon>
          </v-list-item-avatar>

          <v-list-item-content>
            <v-list-item-title>讨论室</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>

      <v-list subheader v-if="user.status === 'C' || user.status === 'D'" >
        <v-list-item link to="/manager/home">
          <v-list-item-icon>
            <v-icon>mdi-home-outline</v-icon>
          </v-list-item-icon>
          <v-list-item-title>主页</v-list-item-title>
        </v-list-item>
        <v-list-item link to="/manager/userMessages">
          <v-list-item-icon
          >
            <v-icon>mdi-account-multiple</v-icon>
          </v-list-item-icon
          >
          <v-list-item-title>用户信息</v-list-item-title>
        </v-list-item>
        <v-list-item link to="/manager/loginMessages">
          <v-list-item-icon>
            <v-icon>mdi-history</v-icon>
          </v-list-item-icon>
          <v-list-item-title>用户登录信息</v-list-item-title>
        </v-list-item>
        <v-list-item link to="/manager/projectMessages">
          <v-list-item-icon
          >
            <v-icon>mdi-book-edit-outline</v-icon>
          </v-list-item-icon
          >
          <v-list-item-title>项目信息</v-list-item-title>
        </v-list-item>
      </v-list>

    </v-navigation-drawer>
    <el-dialog title="创建项目"
               :visible.sync="setupDialog"
               width="50%"
               :before-close="handleClose">
      <el-form :label-position="labelPosition" label-width="80px" :model="form" ref="form">
        <el-form-item label="项目名称">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="活动概述">
          <el-input type="textarea" v-model="form.intro" :autosize="{ minRows: 5, maxRows: 10}"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cancelSetupProject">取 消</el-button>
        <el-button type="primary" @click="setupProject">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog
        title="我的提醒"
        :visible.sync="clockDialog"
        width="50%"
    >
      <v-simple-table>
        <thead>
        <tr>
          <th class="text-left">
            任务
          </th>
          <th class="text-left">
            时间
          </th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="notice in noticeList" :key="notice.noticeId" @mouseenter="arr[notice.taskId] = true"
            @mouseleave="arr[notice.taskId] = false">
          <td>{{ getTaskName(notice.taskId) }}</td>
          <td>{{ new Date(notice.deadline).toLocaleString() }}</td>
          <td>
            <v-icon @click="handleDeleteNotice(notice.noticeId)">mdi-delete</v-icon>
          </td>
        </tr>
        </tbody>
      </v-simple-table>
    </el-dialog>

    <el-dialog
        title="我的通知"
        :visible.sync="noteDialog"
        width="80%"
    >
      <v-btn
          style="top: 35%; right: 2%; width: 10%; position: absolute;"
          depressed
          :color="getTopicColor(user.topic)"
          @click="handleReadAllNotice()"
          v-if="noticeList2.length!==0"
      >一键已读</v-btn>
      <v-simple-table >
        <thead>

        <tr>
<!--          <th class="text-left">-->
<!--            状态-->
<!--          </th>-->

          <th class="text-left">
            时间
          </th>
          <th >

          </th>
          <th class="text-left">
            详情
          </th>
        </tr>
        </thead>
        <tbody>
        <tr    :style="!notice.seen ? 'background-color:#f0f0f0' : 'background-color:white'"  v-for="notice in noticeList2" :key="notice.noticeId" @mouseenter="arr[notice.taskId] = true"
            @mouseleave="arr[notice.taskId] = false">
<!--          <td>-->

<!--            <v-chip :color="getColor(notice.seen)" dark>-->
<!--              {{notice.seen ? "已读":"未读"}}-->
<!--            </v-chip>-->
<!--          </td>-->

          <td>
            {{ new Date(notice.deadline).toLocaleString() }}
          </td>

          <td >

          </td>
          <td v-if="!notice.seen"
              class="clickable-row"
              @click="handleReadNotice(notice.noticeId)"
          >

            <strong v-if="!notice.seen">{{notice.content}}</strong>
            <div v-else>{{notice.content}}</div>
            <v-badge dot class="float-md-end" v-if="!notice.seen" :color="getTopicColor()" inline></v-badge>
          </td>
          <td v-else
          >

            <strong v-if="!notice.seen">{{notice.content}}</strong>
            <div v-else>{{notice.content}}</div>
            <v-badge dot class="float-md-end" v-if="!notice.seen" :color="getTopicColor()" inline></v-badge>
          </td>
<!--          <td style="width: 10%">-->
<!--            <v-btn :color="getTopicColor(user.topic)"-->
<!--                   class="white&#45;&#45;text"-->
<!--                   @click="handleDeleteNotice(notice.noticeId)">-->
<!--              删除-->
<!--            </v-btn>-->
<!--          </td>-->
<!--          <td style="width: 10%">-->
<!--            <v-btn :color="getTopicColor(user.topic)"-->
<!--                   class="white&#45;&#45;text"-->
<!--                   @click="handleReadNotice(notice.noticeId)">-->
<!--              已读-->
<!--            </v-btn>-->
<!--          </td>-->

        </tr>
        </tbody>
      </v-simple-table>
    </el-dialog>

    <!--<el-dialog-->
    <!--    title="通知详情"-->
    <!--    :visible.sync="noteDetailDialog"-->
    <!--    width="80%"-->
    <!--&gt;-->
    <!--  <v-simple-table>-->
    <!--    <thead>-->
    <!--    <tr>-->
    <!--      <th class="text-left">-->
    <!--        所属任务-->
    <!--      </th>-->
    <!--      <th class="text-left">-->
    <!--        时间-->
    <!--      </th>-->
    <!--    </tr>-->
    <!--    </thead>-->
    <!--    <tbody>-->
    <!--    <tr v-for="notice in noticeList2" :key="notice.noticeId" @mouseenter="arr[notice.taskId] = true"-->
    <!--        @mouseleave="arr[notice.taskId] = false">-->
    <!--      <td>{{ getTaskName(notice.taskId) }}</td>-->
    <!--      <td>{{ new Date(notice.deadline).toLocaleString() }}</td>-->
    <!--      <td>-->
    <!--        <v-btn :color="getTopicColor(user.topic)"-->
    <!--               class="white&#45;&#45;text"-->
    <!--               @click="handleDeleteNotice(notice)">-->
    <!--          详情-->
    <!--        </v-btn>-->
    <!--      </td>-->
    <!--    </tr>-->
    <!--    </tbody>-->
    <!--  </v-simple-table>-->
    <!--</el-dialog>-->

    <v-main>
      <router-view v-if="showRouterView"/>
    </v-main>


    <el-dialog
        title="提醒列表"
        width="30%"
        :before-close="handleClose">
      <span>这是一段信息</span>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
    </el-dialog>

  </v-app>
    <v-tour v-if="showTour" name="myTour" :options="myOptions" :steps="steps" :callbacks="myCallbacks"></v-tour>
  </div>
</template>

<script>
import Cookies from "js-cookie"
import {computed} from "vue"
import {
  newProject, showTaskList, watchAllProject, getEmail, showNoticeList, removeNotice,
  userReleaseDocLock, readNotice, getDiffString
} from "@/api/user"
import axios from "axios"
import AllTask from "@/views/user/projectPlanning/allTask.vue"
import AllFile from "@/views/user/document/allFile.vue"
import getIdenticon from "@/utils/identicon";
import topicSetting from "@/utils/topic-setting";
// import allTask from "@/views/user/projectPlanning/allTask"

let user = Cookies.get("user");
let proj;
console.log("user: :" + user);

if (user === undefined) { // 用户未登录
  console.log("not logged in");
  if (window.location.pathname !== "/login") {
    window.location.href = '/login'
  }
} else { // 用户已登录
  let userpath = (window.location.pathname !== "/workDetail" && window.location.pathname !== "/allFile" &&
      window.location.pathname !== "/allPerson" && window.location.pathname !== "/allTask" &&
      window.location.pathname !== "/picture" && !window.location.pathname.startsWith("/plan") &&
      !window.location.pathname.startsWith("/allProject") && window.location.pathname !== "/home" &&
      !window.location.pathname.startsWith("/dev") && window.location.pathname !== "/profile" &&
      window.location.pathname !== "topic" &&
      !window.location.pathname.startsWith("/user")) // 合法的普通用户路径
  let managerpath = (window.location.pathname !== "/profile" &&
      window.location.pathname !== "topic" && !window.location.pathname.startsWith("/manager")) // 合法的纯管理员路径（位于管理端）
  user = JSON.parse(user)
  proj = undefined;
  if (user !== undefined) {
    if (user.status !== 'C' && user.status !== 'D') { // 普通用户
      proj = Cookies.get("proj");
      console.log(proj)
      if (proj !== undefined) {
        if (userpath) {
          window.location.href = "/allTask"
        }
        // if (window.location.pathname !== "/allTask") {
        //   window.location.href = "/allTask"
        // }
      } else {
        if (userpath) {
          window.location.href = "/allProject"
        }
        // if (window.location.pathname !== "/allProject") {
        //   window.location.href = "/allProject"
        // }
      }
    } else { // 管理员
      if (managerpath) {
        window.location.href = "/manager/home"
      }
      // if (window.location.pathname !== "/manager/home") {
      //   window.location.href = "/manager/home"
      // }
    }
  }
}

export default {
  name: "App",
  // created() {
  //     this.$watch(() => this.drawerIndex, (to, from) => {
  //         console.log('drawerIndex change!')
  //         this.showRouterView = false;
  //         this.$nextTick(() => (this.showRouterView = true));
  //     })
  // }
  created() {
    console.log("created");
    window.addEventListener('scroll', () => {
      this.scrollUp = this.isScrollTop()
    })
    this.updateUserProj();
    this.updateTopic();
    let doc = Cookies.get("doc");
    console.log("cookies");
    console.log(doc);
    if (doc !== undefined && doc !== 'undefined') {
      doc = JSON.parse(doc);
      console.log(proj);
      userReleaseDocLock({userId: user.id, projectId: JSON.parse(proj).projectId, docId: doc.id}).then(
          res => {
            console.log("userReleaseDocLock");
            console.log(res);
          }
      )
    }
    Notification.requestPermission()

    let proj = Cookies.get("proj");
    if (proj !== undefined && proj !== 'undefined') {
      proj = JSON.parse(proj)
      this.proj = proj;
    }

    this.getTaskList()

    //设置tour
    this.showTour = user.first_login
    this.myCallbacks = {
      onFinish: this.onTourFinish,
    }

    console.log('setting interval...')
    setInterval(() => {
      this.updateNoticeList();
    }, 5000)
  },
  components: {
    AllTask,
    AllFile,
  },
  watch: {
    selectedProj(n, o) {
      console.log("selectedProj change! from " + o + " to " + n);
      this.showRouterView = false;
      this.$nextTick(() => (this.showRouterView = true));
    },
  },
  mounted: function () {
    if (this.showTour) {
      this.$tours['myTour'].start()
      console.log("tour start")
    }
  },
  data: () => {
    return {
      // from: -1, // -1普通用户，0从用户页面跳转，1从项目页面跳转，存入Cookie
      topic: "https://fastly.picsum.photos/id/53/1280/1280.jpg?hmac=QP5opo-oENp5iFwsSiWH8azQuR0w0bwps6MT6yvhKwA",
      labelPosition: "left",
      drawer: true,
      mini: true,
      showRouterView: true,
      user: user,
      proj: null,
      drawerIndex: null,
      routeSelect: null,
      selectedProj: null,
      setupDialog: false,
      selectedItem: null,
      dialog: false,
      clockDialog: false,
      noteDialog:false,
      editClockDialog: false,
      form: {
        name: "",
        intro: "",
        id: "",
      },
      projectData: [],
      tasks: [],
      scrollUp: true,
      clockList: [],
      noticeList: [],  //存alarm
      noticeList2: [], //存通知
      existUnreadNote: false,
      arr: [],
      whatisclicked: null,
      //vue-tour
      showTour: true,
      myCallBacks:null,
      myOptions: {
        useKeyboardNavigation: true,
        labels: {
          buttonSkip: '跳过',
          buttonPrevious: '上一步',
          buttonNext: '下一步',
          buttonStop: '完成'
        }
      },
      steps:[
      {
        target: '#v-step-quickStart',
        header: {
          title:'Hi！'
        },
        content: '欢迎使用 JiHub <br> 我们将通过简单指引，帮助你快速熟悉核心功能',
        params: {
          placement: 'right',
          class: 'no-arrow'
        }
      },
      {
        target: '#v-step-inviteMember',
        header: {
          title:'团队成员'
        },
        content: '邀请成员加入项目，一起开启高效研发，在这里，可以查看所有人员列表和其工作量',
        params: {
          placement: 'right',
          class: 'no-arrow'
        }
      },
      {
        target: '#v-step-createRepo',
        header: {
          title:'代码管理'
        },
        content: '一键创建GitHub仓库，高效管理项目代码，无需切换到GitHub，即可查看所有分支和提交记录，并创建和处理PR请求',
        params: {
          placement: 'right',
          classes: 'no-arrow'
        }
      },
      {
        target: '#v-step-createTask',
        header: {
          title:'任务管理'
        },
        content: '创建和查看任务，轻松管理项目进度。',
        params: {
          placement: 'right',
          classes: 'no-arrow'
        }
      },
      {
        target: '#v-step-showInsights',
        header: {
          title:'查看图表'
        },
        content: '显示燃尽图、任务进度和协作关系图，全面了解项目进展和团队协作情况。',
        params: {
          placement: 'right',
          classes: 'no-arrow'
        }
      },
      {
        target: '#v-step-AI',
        header: {
          title:'AI代码助手'
        },
        content: '提供AI代码诊断和生成测试数据，提升工作效率并减少错误。',
        params: {
          placement: 'right',
          classes: 'no-arrow'
        }
      },
      {
        target: '#v-step-database',
        header: {
          title:'团队数据库'
        },
        content: '提供团队数据库，实现数据的高效利用和共享。',
        params: {
          placement: 'right',
          classes: 'no-arrow'
        }
      },
      {
        target: '#v-step-communicate',
        header: {
          title:'协作空间'
        },
        content: '提供共享文档和在线聊天室，促进团队协作，帮助团队成员高效共享信息和协作解决问题。',
        params: {
          placement: 'right',
          classes: 'no-arrow'
        }
      },
      {
        target: '#v-step-remind',
        header: {
          title:'通知与提醒'
        },
        content: '点击 <strong> 小铃铛 </strong> 聚合所有与自己相关的工作任务消息，及时了解最新工作任务；<br> 点击 <strong> 小时钟 </strong> 聚合所有相关任务的提醒，及时完成工作任务',
        params: {
          placement: 'bottom',
          classes: 'no-arrow'
        }
      },
      {
        target: '#v-step-done',
        content: ' 开启JiHub高效研发之旅！<br> <br>',
        params: {
          placement: 'right',
        }
      },
      ],
    };
  },
  beforeUpdate() {
    // this.drawer = user && proj && showLabel();
    this.showLabel();
    let proj = Cookies.get("proj");
    if (proj !== undefined) {
      proj = JSON.parse(proj)
      this.proj = proj;
    }
  },
  provide() {
    return {
      user: computed(() => this.user),
      proj: computed(() => this.proj),
      //from: computed(() => this.from),
      selectedProj: computed(() => this.proj),
      changeSelectedProj: this.changeSelectedProj,
      updateUserProj: this.updateUserProj,
      updateUser: this.updateUser,
      updateTopic: this.updateTopic,
      getProj: this.getProj,
      //setFrom: this.setFrom,
      // reload:this.reload
    };
  },
  methods: {
    getIdenticon,
    // setFrom(from) {
    //   console.log("setFrom")
    //   this.from = from
    //   console.log(this.from)
    // },
    updateTopic() {
      if (this.user.topic === 'A') { // 红色
        this.topic = "https://fastly.picsum.photos/id/859/1919/1919.jpg?hmac=24AoHo7Jc5TRRRaJfWO0B4z2wW5Jl14r56rVKeMfpZI"
      } else if (this.user.topic === 'B') { // 橙色
        this.topic = "https://fastly.picsum.photos/id/360/1925/1280.jpg?hmac=vX9T-qgnqfOUqLnxlsxCZfkKn_wi-9vtPKdbq7H6EgU"
      } else if (this.user.topic === 'C') { // 绿色
        this.topic = "https://fastly.picsum.photos/id/509/4608/3456.jpg?hmac=ZvNo59d3NP2XjQE75AaROWBTpW2BAzZgWb0-Jso0l9Y"
      } else if (this.user.topic === 'D') { // 蓝色
        this.topic = "https://fastly.picsum.photos/id/53/1280/1280.jpg?hmac=QP5opo-oENp5iFwsSiWH8azQuR0w0bwps6MT6yvhKwA"
      } else { // 紫色
        this.topic = "https://fastly.picsum.photos/id/723/5000/3333.jpg?hmac=Lq_OA9h0W8RDhE3xt44qzGv5PtQVLSnVCjoizkIiQNs"
      }
    },
    updateUser() {
      var userCookie = Cookies.get("user")
      if (userCookie !== undefined) {
        this.user = JSON.parse(userCookie)
      }
    },
    updateNoticeList() {
      console.log("updating NoticeList...")
      showNoticeList({
        projectId: this.proj.projectId,
        // user_id:this.user.id,
      }).then(
          res => {
            this.existUnreadNote = false;
            res['data']['data'].forEach(item => {
              if(item.user_id === this.user.id) {
                if (item.type === "A" && !item.seen)
                  this.existUnreadNote = true;

                // 如果两个时间小于5秒，就弹出提醒
                if (Math.abs(new Date(item.deadline) - new Date()) < 5000) {
                  console.log(Math.abs(new Date(item.deadline) - new Date()))
                  this.$message({
                    showClose: true,
                    message: "有到期的截止日期！",
                    type: "warning",
                    duration: 0,
                  });
                  if ("Notification" in window) {
                    Notification.requestPermission().then(function (permission) {
                      if (permission === "granted") {
                        let notification = new Notification("有到期的截止日期！", {
                          body: "请及时处理！"
                        });
                      }
                    });
                  }
                }
              }
            })
            console.log(this.noticeList);
          }
      )
    },
    checkClock() {
      this.clockDialog = true;
      showNoticeList({
        projectId: this.proj.projectId,
        // user_id:this.user.id,
      }).then(
          res => {
            var allNotice = res['data']['data'];
            var tmpNoticeList = [];
            for (let i = 0; i < allNotice.length; i++)
              if(allNotice[i].type === "B" && allNotice[i].user_id === this.user.id)
                tmpNoticeList.push(allNotice[i]);
            this.noticeList = tmpNoticeList;
            console.log(this.noticeList);
          }
      )
    },
    closeDocument() {
      this.dialog = false;
    },
    openDocument() {
      this.dialog = true;
    },
    getEmail(id) {
      getEmail({id: id}).then(
          res => {
            console.log("getEmail");
            console.log(res);
            console.log(id);
            return res['data']['data'];
          }
      )
    },
    get_project() {
      Cookies.remove("proj");
      console.log("get_project");
      watchAllProject({userId: this.user.id}).then(
          res => {
            this.projectData = res['data']['data'];
            console.log(this.projectData);
          }
      )
    },
    getTaskList() {
      console.log(this.user.id);
      console.log(this.selectedProj);
      showTaskList({userId: this.user.id, projectId: this.proj.projectId}).then(
          res => {
            console.log("getTaskList");
            console.log(res);
            this.tasks = res['data']['data'];
            console.log(this.tasks);
          }
      );
    },
    gotoPic() {
      showTaskList({userId: this.user.id, projectId: this.proj.projectId}).then(
          res => {
            console.log("getTaskList");
            console.log(res);
            this.tasks = res['data']['data'];
            console.log(this.tasks);
            let projectItem = [];
            let projectItemStart = [];
            let projectItemEnd = [];
            let workloads = [];
            let expectedDates = [];
            let actualDates = [];
            let projectState = [];
            console.log(this.tasks);
            for (let i = 0; i < this.tasks.length; i++) {
              for (let j = 0; j < this.tasks[i].subTaskList.length; j++) {
                projectItem.push(this.tasks[i].subTaskList[j].subTaskName);
                projectItemStart.push(this.tasks[i].subTaskList[j].start_time.slice(0, 10));
                projectItemEnd.push(this.tasks[i].subTaskList[j].deadline.slice(0, 10));
                workloads.push(parseInt(this.tasks[i].subTaskList[j].contribute));
                expectedDates.push(this.tasks[i].subTaskList[j].deadline.slice(0, 10));
                actualDates.push(this.tasks[i].subTaskList[j].complete_time.slice(0, 10));
                projectState.push(this.tasks[i].subTaskList[j].status);
              }
            }
            console.log(this.tasks);
            console.log(projectItem);
            console.log(projectItemStart);
            console.log(projectItemEnd);
            if (projectItem.length == 0) {
              this.$message({
                type: 'info',
                message: '您还没有任务'
              })
              return;
            }
            this.$router.push({
              path: '/picture'
              , query: {
                projectItem: projectItem, projectItemStart: projectItemStart, projectItemEnd: projectItemEnd,
                workloads: workloads, expectedDates: expectedDates, actualDates: actualDates, projectState: projectState
              }
            });
          });
    },
    getProj(item) {
      console.log("getProj");
      console.log(item);
      console.log(JSON.stringify(item));
      Cookies.set('proj', JSON.stringify(item));
      window.location.href = '/allTask';
      this.proj = Cookies.get(proj);
    },
    existManager() {
      return Cookies.get("manager");
    },
    existUser() {
      // console.log("111111")
      // console.log(Cookies.get("user"))
      if (Cookies.get("user") === undefined) {
        return false;
      }
      let tmp = JSON.parse(Cookies.get("user"))
      return tmp.status === 'A'
    },
    //gotoHomePage() {
    //window.location.href = '/allProject'
    //},
    gotoManagerPage() {
      console.log("122343243242432525")
      console.log(Cookies.get("from"))
      Cookies.set("user", Cookies.get("manager"))
      Cookies.remove("manager");
      Cookies.remove("proj");
      console.log("user")
      console.log(Cookies.get("user"))
      console.log("manager")
      console.log(Cookies.get("manager"))
      let from = Cookies.get("from")
      if (from === "0") {
        Cookies.remove("from");
        window.location.href = '/manager/userMessages'
      }
      if (from === "1") {
        Cookies.remove("from");
        window.location.href = '/manager/projectMessages'
      }
    },
    showLabel() {
      if (this.user === null || this.user === undefined || this.user.status === 'C' || this.user.status === 'D') {
        return false;
      }
      console.log("showLabel");
      console.log(this.proj);
      console.log(this.user);
      console.log(this.$route.path);

      console.log(this.user !== null && this.proj !== undefined &&
          !window.location.pathname.startsWith("/manager") &&
          this.$route.path !== "/allProject/" &&
          !window.location.pathname.startsWith("/login") &&
          !window.location.pathname.startsWith("/register"));

      return (
          this.user !== null && this.user !== undefined &&
          this.proj !== null && this.proj !== undefined &&
          !window.location.pathname.startsWith("/manager") &&
          this.$route.path !== "/allProject/" &&
          !window.location.pathname.startsWith("/login") &&
          !window.location.pathname.startsWith("/register") &&
          !window.location.pathname.startsWith("/profile") &&
          !window.location.pathname.startsWith("/topic")
      );
    },
    // getSelectedProj() {
    //     let pid = this.$route.params.projid;
    //     if (pid === undefined) return null;
    //     else return pid;
    // },
    // getDrawerSelection() {
    //   let drawerSelection = this.$route.
    // },
    logoff() {
      Cookies.remove("user");
      Cookies.remove("manager");
      Cookies.remove("proj");
      Cookies.remove("from")
      window.location.href = "/login";
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
          .then(() => {
            done();
          })
          .catch(() => {
          });
      this.whatisclicked = null
    },
    cancelSetupProject() {
      this.setupDialog = false;
      this.form = {
        name: '',
        intro: ''
      }
      this.whatisclicked = null
    },
    setupProject() {
      // console.log(this.search);
      // console.log("submit");
      this.whatisclicked = null
      watchAllProject({userId: this.user.id}).then(
          res => {
            this.projectData = res['data']['data'];
            console.log(this.projectData);
          }
      )
      if (this.form.name.trim() === "") {
        this.$message({
          type: 'error',
          message: '项目名不能为空！'
        });
        return;
      }
      for (let i = 0; i < this.projectData.length; i++) {
        if (this.form.name === this.projectData[i].projectName) {
          this.$message({
            type: 'error',
            message: '已存在同名项目'
          });
          return;
        }
      }
      this.setupDialog = false;
      newProject({projectName: this.form.name, projectIntro: this.form.intro, userId: this.user.id}).then(
          res => {
            console.log(this.user.id);
            console.log(res);
            this.updateUserProj();
            this.get_project();
          }
      );
      this.form = {
        name: '',
        intro: ''
      }
    },
    handleDeleteNotice(noticeId) {
      this.$confirm("确认删除提醒？")
          .then(() => {
            removeNotice({noticeId: noticeId}).then(
                res => {
                  // this.updateNoticeList();
                  showNoticeList({
                    projectId: this.proj.projectId,
                    // user_id:this.user.id,
                  }).then(
                      res => {
                        var allNotice = res['data']['data'];
                        var tmpNoticeList = [];
                        for (let i = 0; i < allNotice.length; i++)
                          if(allNotice[i].user_id === this.user.id && allNotice[i].type === "A")
                            tmpNoticeList.push(allNotice[i]);
                        this.noticeList2 = tmpNoticeList;
                      }
                  )
                }
            )
          })
          .catch(() => {
          });
    },

    handleReadAllNotice() {
      const hasUnreadNotice = this.noticeList2.some(notice => !notice.seen);

      this.noticeList2.forEach(notice => {
        readNotice({id: notice.noticeId}).then(
        )
        notice.seen = true;
      });
    },


    handleReadNotice(noticeId) {
      // this.updateNoticeList()
      readNotice({id: noticeId}).then(
          res => {
            // this.updateNoticeList();
            showNoticeList({
              projectId: this.proj.projectId,
              // user_id:this.user.id,
            }).then(
                res => {
                  var allNotice = res['data']['data'];
                  var tmpNoticeList = [];
                  this.existUnreadNote = false;
                  for (let i = 0; i < allNotice.length; i++)
                    if(allNotice[i].type === "A" && allNotice[i].user_id === this.user.id)
                    {
                      tmpNoticeList.push(allNotice[i]);
                      if(!allNotice[i].seen){
                        this.existUnreadNote = true;
                      }
                    }

                  this.noticeList2 = tmpNoticeList;
                }
            )
          }
      )
    },

    handleNoticeDetail(notice){
      getDiffString({
        user_id: JSON.parse(Cookies.get("user")).id,
        remote_path: notice.remote_path,
        project_id: this.proj.projectId,
        ghpr_id: notice.ghpr_id,
      }).then(
          res =>{
            console.log(res);
            this.$message({
              type: 'success',
              message: '正在进入！'
            });
            localStorage.setItem("diffString", res['data']['diff_output']);
            localStorage.setItem("comment", res['data']['comment']);
            localStorage.setItem("title", res['data']['title']);
            localStorage.setItem("description",res['data']['description']);
            // Cookies.set("diffString",JSON.stringify(res['data']['diff_output']));
            console.log("cbycby3" + res['data']['diff_output']);
            Cookies.set("PrToAudit_ReadOnly", true);
            this.noteDialog = false;
            this.$router.push({ name: 'audit' });
          }
      )


    },

    changeSelectedProj(proj) {
      this.selectedProj = proj;
    },
    updateUserProj() {
      if (this.user !== undefined) {
        axios
            .post("/api/plan/watchAllProject", {
              userId: this.user.id,
            })
            .then((res) => {
              console.log("updateProj");
              console.log(res);
              if (res.data.errcode === 0) {
                this.user.projects = res['data']['data'];
                Cookies.set("user", JSON.stringify(this.user));
              } else {
                alert(
                    "updateUserProj failure! with non 0 errcode " + res.data.errcode
                );
              }
            })
            .catch((err) => {
              alert("updateUserProj failure! with error " + err);
            });
      }
    },
    isScrollTop() {
      if (document.body.scrollHeight > (window.innerHeight || document.documentElement.clientHeight)) {
        let scrollTop = window.scrollY || document.documentElement.scrollTop || document.body.scrollTop || 0;
        return scrollTop === 0
      }
    },
    getTaskName(taskId) {
      // this.getTaskList();
      console.log("getTaskName is called");
      console.log(this.tasks);
      for (let i = 0; i < this.tasks.length; i++) {
        for (let j = 0; j < this.tasks[i].subTaskList.length; j++) {
          if (this.tasks[i].subTaskList[j].subTaskId === taskId) {
            return this.tasks[i].subTaskList[j].subTaskName;
          }
        }
      }
    },

    checkNote(){
      this.noteDialog = true;
      showNoticeList({
        projectId: this.proj.projectId,
        // user_id:this.user.id,
      }).then(
          res => {
            var allNotice = res['data']['data'];
            var tmpNoticeList = [];
            for (let i = 0; i < allNotice.length; i++)
              if(allNotice[i].type === "A" && allNotice[i].user_id === this.user.id)
              {
                if(!allNotice[i].seen)
                  this.existUnreadNote = true;
                tmpNoticeList.push(allNotice[i]);
              }

            this.noticeList2 = tmpNoticeList;
            return true;
          }
      )

    },
    getColor(state){
      return state ? "green" : "red";
    },

    onTourFinish() {
      console.log("tour finished")
      if (this.user.first_login) {
        this.user.first_login = false
        Cookies.set('user', JSON.stringify(this.user))
        console.log("set user's first_login to false")
      }
      this.showTour = false
    },

    startTour() {
      this.showTour = true
      this.$nextTick(() => {
        if (this.$tours && this.$tours['myTour']) {
          this.$tours['myTour'].start();
        }
      });
    },

    getTopicColor: topicSetting.getColor,
    getDarkColor: topicSetting.getDarkColor,
    getLinearGradient: topicSetting.getLinearGradient,
    getLinearGradientLiteral: topicSetting.getLinearGradientLiteral,
  },
};
</script>

<style>
#username {
  font-size: 1.2em;
  line-height: 1.2em;
}

html,
body,
.main_page{
  height: 100%;
  margin: 0;
  padding: 0;
}

.menu_list {
  display: flex;
  align-items: center;

}
.menu_list .v-icon {
  margin-right: 12px;
}
.description {
  word-break:normal;
  width:auto;
  display:block;
  white-space:pre-wrap;
  word-wrap : break-word ;
  overflow: hidden ;
  line-height: 1.5rem;
}

.no-arrow .v-step__arrow {
  display: none;
}

.start-class {
  font-size: small;
}

.v-step__content {
  width: 95%;
  text-align: left;
  font-size: small;
}

.v-step__button {
  height: 180px ;
}

.v-step {
  width: 220px;


}

.clickable-row:hover {
  background-color: #f0f0f0;
  cursor: pointer;
}





</style>