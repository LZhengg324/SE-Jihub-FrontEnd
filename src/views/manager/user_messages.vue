<template>
  <v-container>
    <v-card>
      <v-card-title>
        <v-container fluid>
          <v-row>
            <v-col cols="4">
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <v-select
                      v-model="filterStatus"
                      :items="filterStatusList"
                      item-value="value"
                      item-text="label"
                      label="状态"
                      single-line
                      hide-details
                      v-on="on"
                  ></v-select>
                </template>
                <span>过滤显示状态</span>
              </v-tooltip>
            </v-col>
              <v-col cols="8">
              <v-text-field
                  v-model="search"
                  append-icon="mdi-magnify"
                  label="搜索"
                  single-line
                  hide-details
                  title="状态"
              ></v-text-field>
            </v-col>
          </v-row>
        </v-container>

      </v-card-title>


      <v-data-table
          :headers="headers"
          :items="filteredUserMessages"
          :search="search"


      >
        <template #item.status="{item}">
          <v-chip :color="getColor(item.status)" dark @click="openChangeUserStatusDialog(item)">
            {{
              transform(item.status) }}
<!--            <v-text v-if="item.status==='A'"> 正常 </v-text>-->
<!--            <v-text v-if="item.status==='B'"> 禁用 </v-text>-->
          </v-chip>
        </template>
        <template #item.resetPassword="{item}">
          <v-btn class="ml-1" small outlined @click="openResetPasswordDialog(item)">重置用户密码</v-btn>
        </template>
        <template #item.registerTime="{item}">
          {{ pro(item.registerTime) }}
        </template>
<!--        <template #item.changeStatus="{item}">-->
<!--          <v-btn class="ml-1" small outlined @click="openChangeUserStatusDialog(item)">修改用户状态</v-btn>-->
<!--        </template>-->
        <!--
        <template #item.userProfile="{item}">
          <v-btn class="ml-1" small outlined @click="openUserProfileDialog(item)">用户个人信息</v-btn>
        </template>
        -->
        <template #item.userProject="{item}">
<!--          <v-btn class="ml-1" small outlined @click="gotoUserPage(item)">用户所在项目</v-btn>-->
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn icon color="blue" v-bind="attrs" v-on="on" @click="gotoUserPage(item)">
                <v-icon>mdi-link-variant</v-icon>
              </v-btn>
            </template>
            <span>以该用户视角进入用户端</span>
          </v-tooltip>
        </template>
      </v-data-table>
    </v-card>
    <v-dialog v-model="showResetPassword" width="300">
      <template>
        <v-container class="pa-0">
          <v-card>
            <v-card-title class="headline font-weight text-left"> 重置用户{{ userResetPasswordDialogMessage.name }}的密码 </v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="red" text @click="closeResetPasswordDialog">取消</v-btn>
              <v-btn color="blue" text @click="resetPassword">确认</v-btn>
            </v-card-actions>
          </v-card>
        </v-container>
      </template>
    </v-dialog>
    <v-dialog v-model="showChangeUserStatus" width="300">
      <template>
        <v-container class="pa-0">
          <v-card>
            <v-card-title class="headline font-weight text-left"> 修改用户{{userStatusDialogMessage.name}}的状态 </v-card-title>
            <v-divider></v-divider>
            <v-card-text>
              <v-radio-group v-model="selectedStatus">
                <v-radio v-for="i in statusList" :key="i.value" :label="i.label" :value="i.value"></v-radio>
              </v-radio-group>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="red" text @click="closeChangeUserStatusDialog">取消</v-btn>
              <v-btn color="blue" text @click="changeStatus">确认修改</v-btn>
            </v-card-actions>
          </v-card>
        </v-container>
      </template>
    </v-dialog>
    <v-dialog v-model="showUserProfile" width="500">
      <template>
        <v-container class="pa-0">
          <v-card>
            <v-card-title class="headline font-weight text-left"> 用户{{userProfileDialogMessage.name}}的个人信息 </v-card-title>
            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="red" text @click="closeUserProfileDialog">关闭</v-btn>
              <v-btn color="blue" text @click="saveProfile">确认修改</v-btn>
            </v-card-actions>
          </v-card>
        </v-container>
      </template>
    </v-dialog>
  </v-container>
</template>

<script>
import axios from "axios";
import Cookies from "js-cookie";
import util from "@/views/util";

