<template>
  <div v-if="this.selectedProj == null" style="width: 100%; height: 100%;"></div>
  <div v-else style="width: 100%; height: 100%;">
    <div class="one">
      <h1 style="position:absolute;left:5%;top:10%">项目人员列表</h1>
    </div>
    <div class="three">
      <v-data-table
          :headers="headers"
          hide-default-header
          :items="personData"
          :items-per-page="5"
          class="elevation-1"
          item-key='name'
          :search=search
          :custom-filter="filterName"
          height=""
      >
        <template v-slot:no-data>
          <div style="text-align: center;">
            <img src="../../../assets/nobody.png" height="200px" width="170px"/>
          </div>
          <div style="font-size:20px;font-weight: bold">
            没有找到成员
          </div>
        </template>
        <template v-slot:no-results>
          <div style="text-align: center;">
            <img src="../../../assets/nobody.png" height="200px" width="170px"/>
          </div>
          <div style="font-size:20px;font-weight: bold">
            没有找到成员
          </div>
        </template>
        <template v-slot:top>
          <div style="width: 100%; height: 100%;position: relative;">
            <v-row>
              <v-col cols="3">
              <v-text-field
                  v-model="search"
                  label="输入名称"
                  class="mx-4"
                  style="width:30%;display: inline-block"
              ></v-text-field>
              </v-col>
              <v-col cols="5"> &nbsp;</v-col>
              <v-col cols="2">
              <v-btn
                  style="top:20%;right:14%;height:60%;width:13%;position: absolute"
                  depressed
                  :color="getTopicColor(user.topic)"
                  @click="gotoWork"
              >查看工作量分配
                <v-icon
                    dark
                    right
                >
                  mdi-account-details
                </v-icon>
              </v-btn>
              </v-col>

              <v-col cols="2">
              <v-btn
                  style="top:20%;right:2%;height:60%;width:10%;position: absolute"
                  depressed
                  :color="getTopicColor(user.topic)"
                  @click="setupDialog = true"
              >添加成员
                <v-icon
                    dark
                    right
                >
                  mdi-account-plus
                </v-icon>
              </v-btn>
              </v-col>
            </v-row>
          </div>
        </template>
        <template v-slot:[`item.icon`]="{item}">
          <!--    <div style="height: 70px;position: relative;width:900px;">-->
          <!--      <v-avatar size="60" style="position: absolute;top:10%" color="indigo" >-->
          <!--        <span class="white&#45;&#45;text text-h5">{{ item.peopleName}}</span>-->
          <!--      </v-avatar>-->
          <!--      <p style="position:absolute;top: 5%;left: 10%;font-size: large;font-weight: 500;">{{ item.peopleName }}</p>-->
          <!--      <v-chip-->
          <!--          :color="getColor(item.peopleJob)"-->
          <!--          dark-->
          <!--          style="position:absolute;top: 50%;left: 9%"-->
          <!--        >-->
          <!--          {{ classify(item.peopleJob) }}-->
          <!--      </v-chip>-->
          <!--    </div>-->
          <v-list-item two-line>
            <v-list-item-avatar size="60" color="indigo">
              <!--            <span class="white&#45;&#45;text text-h5">{{ item.peopleName}}</span>-->
              <v-img :src="getIdenticon(item.peopleName, 60, 'user')"></v-img>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title class="text-h5"> {{ item.peopleName }}</v-list-item-title>
              <v-list-item-subtitle>
                <v-chip
                    :color="getColor(item.peopleJob)"
                    dark
                >
                  {{ classify(item.peopleJob) }}
                </v-chip>
              </v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </template>
        <template v-slot:[`item.remove`]="{item}">
          <v-btn v-if="item.peopleJob !== 'C'" depressed @click="handleDelete(item)">
            移除用户
          </v-btn>
        </template>
        <!--  <template v-slot:[`item.change`] ="{item}" >-->
        <!--     <v-btn v-if="item.peopleJob !== 'C'" depressed @click="handleChange(item)">-->
        <!--      更改角色 -->
        <!--    </v-btn>-->
        <!--  </template>-->
      </v-data-table>
    </div>
    <el-dialog
        title="添加成员"
        :visible.sync="setupDialog"
        width="50%"
        :before-close="handleClose"
        style="position:relative">
      <el-form :label-position="labelPosition" label-width="120px">
        <el-form-item label="用户名称/邮箱">
          <el-input v-model="newPersonForm.nameOrEmail"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
  <el-button @click="cancelSetupPerson">取 消</el-button>
  <el-button type="primary" @click="setupPerson">确 定</el-button>
</span>
    </el-dialog>

    <el-dialog
        :visible.sync="changeDialog"
        width="40%"
        height="30"
        :before-close="handleClose"
        style="position:relative">
      <template v-slot:title>
        <div>请选择你要为该成员分配的<strong>角色</strong></div>
      </template>
      <v-radio-group v-model="changeRoleForm.role" style="top:50px;position: absolute;">
        <v-radio value="项目经理">
          <template v-slot:label>
            <div><strong class="success--text">项目经理</strong></div>
          </template>
        </v-radio>
        <v-radio value="开发人员">
          <template v-slot:label>
            <div><strong class="orange--text">开发人员</strong></div>
          </template>
        </v-radio>
      </v-radio-group>
      <span slot="footer" class="dialog-footer">
    <el-button @click="changeDialog = false">取 消</el-button>
    <el-button type="primary" @click="changeRole">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
import {showPersonList, removeMember, showContribute, modifyRole, addMember} from '@/api/user'
import project_messagesVue from '@/views/manager/project_messages.vue'
import getIdenticon from "@/utils/identicon";
import topicSetting from "@/utils/topic-setting";

