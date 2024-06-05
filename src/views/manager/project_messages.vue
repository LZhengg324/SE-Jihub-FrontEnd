<template>
  <v-container>
    <v-card>
      <v-card-title>
        <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="搜索"
            single-line
            hide-details
        ></v-text-field>
      </v-card-title>
      <v-data-table
          :headers="headers"
          :items="projectMessages"
          :search="search"

      >
        <template #item.access="{item}">
          <v-chip :color="getColor(item.access)" dark @click="openChangeProjectAccessDialog(item)">
            {{ transform(item.access) }}
<!--            <v-text v-if="item.access==='A'"> 正常 </v-text>-->
<!--            <v-text v-if="item.access==='B'"> 禁用 </v-text>-->
          </v-chip>
        </template>

        <template #item.name="{item}">
          <v-btn text style="font-family: 'Arial',serif;" @click="openProjectMemberDialog(item)">{{item.name}} </v-btn>
        </template>

<!--        <template #item.progress="{item}">-->
<!--          {{ item.progress }} %-->
<!--        </template>-->
<!--        <template #item.changeAccess="{item}">-->
<!--          <v-btn class="ml-1" small outlined @click="openChangeProjectAccessDialog(item)">修改项目状态</v-btn>-->
<!--        </template>-->
        <template #item.projectDetail="{item}">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn icon color="blue" v-bind="attrs" v-on="on" @click="gotoProjectDetailPage(item)">
                <v-icon>mdi-link-variant</v-icon>
              </v-btn>
            </template>
            <span>以项目负责人身份进入用户端的项目界面</span>
          </v-tooltip>
        </template>
        <template v-if="this.user.status === 'C'" #item.showAssistants="{item}" >
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn icon color="orange" v-bind="attrs" v-on="on" class="ml-1" rounded small @click="openChangeProjectAssistantDialog(item)">
                <v-icon>mdi-account-tie</v-icon>
              </v-btn>
            </template>
            <span>设置项目管理员</span>
          </v-tooltip>
        </template>

        <template #item.createTime="{item}">
          {{ pro(item.createTime) }}
        </template>
      </v-data-table>
    </v-card>
    <v-dialog v-model="showChangeProjectAccess" width="300">
      <template>
        <v-container class="pa-0">
          <v-card>
            <v-card-title class="headline font-weight text-left"> 修改项目状态 </v-card-title>
            <v-card-text> 项目名:{{ changeProjectAccessMessage.name }} / 创建人:{{ changeProjectAccessMessage.leader }}</v-card-text>
            <v-divider></v-divider>
            <v-card-text>
              <v-radio-group v-model="selectedAccess">
                <v-radio v-for="i in accessList" :key="i.value" :label="i.label" :value="i.value"></v-radio>
              </v-radio-group>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="red" text @click="closeChangeProjectAccessDialog">取消</v-btn>
              <v-btn color="blue" text @click="changeAccess">确认修改</v-btn>
            </v-card-actions>
          </v-card>
        </v-container>
      </template>
    </v-dialog>
    <v-dialog v-model="showChangeProjectAssistant" width="300">
      <template>
        <v-container class="pa-0">
          <v-card>
            <v-card-title class="headline font-weight text-left"> 设置项目管理员 </v-card-title>
            <v-card-text> 项目名:{{ changeProjectAssistantMessage.name }} / 创建人:{{ changeProjectAssistantMessage.leader }}</v-card-text>
            <v-divider></v-divider>
            <v-card-text>
              <v-checkbox v-model="i.checked" v-for="i in assistantList"  :label="i.name + ' (' + i.email + ')'" :value="i.label"></v-checkbox>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="red" text @click="closeChangeProjectAssistantDialog">取消</v-btn>
              <v-btn color="blue" text @click="changeProjectAssistant">确认修改</v-btn>
            </v-card-actions>
          </v-card>
        </v-container>
      </template>
    </v-dialog>

    <!-- -->
    <v-dialog v-model="showProjectMemberDialog" width="800">
      <template>
        <v-container class="pa-0">
          <v-card>
            <v-card-title class="headline font-weight text-left"> 查看项目人员 </v-card-title>
            <v-card-text> 项目名:{{ showMemberProjectMessage.name }} / 创建人:{{ showMemberProjectMessage.leader }}</v-card-text>
            <v-divider></v-divider>
            <v-card-text>
              <v-text-field
                  v-model="memberSearch"
                  append-icon="mdi-magnify"
                  label="搜索"
                  single-line
                  hide-details
              ></v-text-field>
              <v-divider></v-divider>
              <v-data-table
                :headers="memberHeaders"
                :items="projectMember"
                :search="memberSearch"
              >
                <template #item.changeStatus="{item}">
                  <v-chip :color="getColorStatus(item.peopleStatus)" dark @click="openChangeMemberAccessDialog(item)">
                    {{ classifyStatus(item.peopleStatus) }}
                  </v-chip>
                </template>
              </v-data-table>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue" text @click="closeProjectMemberDialog">关闭</v-btn>
            </v-card-actions>
          </v-card>
        </v-container>
      </template>
    </v-dialog>
    <v-dialog v-model="showChangeMemberAccessDialog" width="400">
      <template>
        <v-container class="pa-0">
          <v-card>
            <v-card-title class="headline font-weight text-left">
              修改用户 {{changeMemberAccessMessage.peopleName}} 对 {{showMemberProjectMessage.name}} 的提交权限
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text>
              <v-radio-group v-model="selectedStatus">
                <v-radio v-for="i in statusList" :key="i.value" :label="i.label" :value="i.value"></v-radio>
              </v-radio-group>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="red" text @click="closeChangeMemberAccessDialog">取消</v-btn>
              <v-btn color="blue" text @click="changeMemberStatus">确认修改</v-btn>
            </v-card-actions>
          </v-card>
        </v-container>
      </template>
    </v-dialog>
  </v-container>