let user = Cookies.get("user")
export default {
  inject: {
    user: { default: null },
    //setFrom: { default: null}
  },
  data () {
    return {
      msg: null,
      search: '',
      filterStatus: 'ALL',
      headers: [
        {
          text: '用户名',
          align: 'start',
          // sortable: false,
          value: 'name',
        },
        { text: '邮箱', value: 'email' },
        { text: '注册时间', value: 'registerTime' },
        { text: '状态', value: 'status' },
        {
          text: '',
          sortable: false,
          value: 'resetPassword'
        },
        // {
        //   text: '',
        //   sortable: false,
        //   value: 'changeStatus'
        // },
        {
          text: '',
          sortable: false,
          value: 'userProfile'
        },
        {
          text: '',
          sortable: false,
          value: 'userProject'
        },
      ],
      userMessages: [
         {
           name: 'user1',
           id: '1',
           email: '123@qq.com',
           registerTime: '2023.1.1',
           status: 'A',
         },
         {
           name: 'faskfl',
           id: '2',
           email: 'gers@qq.com',
           registerTime: '2023.2.1',
           status: 'B',
         },
         {
           name: 'saga',
           id: '3',
           email: '53@qq.com',
           registerTime: '2023.3.1',
           status: 'A',
         },
      ],
      // 重置用户密码dialog相关信息
      showResetPassword: false,
      userResetPasswordDialogMessage: '',
      // 用户状态dialog相关信息
      showChangeUserStatus: false,
      userStatusDialogMessage: '',
      selectedStatus: '',
      statusList: [
        {
          label: '正常',
          value: 'A'
        },
        {
          label: '禁用',
          value: 'B'
        },
        {
          label: '项目管理员',
          value: 'D'
        },
      ],
      filterStatusList: [
        {
          label: '全部',
          value: 'ALL',
        },
        {
          label: '正常',
          value: 'A'
        },
        {
          label: '禁用',
          value: 'B'
        },
        {
          label: '项目管理员',
          value: 'D'
        },
      ],

      // 用户个人信息dialog相关信息
      showUserProfile: false,
      userProfileDialogMessage: '',
      // 跳转到用户端的相关信息
      gotoUserProjectPageMessage: '',
    }
  },
  created() {
    this.showUserMessages()
  },
  computed: {
    filteredUserMessages() {
      // 先根据搜索关键词过滤
      let filtered = this.userMessages.filter(message => {
        return message.name.toLowerCase().includes(this.search.toLowerCase());
      });

      // 再根据状态过滤
      if (this.filterStatus !== 'ALL') {
        filtered = filtered.filter(message => {
          return message.status === this.filterStatus;
        });
      }

      return filtered;
    }
  },
  // TODO：传给后端管理员id，如果报错，不显示信息而显示弹窗
  methods: {
    pro(registerTime) {
      return util.processTime(registerTime);
    },
    // 显示用户信息
    showUserMessages() {
      console.log(this.user.id)
      axios.post("/api/management/showUsers", {managerId: this.user.id})
          .then((response) => {
            console.log(response)
            if (response.data.errcode === 1) {
              this.$message({
                type: 'error',
                message: "您没有权限"
              });
            } else if (this.user.status === 'D') {
              //过滤掉小管理员
              this.userMessages = response.data.users.filter(user => user.status !== 'D')
            } else {
              this.userMessages = response.data.users

            }
          })
          .catch((err) => {
            console.error(err);
            this.userMessages = null
          })
    },
    // 打开重置用户密码窗口
    openResetPasswordDialog(item) {
      if (this.user.status === 'D') {
        this.$message({
          type: 'error',
          message: "您没有权限"
        });
        return
      }
      this.userResetPasswordDialogMessage = item
      console.log("open reset password dialog")
      console.log(this.userResetPasswordDialogMessage)
      this.showResetPassword = true
    },
    // 关闭重置用户密码窗口
    closeResetPasswordDialog() {
      if (user.status === 'C') {
        this.showResetPassword = false
        console.log("close reset password dialog")
        this.userResetPasswordDialogMessage = ''
      } else {
        this.$message.error("您不是超级管理员，没有权限")
      }
    },
    // 重置用户密码
    resetPassword() {
      if (user.status === 'C') {
        let userId = this.userResetPasswordDialogMessage.id
        axios.post("/api/management/resetUserPassword", {managerId: this.user.id, userId: userId})
            .then((response) => {
              console.log(response.data)
              if (response.data.errcode === 1) {
                this.$message({
                  type: 'error',
                  message: "您没有该权限"
                });
              } else {
                this.$message({
                  type: 'success',
                  message: "成功将用户" + response.data.name + "的密码修改为" + response.data.resetPassword
                });
              }
            })
            .catch((err) => {
              console.error(err);
            })
        this.showResetPassword = false
        this.userResetPasswordDialogMessage = ''
      } else {
        this.$message.error("你不是超级管理员，没有权限")
      }
    },
    // 打开修改用户状态窗口，并显示当前状态
    openChangeUserStatusDialog(item) {
      console.log(item)
      if (this.user.status === 'D') {
        this.$message({
          type: 'error',
          message: "您没有权限"
        });
        return
      }
      console.log("open change user status dialog")
      this.userStatusDialogMessage = item
      this.selectedStatus = item.status
      this.showChangeUserStatus = true
    },
    // 关闭修改用户状态窗口
    closeChangeUserStatusDialog() {
        this.showChangeUserStatus = false
        console.log("close change user status dialog")
        this.userStatusDialogMessage = ''
        this.selectedStatus = ''
    },
    // 修改用户状态
    changeStatus() {
      console.log(this.selectedStatus)
      let userId = this.userStatusDialogMessage.id
      let managerId = this.user.id
      axios.post("/api/management/changeUserStatus", {
        managerId: managerId,
        userId: userId,
        changeToStatus: this.selectedStatus
      })
          .then((response) => {
            this.showChangeUserStatus = false
            console.log(response.data)
            if (response.data.errcode === 1) {
              this.$message({
                type: 'error',
                message: "您没有该权限"
              });
            } else if (response.data.errcode === 2) {
              let showStatus;
              if (this.selectedStatus === 'A') {
                showStatus = "正常"
              } else if (this.selectedStatus === 'B'){
                showStatus = "禁用"
              } else if (this.selectedStatus === 'D'){
                showStatus = "项目管理员"
              }
              this.$message({
                type: 'info',
                message: "用户" + this.userStatusDialogMessage.name + "的状态已为" + showStatus
              });
            } else if (response.data.errcode === 3) {
              console.log(response.data)
              this.$message({
                type: 'info',
                message: "用户" + this.userStatusDialogMessage.name + "不能设置为项目管理员"
              })
            } else {
              if (this.selectedStatus === 'A') {
                this.$message({
                  type: 'success',
                  message: "成功将用户" + this.userStatusDialogMessage.name + "的状态恢复为正常"
                });
              } else if (this.selectedStatus === 'B'){
                this.$message({
                  type: 'success',
                  message: "成功将用户" + this.userStatusDialogMessage.name + "的状态修改为禁用"
                });
              } else if (this.selectedStatus === 'D'){
              this.$message({
                type: 'success',
                message: "成功将用户" + this.userStatusDialogMessage.name + "的状态修改为项目管理员"
              });

            }
            }
            this.userStatusDialogMessage = ''
            this.selectedStatus = ''
            this.showUserMessages()
          })
          .catch((err) => {
            this.showChangeUserStatus = false
            this.userStatusDialogMessage = ''
            this.selectedStatus = ''
            console.error(err);
          })
    },
    // 打开用户个人信息窗口
    openUserProfileDialog(item) {
      if (user.status === 'C') {
        console.log(item)
        this.userProfileDialogMessage = item
        this.showUserProfile = true
      } else {
        this.$message.error("你不是超级管理员，没有权限")
      }
    },
    // 关闭用户个人信息窗口
    closeUserProfileDialog() {
      if (user.status === 'C') {
        this.showUserProfile = false
        this.userProfileDialogMessage = ''
      } else {
        this.$message.error("你不是超级管理员，没有权限")
      }
    },
    // 保存用户个人信息的修改
    saveProfile() { // TODO
      this.showUserProfile = false
    },
    // 跳转到用户端页面
    gotoUserPage(item) {
      if (this.user.status === 'D') {
        this.$message({
          type: 'error',
          message: "您没有权限"
        });
        return
      }
      console.log("232534")
      Cookies.set('manager', Cookies.get('user'))
      console.log(Cookies.get('manager'))
      axios.post("/api/getUserInfo", {
        managerId: this.user.id,
        userId: item.id,
      })
          .then((response) => {
            console.log(response.data)
            if (response.data.errcode === 1) {
              this.$message({
                type: 'error',
                message: "用户不存在"
              });
            } else {
              Cookies.set('user', JSON.stringify(response.data.data))
              this.$message({
                type: 'success',
                message: "跳转成功"
              });
              Cookies.set("from", 0)
              window.location.href = '/allProject'
            }
          })
          .catch((err) => {
            console.error(err);
          })
    },
    getColor(status) {
      if (status === "A") {
        return "green";
      } else if (status === "B") {
        return "red";
      } else if (status === "D") {
        return "orange";
      }
    },
    transform(status) {
      if (status === "A") {
        return "正常";
      } else if (status === "B") {
        return "禁用";
      } else if (status === "D") {
        return "项目管理员";
      }
    },
  },
}
</script>

<style scoped>

</style>