export default {
  name: 'AllProject',
  // inject: {user: {default: null},
  //          selectedProj: {default: null}},
  created() {
    if (this.selectedProj == null) {
      this.$message({
        type: 'info',
        message: '请在左侧选择项目以继续！'
      });
    } else {
      this.getPersonList();
    }
  },
  inject: {
    'user': {defualt: null},
    'selectedProj': {defualt: null},
  },
  data() {
    return {
      labelPosition: 'left',
      headers: [
        {
          text: "图标",
          align: 'start',
          sortable: false,
          value: 'icon',
        },
        {text: '移除', value: "remove"},
        {text: '更改', value: 'change'},
      ],
      radioGroup: "管理员",
      personData: [],
      //     "icon": '',
      //     'name': '梅西',
      //     "role": '管理员'
      //   },
      //   {
      //     "icon": '',
      //     'name': 'C罗',
      //     "role": '开发人员'
      //   },
      // {
      //   "icon": '',
      //   "name": '罗本',
      //   'role': '负责人'
      // }],
      search: '',
      setupDialog: false,
      changeDialog: false,
      newPersonForm: {
        nameOrEmail: '',
      },
      idTemp: '',
      changeRoleForm: {
        id: '',
        role: '',
      }
    }
  },
  methods: {
    getIdenticon,
    gotoWork() {
      showContribute({projectId: this.selectedProj.projectId, userId: this.user.id}).then(
          res => {
            let errcode = res['data']['errcode'];
            if (errcode === 1) {
              this.$message({
                type: 'error',
                message: "您没有权限查看！"
              })
            } else {
              let dict = res['data']['data'];
              let nameList = [];
              let valueList = [];
              for (let key in dict) {
                nameList.push(key);
                valueList.push(dict[key]);
              }
              this.$router.push({
                path: '/workDetail'
                , query: {
                  nameList: nameList, valueList: valueList
                }
              });
            }
          }
      )
    },
    getPersonList() {
      showPersonList({projectId: this.selectedProj.projectId, userId: this.user.id}).then(
          res => {
            console.log(res);
            this.personData = res['data']['data'];
            this.updateP
          }
      );
    },
    filterName(value, search, item) {
      console.log('123');
      console.log(item);
      return search != null &&
          item['peopleName'].toString().indexOf(search) !== -1
    },
    handleChange(row) {
      console.log('123');
      this.changeRoleForm.id = row.peopleId;
      this.changeDialog = true;
      console.log(this.changeRoleForm);
    },
    handleClose(done) {
      this.$confirm('确认关闭？')
          .then(() => {
            done();
          })
          .catch(() => {
          });
    },
    handleDelete(row) {
      this.$confirm('此操作将移除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        console.log(row.peopleJob);
        removeMember({projectId: this.selectedProj.projectId, personId: row.peopleId, userId: this.user.id}).then(
            res => {
              console.log(res);
              let errcode = res['data']['errcode'];
              if (errcode == 3) {
                this.$message({
                  type: 'error',
                  message: '您没有权限！'
                })
              }
            })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    setupPerson() {
      addMember({projectId: this.selectedProj.projectId, nameOrEmail: this.newPersonForm.nameOrEmail}).then(
          res => {
            console.log(res);
            var errorCode = res['data']['errcode'];
            console.log(errorCode);
            if (errorCode === 3) {
              this.$message({
                type: 'error',
                message: '您没有权限邀请成员'
              });
            } else if (errorCode === 1) {
              this.$message({
                type: 'error',
                message: '用户不存在'
              });
            } else if (errorCode === 2) {
              this.$message({
                type: 'error',
                message: '用户已经加入团队'
              });
            } else if (errorCode === 0) {
              this.$message({
                type: 'success',
                message: '邀请成功'
              });
              this.setupDialog = false;
              this.getPersonList();
              this.newPersonForm.nameOrEmail = '';
            }
          }
      );
    },
    cancelSetupPerson() {
      this.setupDialog = false;
      this.newPersonForm.id = '';
    },
    changeRole() {
      if (this.changeRoleForm.role === "") {
        this.$message({
          type: 'error',
          message: '您还没有选择角色！'
        });
        return;
      }
      if (this.changeRoleForm.role == '开发人员') {
        this.changeRoleForm.role = 'A';
      } else if (this.changeRoleForm.role == '项目经理') {
        this.changeRoleForm.role = 'C';
      }
      console.log(this.changeRoleForm);
      modifyRole({
        projectId: this.selectedProj.projectId,
        userId: this.user.id,
        role: this.changeRoleForm.role,
        personId: this.changeRoleForm.id
      }).then(
          res => {
            console.log(res);
            var errorCode = res['data']['errcode'];
            if (errorCode == 3) {
              this.$message({
                type: 'info',
                message: '您没有权限更改角色'
              });
            }
            this.getPersonList();
            this.changeDialog = false;
            this.changeRoleForm.id = '';
            this.changeRoleForm.role = '';
          }
      );
    },
    getColor(role) {
      if (role == 'A') {
        return 'orange';
      } else if (role == 'B') {
        return 'green';
      } else if (role == 'C') {
        return 'blue';
      } else {
        return 'red';
      }
    },
    classify(role) {
      if (role == 'A') {
        return '开发人员'
      } else if (role == 'B') {
        return '管理员'
      } else if (role == 'C') {
        return '项目经理'
      } else if (role == 'E') {
        return '禁用'
      } else {
        return '未知'
      }
    },
    getTopicColor: topicSetting.getColor
  }
}
</script>


<style scoped>
.one {
  height: 10%;
  position: relative;
}

.two {
  height: 10%;
}

.three {
  position: absolute;
  left: 5%;
  right: 5%;
  height: 80%;
}

.xiangmu {
  position: absolute;
  left: 20%;
}
</style>