</template>

<script>
import axios from "axios";
import util from "@/views/util";
import Cookies from "js-cookie";

export default {
  inject: {
    user: { default: null },
    getProj: { default: null },
    //setFrom: { default: null }
  },
  data () {
    return {
      search: '',
      headers: [
        {
          text: '项目名',
          align: 'start',
          // sortable: false,
          value: 'name',
        },
        { text: '创建人', value: 'leader' },
        { text: '创建人邮箱', value: 'email' },
        { text: '创建时间', value: 'createTime' },
        // { text: '完成进度', value: 'progress' },
        { text: '状态', value: 'access' },
        {
          text: '',
          sortable: false,
          value: 'changeAccess'
        },
        {
          text: '',
          sortable: false,
          value: 'showAssistants',
        },
        {
          text: '',
          sortable: false,
          value: 'projectDetail'
        }
      ],
      projectMessages: [
        // {
        //   name: 'project1',
        //   projectId: 0,
        //   leader: '123',
        //   leaderId: 1,
        //   email: '123@qq.com',
        //   createTime: '2023.1.1',
        //   progress: 80,
        //   status: 'A',
        //   access: 'A',
        // },
        // {
        //   name: 'faskfl',
        //   projectId: 1,
        //   leader: '432gsdf',
        //   leaderId: 2,
        //   email: 'gers@qq.com',
        //   createTime: '2023.2.1',
        //   progress: '60',
        //   status: 'B',
        //   access: 'A',
        // },
        // {
        //   name: 'project3',
        //   projectId: 2,
        //   leader: '435',
        //   leaderId: 1,
        //   email: '53@qq.com',
        //   createTime: '2023.3.1',
        //   progress: '100',
        //   status: 'A',
        //   access: 'A'
        // },
      ],
      showChangeProjectAccess: false,
      changeProjectAccessMessage: '',
      selectedAccess: '',
      accessList: [
        {
          label: '正常',
          value: 'A'
        },
        {
          label: '禁用',
          value: 'B'
        },
      ],
      //设置项目小管理员
      showChangeProjectAssistant: false,
      changeProjectAssistantMessage: '',
      selectedAssistant:[],
      assistantList:[],


      //查看项目人员
      memberHeaders: [
        {
          text: '用户名',
          align: 'start',
          value: 'peopleName',
        },
        { text: '邮箱', value: 'peopleEmail' },
        {
          text: '状态',
          sortable: false,
          value: 'changeStatus'
        },
      ],
      memberSearch:'',
      projectMember:[],
      showProjectMemberDialog: false,
      showMemberProjectMessage:'',

      //修改项目人员提交权限
      showChangeMemberAccessDialog:false,
      changeMemberAccessMessage:'',
      selectedStatus: '',
      statusList:[
        {
            label:'开发人员',
            value:'A',
        },
        {
          label:'项目经理',
          value:'C',
        },
        {
          label:'禁用',
          value:'E',
        }
      ]
    }
  },
  created() {
    this.showProjectMessages()
  },
  // TODO：传给后端管理员id，如果报错，不显示信息而显示弹窗
  methods: {
    pro(createTime) {
      return util.processTime(createTime);
    },
    // 显示项目信息
    showProjectMessages() {
      console.log(this.user.id)
      axios.post("/api/management/showAllProjects", {managerId: this.user.id})
          .then((response) => {
            console.log(response)
            if (response.data.errcode === 1) {
              window.alert("您没有权限")
            } else {
              this.projectMessages = response.data.projects
            }
          })
          .catch((err) => {
            console.error(err);
            this.projectMessages = null
          })
    },
    // 打开修改项目状态窗口，并显示当前状态
    openChangeProjectAccessDialog(item) {
      console.log(item)
      this.changeProjectAccessMessage = item
      this.selectedAccess = item.access
      this.showChangeProjectAccess = true
    },
    // 关闭修改项目状态窗口
    closeChangeProjectAccessDialog() {
      this.showChangeProjectAccess = false
    },
    // 修改项目状态
    changeAccess() {
      console.log(this.selectedAccess)
      let projectId = this.changeProjectAccessMessage.projectId
      let managerId = this.user.id
      axios.post("/api/management/changeProjectAccess", {
        managerId: managerId,
        projectId: projectId,
        changeToAccess: this.selectedAccess
      })
          .then((response) => {
            this.showChangeProjectAccess = false
            console.log(response.data)
            if (response.data.errcode === 1) {
              this.$message({
                type: 'error',
                message: "您没有权限"
              });
            } else if (response.data.errcode === 2) {
              let showAccess;
              if (this.selectedAccess === 'A') {
                showAccess = "正常"
              } else {
                showAccess = "禁用"
              }
              this.$message({
                type: 'success',
                message: "项目" + this.changeProjectAccessMessage.name + "的状态已为" + showAccess
              });
            } else {
              if (this.selectedAccess === 'A') {
                this.$message({
                  type: 'success',
                  message: "成功将项目" + this.changeProjectAccessMessage.name + "的状态恢复为正常"
                });
              } else {
                this.$message({
                  type: 'success',
                  message: "成功将项目" + this.changeProjectAccessMessage.name + "的状态修改为禁用"
                });
              }
              this.showProjectMessages()
            }
            this.changeProjectAccessMessage = ''
            this.selectedAccess = ''
          })
          .catch((err) => {
            this.showChangeProjectAccess = false
            this.changeProjectAccessMessage = ''
            this.selectedAccess = ''
            console.error(err);
          })
    },
    // 跳转到项目详情页面
    gotoProjectDetailPage(item) {
      console.log("232534")
      Cookies.set('manager', Cookies.get('user'))
      console.log(Cookies.get('manager'))
      console.log(item.projectId)
      console.log(item.leaderId)
      axios.post("/api/getUserInfo", {
        managerId: this.user.id,
        userId: item.leaderId,
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
            }
          })
          .catch((err) => {
            console.error(err);
          })
      axios.post("/api/plan/ProjectInfo", {
        projectId: item.projectId,
      })
          .then((response) => {
            console.log(response.data)
            if (response.data.errcode === 1) {
              this.$message({
                type: 'error',
                message: "项目不存在"
              });
            } else if (response.data.errcode === 3) {
              this.$message({
                type: 'error',
                message: "您不具有该权限"
              });
            } else {
              console.log(response.data.data)
              this.$message({
                type: 'success',
                message: "跳转成功"
              });
              Cookies.set("from", 1)
              this.getProj(response.data.data)
              // window.location.href = '/allProject'
            }
          })
          .catch((err) => {
            console.error(err);
          })
    },

    //项目状态
    getColor(access) {
      if (access === "A") {
        return "green";
      } else if (access === "B") {
        return "red";
      }
    },
    transform(access) {
      if (access === "A") {
        return "正常";
      } else if (access === "B") {
        return "禁用";
      }
    },

    //项目人员状态
    getColorStatus(role) {
      if (role == 'A') {
        return 'orange';
      } else if (role == 'B') {
        return 'green';
      } else if (role == 'C'){
        return 'blue';
      } else {
        return 'red';
      }
    },
    classifyStatus(role) {
      if (role == 'A') {
        return '开发人员'
      } else if (role == 'B') {
        return '管理员'
      } else if (role == 'C') {
        return '项目经理'
      } else if (role == 'E') {
        return '禁用'
      }
    },

    // 打开设置项目小管理员窗口，并显示当前管理员
    openChangeProjectAssistantDialog(item) {
      //
      console.log(item)
      this.changeProjectAssistantMessage = item
      this.showAllAssistant()
      this.showProjectSelectedAssistant(item);
      this.showChangeProjectAssistant = true

    },

    // 关闭设置项目小管理员窗口
    closeChangeProjectAssistantDialog() {
      this.showChangeProjectAssistant = false
      this.selectedAssistant = []
      this.changeProjectAssistantMessage = ''
    },

    showAllAssistant() {
      console.log(this.user.id)
      axios.post("/api/management/showAssistants", {managerId: this.user.id})
          .then((response) => {
            console.log(response)
            if (response.data.errcode === 1) {
              window.alert("您没有权限")
            } else {
              this.assistantList = response.data.users;
            }
          })
          .catch((err) => {
            console.error(err);
            this.assistantList = null
          })
      console.log(this.assistantList)
    },

    showProjectSelectedAssistant(item) {
      console.log(item)
      console.log(this.assistantList)
      axios.post("/api/management/getProjectAssistants", {managerId: this.user.id , projectId: item.projectId})
          .then((response) => {
            console.log(response)
            if (response.data.errcode === 1) {
              window.alert("您没有权限")
            } else {
              this.selectedAssistant = response.data.assistants
              console.log(this.selectedAssistant)
              this.assistantList.forEach(assistant => {
                  assistant.checked = !!this.selectedAssistant.find(a => a.id === assistant.id);
                  console.log(assistant)
                });
              console.log(this.assistantList)
              this.$forceUpdate();
            }
          })
          .catch((err) => {
            console.error(err);
            this.selectedAssistant = null
          })
    },

    changeProjectAssistant() {
      if (this.assistantList.every(i=>!i.checked)) {
          console.log("error")
          this.$message({
            type: 'warning',
            message: "请选择至少一名项目管理员！"
          })
        } else {
          this.selectedAssistant = this.assistantList.filter(i => i.checked)
          console.log(this.selectedAssistant)
        let projectId = this.changeProjectAssistantMessage.projectId
        let managerId = this.user.id
        axios.post("/api/management/setAssistantAccess", {
          managerId: managerId,
          projectId: projectId,
          assistantsId: this.selectedAssistant.map(assistant => assistant.id)
        })
            .then((response) => {
              this.showChangeProjectAssistant = false
              console.log(response.data)
              if (response.data.errcode === 1) {
                this.$message({
                  type: 'error',
                  message: "您没有权限"
                });
              } else {
                this.$message({
                  type: "success",
                  message: "成功设置项目管理员"
                })
                this.showProjectMessages()
              }
              this.changeProjectAssistantMessage = ''
              this.selectedAssistant = ''
            })
            .catch((err) => {
              this.showChangeProjectAssistant = false
              this.changeProjectAssistantMessage = ''
              this.selectedAssistant = ''
              console.error(err);
            })
      }
      //console.log(this.selectedAssistant)

      // if (this.assistantList.every(i=>!i.checked)) {
      //   console.log("error")
      //   this.$message({
      //     type: 'warning',
      //     message: "请选择至少一名小管理员！"
      //   })
      // } else {
      //   this.selectedAssistant = this.assistantList.filter(i => i.checked)
      //   console.log(this.selectedAssistant)
      //   let projectId = this.changeProjectAssistantMessage.projectId
      //   let managerId = this.user.id
      //   console.log(projectId +": set success")
      //   this.$message({
      //     type: "success",
      //     message: "成功设置小管理员"
      //
      //   })
      //   this.showChangeProjectAssistant = false
      //   this.selectedAssistant = []
      //   this.changeProjectAssistantMessage = ''
      // }
    },


    //获取项目人员信息
    showAllProjectMembers(item) {
      let projectId = item.projectId
      let managerId = this.user.id
      axios.post("/api/management/getProjectUsers", {projectId: projectId, managerId: managerId})
          .then((response) => {
            console.log(response.data)
            if (response.data.errcode === 1) {
              this.$message({
                type: 'error',
                message: "您没有该权限"
              });
            } else {
              console.log("load member details success")
              this.projectMember = response.data.users
            }
          })
          .catch((err) => {
            console.error(err);
            this.userMessages = null
          })
    },


    //打开显示项目人员页面
    openProjectMemberDialog(item) {
      console.log(item)
      this.showMemberProjectMessage = item
      this.showAllProjectMembers(item)
      this.showProjectMemberDialog = true

    },

    //关闭显示项目人员页面
    closeProjectMemberDialog() {
      this.showProjectMemberDialog = false
      console.log("close project member dialog")
      this.projectMember = []
      this.memberSearch = ''
      this.showMemberProjectMessage = ''
      this.selectedAccess = ''
    },

    //打开修改项目人员提交权限页面
    openChangeMemberAccessDialog(item) {
      console.log(item)
      this.changeMemberAccessMessage = item
      this.selectedStatus = item.peopleStatus
      this.showChangeMemberAccessDialog = true
    },

    //打开修改项目人员提交权限页面
    closeChangeMemberAccessDialog() {
      this.showChangeMemberAccessDialog = false
      this.selectedStatus = ''
      this.changeMemberAccessMessage = ''
    },

    changeMemberStatus() {
      console.log(this.selectedStatus)
      let projectId = this.showMemberProjectMessage.projectId
      let managerId = this.user.id
      let memberId = this.changeMemberAccessMessage.peopleId

      axios.post("/api/plan/modifyRole", {
        projectId: projectId,
        personId: memberId,
        role: this.selectedStatus,
        userId: managerId
      })
          .then((response) => {
            this.showChangeMemberAccessDialog = false
            console.log(response.data)
            if (response.data.errcode === 1) {
              this.$message({
                type: 'error',
                message: "用户、项目、状态不存在"
              });
            } else if (response.data.errcode === 3) {
              this.$message({
                type: 'error',
                message: "出现错误"
              });

            } else {
              if (this.selectedStatus === 'A') {
                this.$message({
                  type: 'success',
                  message: "成功将用户" +  this.changeMemberAccessMessage.peopleName + "的状态修改为开发人员"
                });
              } else if (this.selectedStatus === 'B') {
                this.$message({
                  type: 'success',
                  message: "成功将用户" +  this.changeMemberAccessMessage.peopleName + "的状态修改为管理员"
                });
              } else if (this.selectedStatus === 'C') {
                this.$message({
                  type: 'success',
                  message: "成功将用户" +  this.changeMemberAccessMessage.peopleName + "的状态修改为项目经理"
                });
              } else {
                this.$message({
                  type: 'success',
                  message: "成功将用户" +  this.changeMemberAccessMessage.peopleName + "的状态修改为禁用"
                });
              }
              this.showAllProjectMembers(this.showMemberProjectMessage)
            }
            this.selectedAccess = ''
            this.changeMemberAccessMessage = ''
          })
    }
  },
}
</script>

<style scoped>

</style>