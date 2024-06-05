<template xmlns="http://www.w3.org/1999/html">
  <div>
    <v-container class="chatroom_">
      <h1>聊天室</h1>
      <!--主要页面-->
      <v-row>
        <v-col cols="3">
          <v-toolbar>
            <v-toolbar-title>历史聊天室</v-toolbar-title>
            <v-spacer></v-spacer>
            <!--搜索聊天室 -->
            <!--              <v-btn icon  small :color="getDarkColor(user.topic)">-->
            <!--                <v-icon>mdi-magnify</v-icon>-->
            <!--              </v-btn>-->
            <!-- 创建新的聊天室 }}-->
            <v-btn  icon ripple small :color="getDarkColor(user.topic)" @click="openSelectMemberDialog">
              <v-icon >mdi-plus-circle</v-icon>
            </v-btn>
          </v-toolbar>

          <!--历史聊天室 -->
          <div class="chatroom">
            <v-list style="overflow-y: auto;">
              <v-list-item-group  v-model="selectedRoom" >
                <v-list-item two-line v-for="item in chatRooms" :key="item.id" @click="selectToRoom(item)">
                  <v-list-item-content>
                    <v-list-item-title style="font-weight: bold">
                      <span :style="'color: ' + getDarkColor(user.topic) ">{{ item.title }}</span>
                      <span class="float-end grey--text">{{item.desc}}</span>
                    </v-list-item-title>
                    <v-list-item-subtitle >
                      <!--                    {{item.history.length === 0 ? ' ' : item.history[0].senderName + ' : ' }}-->
                      <!--                    {{item.history[0].type === 'A' ? item.history[0].content :-->
                      <!--                      item.history[0].type === 'B' ? '[PHOTO]' : '[FILE]'}}-->
                      <!--                    {{' (' +  new Date(item.history[0].time).toLocaleTimeString() + ")"}}-->
                      {{item.unread === 0 ? ' ' : '您有新的未读消息'}}
                      <v-badge class="float-end" v-if="item.unread > 0" color="error" :content="item.unread" inline></v-badge>
                    </v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
              <v-divider></v-divider>
            </v-list>
          </div>
        </v-col>

        <v-col cols="6">
          <!-- 聊天窗口 -->
          <v-card class="chatroom_" v-if="this.chatRooms.length !== 0 & this.selectedRoom !== null">
            <v-card-title :style="getLinearGradient(user.topic)">
              {{ roomNow.title }}
            </v-card-title>
            <!-- 聊天内容区域 -->
            <div class="messages">
              <v-card-text>
                <v-list dense>
                  <v-list-item :class="message.type === 'A' ? 'message' : 'notifyMessage' "  v-for="message in messages" :key="message.id">
                    <v-list-item-avatar size="30px" style="align-self: flex-start;">
                      <v-img :src="getIdenticon(message.from, 50, 'user')"></v-img>
                    </v-list-item-avatar>
                    <v-list-item-content v-if="message.type === 'A'">
                      <v-list-item-title >
                        {{ message.from }}
                        <span class="float-end"> {{ message.time | formatDate(message.time)}}</span>
                      </v-list-item-title>
                      <v-list-item-subtitle style="overflow: visible; white-space: normal;">
                        {{ message.content }}
                      </v-list-item-subtitle>
                    </v-list-item-content>
                    <v-list-item-content v-else>
                      <v-list-item-subtitle >
                        {{message.from}} {{message.content}}
                        <span class="float-end"> {{ message.time | formatDate(message.time)}}</span>
                      </v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                </v-list>
              </v-card-text>
            </div>
            <!-- 消息输入区域 -->
            <div class="input">
              <input v-model="messageInput" @keyup.enter="sendMessage" placeholder="Type your message here...">
              <button @click="sendMessage">Send</button>
            </div>
          </v-card>

          <!-- 没有聊天室时 -->
          <v-card class="chatroom_" v-else>
            <v-card-title>
            </v-card-title>
            <div class="messages">
            </div>
            <!-- 消息输入区域 -->
            <div class="input">
              <input :disabled = "true" v-model="messageInput" @keyup.enter="sendMessage" placeholder="Type your message here...">
              <button :disabled="true" @click="sendMessage">Send</button>
            </div>
          </v-card>

        </v-col>

        <!-- 聊天室成员-->
        <v-col cols="3">
          <div v-if="this.selectedRoom !== null" >
            <v-toolbar>
              <v-toolbar-title>聊天室成员</v-toolbar-title>
              <v-spacer></v-spacer>
              <!--拉人按钮 -->
              <v-btn v-if="this.roomNow.type === 'PUB'" icon ripple small :color="getDarkColor(user.topic)" @click="openShowInviteMember()">
                <v-icon>mdi-plus-circle</v-icon>
              </v-btn>
            </v-toolbar>


              <v-list style="overflow-y: auto;" >
                <div class="memberList">
                <v-list-item v-for="item in roomNow.users">
                  <v-list-item-avatar>
                    <v-img :src="getIdenticon(item.userName, 50, 'user')" ></v-img>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    {{ item.userName }}
                  </v-list-item-content>
                  <!--踢人按钮 -->
                  <div v-if="roomNow.type === 'PUB' && user.id === roomNow.admin.userId">
                    <v-btn v-if="item.userId !== roomNow.admin.userId" icon ripple small color="red" @click="openShowConfirmDeleteMember(item)">
                      <v-icon>mdi-minus-circle-outline</v-icon>
                    </v-btn>
                  </div>

                </v-list-item>
                  </div>
                <v-list-item v-if="this.roomNow.type==='PUB' && user.id !== this.roomNow.admin.userId" >
                  <v-btn block outlined color="red" @click="openShowConfirmLeaveRoom()" >
                    退出
                  </v-btn>
                </v-list-item>
                <v-list-item v-if="this.roomNow.type==='PUB' && user.id===this.roomNow.admin.userId" >
                  <v-btn block @click="openShowConfirmDeleteRoom()" >
                    解散群聊
                  </v-btn>
                </v-list-item>

              </v-list>
            </div>

            <v-divider></v-divider>

        </v-col>
      </v-row>


      <!--创建聊天室 -->
      <v-dialog v-model="createNewChatSheet" width="390" height="20vh" scrollable>
        <v-card >
          <v-card-title>
            选择成员
          </v-card-title>
          <v-card-subtitle v-if="selectedMembers.length > 1">
            <v-row>
              <v-col cols="6">
                <v-text-field label="聊天室名称" v-model="createRoomName"></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-text-field label="聊天室简介" v-model="createRoomDesc"></v-text-field>
              </v-col>
            </v-row>
          </v-card-subtitle>

          <v-card-text>
            <span v-if="selectedMembers.length !== 0">已选择的成员：</span>
            <v-spacer></v-spacer>
            <v-chip v-for="item in selectedMembers" class="ma-2 accent-1" color="green" :key="item.peopleId"
                    @click="() => {
                              deselectedMembers.push(item)
                              selectedMembers.splice(selectedMembers.indexOf(item), 1)
                            }"
            >
              <v-avatar left><v-img :src="getIdenticon(item.peopleName, 50, 'user')" ></v-img></v-avatar>
              {{ item.peopleName }}
            </v-chip>

            <v-spacer></v-spacer>
            <span v-if="deselectedMembers.length !== 0">成员：</span>
            <span v-else>大家都在聊天室里了哦</span>

            <v-spacer></v-spacer>
            <v-chip  v-for="item in deselectedMembers" class="ma-2 accent-1"
                     @click="() => {
                    selectedMembers.push(item)
                    deselectedMembers.splice(deselectedMembers.indexOf(item), 1)
                  }">
              <v-avatar left><v-img :src="getIdenticon(item.peopleName, 50, 'user')" ></v-img></v-avatar>
              {{ item.peopleName }}
            </v-chip >

          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn v-if="selectedMembers.length === 1 " :color="getRadialGradient(user.topic)" block ripple @click="createChatRoom">创建私聊！</v-btn>
            <v-btn v-else-if="selectedMembers.length > 1 " :color="getRadialGradient(user.topic)" block ripple @click="createChatRoom">创建群聊！</v-btn>

          </v-card-actions>
        </v-card>

      </v-dialog>

      <!-- 确认移除成员-->
      <v-dialog v-model="showConfirmDeleteMember" max-width="300" persistent>
        <v-card>
          <v-card-title> 确认移除成员
            <div v-if="this.tempSelectedMember">
              <v-chip class="ma-2 accent-1">
                <v-avatar left><v-img :src="getIdenticon(this.tempSelectedMember.userName, 50, 'user')" ></v-img></v-avatar>
                {{ this.tempSelectedMember.userName }}
              </v-chip>
            </div>
          </v-card-title>
          <v-card-text> 警告！这样做会导致成员 {{this.tempSelectedMember.userName}} 无法访问聊天室 </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="green" class="white--text" @click="closeShowConfirmDeleteMember">
              再想想
            </v-btn>
            <v-btn color="red" class="white--text" @click="deleteMember()">
              确定
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- 确认解散群聊-->
      <v-dialog v-if="this.roomNow !== null" v-model="showConfirmDeleteRoom" max-width="300" persistent>
        <v-card>
          <v-card-title> 解散群聊{{ this.roomNow.title }}？</v-card-title>
          <v-card-text> 所有聊天记录都会消失哦 </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="green" class="white--text" @click="closeShowConfirmDeleteRoom()">
              取消
            </v-btn>
            <v-btn color="red" class="white--text" @click="deleteRoom()">
              确定
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- 确认退出群聊-->
      <v-dialog v-if="this.roomNow !== null" v-model="showConfirmLeaveRoom" max-width="300" persistent>
        <v-card>
          <v-card-title> 确认退出聊天室 </v-card-title>
          <v-card-text> 退出后将无法访问聊天室哦 </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="green" class="white--text" @click="closeShowConfirmLeaveRoom()">
              取消
            </v-btn>
            <v-btn color="red" class="white--text" @click="leaveRoom()">
              确定
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!--拉人 -->
      <v-dialog v-model="showInviteMember" width="390" height="20vh" scrollable>
        <v-card >
          <v-card-title>
            添加新的成员
          </v-card-title>
          <v-card-text >
            <span v-if="this.notInRoomMember && this.notInRoomMember.length !== 0">请选择想要邀请的成员：</span>
            <span v-else>大家都在聊天室里了哦</span>
            <v-list max-height="80%" scrollable>
              <template v-for="item in this.notInRoomMember">
                <v-list-item>
                  <v-chip @click="selectInviteMember(item)">
                    <v-avatar left><v-img :src="getIdenticon(item.userName, 50, 'user')" ></v-img></v-avatar>
                    {{ item.userName }}
                  </v-chip >
                </v-list-item>
              </template>
            </v-list>
          </v-card-text>

          <v-divider></v-divider>
          <v-card-actions v-if="this.tempSelectedMember">
            <span>确认邀请成员</span>
            <div v-if="this.tempSelectedMember">
              <v-chip class="ma-2 accent-1" color="green">
                <v-avatar left><v-img :src="getIdenticon(this.tempSelectedMember.userName, 50, 'user')" ></v-img></v-avatar>
                {{ this.tempSelectedMember.userName }}
              </v-chip>
            </div>
            <v-spacer></v-spacer>
            <v-btn :color="getRadialGradient(user.topic)" ripple @click="inviteMember()">确定</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

    </v-container>
    <v-tour name="myTour" :steps="steps"></v-tour>
  </div>




</template>

<script>
import topicSetting from "@/utils/topic-setting";
import getIdenticon from "@/utils/identicon";
import ChatMessage from "@/views/chat/ChatMessage.vue";
import ChatMessageLeft from "@/views/chat/ChatMessageLeft.vue";
import axios from "axios";
import Vue from "vue";


export default {
  components: {ChatMessageLeft, ChatMessage},
  inject: {
    user: {default: null},
    proj: {default: null}
  },
  filters: {
    formatDate: function (value) {
      const date = new Date(value);
      const year = date.getFullYear();
      const month = (date.getMonth() + 1).toString().padStart(2, '0');
      const day = date.getDate().toString().padStart(2, '0');
      const hours = date.getHours().toString().padStart(2, '0');
      const minutes = date.getMinutes().toString().padStart(2, '0');
      return `${year}-${month}-${day} ${hours}:${minutes}`;
    }
  },

  created() {
    this.getProjectMember()
    this.updateChatRooms()
    //this.initWS(1)
  },
  beforeRouteLeave(to, from, next) {
    if (window.socket && window.socket.readyState === WebSocket.OPEN) {
      console.log('Closing WebSocket due to leaving the route');
      window.socket.close(1000, 'Navigating to a different route');
    }
    next();
  },

  data() {

    return {
      steps:[
        {
          target: '#v-step2',
          content: 'Created a Chat!'
        },
        {
          target: '#v-step3',
          content: 'GOOd!'
        }
      ],

      selectedRoomIndex:0,
      selectedRoom:null,
      messageInput: '',
      messages: [],
      createNewChatSheet: false,
      projectMembers:[],
      selectedMembers:[],
      deselectedMembers:[],
      createRoomName:'',
      createRoomDesc:'',

      chatRooms:[],
      search_Chatroom:'',

      groupMember:[],
      roomNow:null,
      ws:null,


      tempSelectedMember:'',
      showConfirmDeleteMember: false,
      showConfirmDeleteRoom: false,


      showInviteMember: false,
      inRoomMember:[],
      notInRoomMember:[],

      showConfirmLeaveRoom: false,


    };
  },
  mounted: function () {
    this.$tours['myTour'].start()
  },

  methods: {
    getIdenticon,

    async updateChatRooms() {
      console.log('updating chat rooms...')
      console.log(this.chatRooms)
      let prevLength = this.chatRooms.length
      try{
        let res = await axios.post('/api/chat/getRoomList', {
          projectId: this.proj.projectId,
          currentUserId: this.user.id
        })
        console.log(res.data)
        if (res.data.errcode === 0) {
          // this.chatRooms = res.data.data.rooms.map((item, index) => {
          //   const groupAdmin = item.users.find(user => user.userRole === 'B');
          //   return {
          //     id: item.roomId,
          //     title: item.roomName,
          //     desc: item.outline,
          //     users: item.users,
          //     history: [],
          //     unread: item.unReadNums,
          //     time: item.time,
          //     type: item.roomType,
          //     admin: groupAdmin ? groupAdmin : null
          //   }
          // })
          Vue.set(this, 'chatRooms', res.data.data.rooms.map((item, index) => {
            const groupAdmin = item.users.find(user => user.userRole === 'B');
            return {
              id: item.roomId,
              title: item.roomName,
              desc: item.outline || '私聊',
              users: item.users,
              history: [],
              unread: item.unReadNums,
              time: item.time,
              type: item.roomType,
              admin: groupAdmin ? groupAdmin : null
            };
          }));
        } else {
          throw new Error('get discussion list failure with non 0 errcode (' + res.data.errcode + ')')
        }

        //new
        if (this.ws === null) {
          this.ws = this.initWS()
        }

      } catch(err) {
        this.chatRooms = []
        console.error(err)
        this.$message({
          type: 'error',
          message: 'get discussion list failure with error: ' + err
        })
      } finally {
        console.log("after updating chatrooms")
        console.log(this.chatRooms)


        if (this.roomNow != null && !this.chatRooms.find(room => room.id === this.roomNow.id)) {
          this.roomNow = null
          this.selectedRoom = null
        } else if (this.roomNow != null) {
          this.roomNow.users = this.chatRooms.find(room => room.id === this.roomNow.id).users
        }
        //this.scrollToTop()
        //this.scrollToItem()
      }
    },

    // updateChatRooms() {
    //   console.log('updating chat rooms...')
    //   console.log(this.chatRooms)
    //   // // 之前的
    //   // let tempWS = {}
    //   // let tempHistory = {}
    //   // this.chatRooms.forEach((item, index) => {
    //   //   if (item.ws !== null) {
    //   //     tempWS[item.id] = item.ws
    //   //     tempHistory[item.id] = item.history
    //   //     console.log(item.ws)
    //   //     console.log(tempWS)
    //   //   }
    //   // })
    //   // console.log(tempWS)
    //
    //   axios.post('/api/chat/getRoomList', {
    //     projectId: this.proj.projectId,
    //     currentUserId: this.user.id
    //   }).then((res) => {
    //     console.log(res.data)
    //         if (res.data.errcode === 0) {
    //           this.chatRooms = res.data.data.rooms.map((item, index) => {
    //             const groupAdmin = item.users.find(user => user.userRole === 'B');
    //             return {
    //               id: item.roomId,
    //               title: item.roomName,
    //               desc: item.outline,
    //               users: item.users,
    //               // history: tempHistory[item.id] || [],
    //
    //               //new
    //               history: [],
    //               unread: item.unReadNums,
    //               time: item.time,
    //               type: item.roomType,
    //               admin: groupAdmin ? groupAdmin : null
    //             }
    //           })
    //         } else {
    //           throw new Error('get discussion list failure with non 0 errcode (' + res.data.errcode + ')')
    //         }
    //
    //   }).catch((err) => {
    //     this.chatRooms = []
    //     console.error(err)
    //     this.$message({
    //       type: 'error',
    //       message: 'get discussion list failure with error: ' + err
    //     })
    //   }).finally(() => {
    //     console.log("after updating chatrooms")
    //     console.log(this.chatRooms)
    //         // // 重新分配ws
    //         // this.chatRooms.forEach((item, index) => {
    //         //   if (tempWS[item.id] !== undefined) {
    //         //     item.ws = tempWS[item.id]
    //         //     tempWS[item.id] = undefined
    //         //     item.history = tempHistory[item.id]
    //         //   } else {
    //         //     item.ws = this.initWS(item.id)
    //         //     console.log(item)
    //         //     this.getChatHistory(item)
    //         //   }
    //         // })
    //
    //     //new
    //     if (this.ws === null) {
    //       this.ws = this.initWS()
    //     }
    //     //
    //     // // 值钱的
    //     // // 关闭多余的ws
    //     // for (const [key, value] of Object.entries(tempWS)) {
    //     //   if (value !== undefined) {
    //     //     value.close()
    //     //   }
    //     // }
    //
    //     if (this.selectedRoom) {
    //       console.log(this.selectedRoom)
    //       this.selectToRoom(this.chatRooms[this.selectedRoom])
    //     }
    //   })
    // },

    getProjectMember() {
      console.log(this.proj.projectId)
      axios.post('/api/plan/showPersonList', {
        projectId: this.proj.projectId,
        userId: this.user.id
      }).then((res) => {
        if (res.data.errcode !== 0) {
          throw new Error('get person list failure with non 0 errcode (' + res.data.errcode + ')')
        } else {
          console.log(res.data)
          this.projectMembers = res.data.data.filter((item, index) => {
            return item.peopleId !== this.user.id
          })
          this.deselectedMembers = res.data.data.filter((item, index) => {
            return item.peopleId !== this.user.id
          })

        }
      }).catch((err) => {
        console.error(err)
        this.$message({
          type: 'error',
          message: 'get person list failure with error: ' + err
        })
      })
    },

    openSelectMemberDialog() {
      this.createNewChatSheet = true
      this.selectedMembers = []
      this.deselectedMembers = []
      this.getProjectMember()
    },

    createChatRoom() {
      if (this.selectedMembers.length === 1) {
        axios.post('/api/chat/createPrivate', {
          currentUserId: this.user.id,
          projectId: this.proj.projectId,
          targetUserId: this.selectedMembers[0].peopleId
        }).then((res) => {
          this.updateChatRooms().then(() => {
            console.log("after create: go to room")
            let room = this.chatRooms.find((item) => {
              return item.id === res.data.data.roomId
            })
            console.log(room)
            this.selectedRoom = this.chatRooms.indexOf(room)
            this.selectToRoom(room)

            this.ws.send(JSON.stringify({
              type: 3,
            }));
          })
        })
      } else {
        if (this.createRoomName === '') {
          this.$message({
            type: 'warning',
            message: '请输入聊天室名称'
          })
          return
        } else if (this.createRoomDesc === '') {
          this.$message({
            type: 'warning',
            message: '请输入聊天室简介'
          })
          return
        }
        let users = this.selectedMembers.map((item, index) => {
          return item.peopleId
        })
        axios.post('/api/chat/createRoom', {
          currentUserId: this.user.id,
          projectId: this.proj.projectId,
          users: users,
          roomName: this.createRoomName,
          outline: this.createRoomDesc
        }).then((res) => {
          this.updateChatRooms().then(() => {
            console.log("after create: go to room")
            console.log(this.chatRooms)
            this.selectedRoom = 0
            this.$forceUpdate()
            this.selectToRoom(this.chatRooms[this.selectedRoom])

            this.ws.send(JSON.stringify({
              type: 3,
            }));

          })

        })
      }
      this.selectedMembers = []
      this.deselectedMembers = []
      this.createRoomName=''
      this.createRoomDesc=''
      this.createNewChatSheet = false
    },

    getChatHistory(room) {
      console.log(room)
      if (room) {
        console.log("getting chat history")
        axios.post('/api/chat/getRoomMessages', {
          roomId: room.id,
          currentUserId: this.user.id
        }).then((res) => {
          if (res.data.errcode === 0) {
            room.history = res.data.data.messages.map(((item, index) => {
              return {
                from: item.senderName,
                fromId: item.senderId,
                content: item.content,
                time: item.time,
                type: item.type
              }
            }))
            this.messages = room.history
            //this.scrollToTop()
            //this.scrollToItem()
            this.scrollToBottom()
          }
        })
      }
    },

    initWS() {
      console.log('initWS: connecting to ws://127.0.0.1:8000/ws/chat/' + this.user.id.toString() + '/' + this.proj.projectId.toString())
      const socket = new WebSocket('ws://127.0.0.1:8000/ws/chat/' + this.user.id.toString() + '/' + this.proj.projectId.toString());

      socket.onopen = (e) => {
        console.log('socket opened')
      }

      const onmessage = (fromName, fromId, content, time,type) => {
        this.updateChatRooms()
        console.log(this.roomNow)
        if (this.roomNow === undefined) {
          console.error('room not found')
        } else {
          this.roomNow.history.push({
            from: fromName,
            type: type,
            content: content,
            time: time,
          })
          console.log("显示聊天记录")
          console.log(this.messages)
          //this.scrollToBottom()
        }
      }

      const onmessage_remind = () => {
        console.log("update unread or group")
        //this.roomNow.unread = 0
        this.updateChatRooms()
      }


      socket.onmessage = function (event) {
        console.log('Message from server ', event.data);
        var data = JSON.parse(event.data)
        if (data.type === 1) {
          onmessage(data.senderName, data.senderId, data.mes, data.time, data.mes_type)
        } else if (data.type === 2) {
          onmessage_remind()
        }
      };

      socket.onerror = function (event) {
        console.error('WebSocket error observed:', event)
      }
      const initagain = () => {
        this.ws = this.initWS()
      }

      socket.onclose = function (e) {
        console.log('Socket is closed.', e.reason);
        //initagain()
      }
      window.socket=socket

      return socket
    },


    // 多个socket版 之前的
    // initWS(rid) {
    //   // console.log('initWS: connecting to ws://127.0.0.1:8000/ws/chat/' + this.user.id.toString() + '/' + rid.toString())
    //   // const socket = new WebSocket('ws://127.0.0.1:8000/ws/chat/' + this.user.id.toString() + '/' + rid.toString());
    //
    //   console.log('initWS: connecting to ws://127.0.0.1:8000/ws/chat/' + this.user.id.toString() + '/' + this.proj.projectId.toString())
    //   const socket = new WebSocket('ws://127.0.0.1:8000/ws/chat/' + this.user.id.toString() + '/' + this.proj.projectId.toString());
    //
    //   socket.onopen = (e) => {
    //     console.log('socket opened')
    //   }
    //
    //   const onmessage = (fromName, fromId, content, time) => {
    //     // 找到this.chatRooms中id为rid的room
    //     // console.log("on message "+ rid)
    //     console.log(this.roomNow)
    //     // let room = this.chatRooms.find((item, index) => {
    //     //   return item.id === rid
    //     // })
    //     if (this.roomNow === undefined) {
    //       console.error('room not found')
    //     } else {
    //       this.roomNow.history.push({
    //         from: fromName,
    //         type: 'group',
    //         content: content,
    //         time: time
    //       })
    //       console.log("显示聊天记录")
    //       console.log(this.messages)
    //       //this.scrollToBottom()
    //     }
    //   }
    //
    //
    //   socket.onmessage = function (event) {
    //     console.log('Message from server ', event.data);
    //     var data = JSON.parse(event.data)
    //     if (data.type === 1) {
    //       onmessage(data.senderName, data.senderId, data.mes, data.time)
    //     } else if (data.type === 2) {
    //
    //     }
    //
    //   };
    //
    //   socket.onerror = function (event) {
    //     console.error('WebSocket error observed:', event)
    //   }
    //
    //   socket.onclose = function (e) {
    //     console.log('Socket is closed.', e.reason);
    //   }
    //
    //   return socket
    // },

    // selectToRoom(room) {
    //
    //   this.getChatHistory(room)
    //   this.chatRooms[this.selectedRoom].ws.send(JSON.stringify({
    //     type: 1,
    //     roomId: this.chatRooms[this.selectedRoom].id,
    //   }));
    //   console.log("send msg-type 1")
    //   //this.scrollToTop()
    //   this.scrollToBottom()
    // },

    // 多个socket 版  成功的
    // selectToRoom(room) {
    //   console.log(room)
    //   this.roomNow = room
    //   console.log("select to room, roomNow")
    //   console.log(this.roomNow)
    //
    //   //this.getChatHistory(room)
    //   // room.ws.send(JSON.stringify({
    //   //   type: 1,
    //   //   roomId: room.id,
    //   // }));
    //
    //   this.getChatHistory(this.roomNow)
    //   room.ws.send(JSON.stringify({
    //     type: 1,
    //     roomId: this.roomNow.id,
    //   }));
    //   this.scrollToBottom()
    // },

    selectToRoom(room) {
      console.log(room)
      this.roomNow = room
      console.log("select to room, roomNow")
      console.log(this.roomNow)
      this.getChatHistory(this.roomNow)

      this.ws.send(JSON.stringify({
        type: 1,
        roomId: this.roomNow.id,
      }));
      //this.scrollToTop()
      //this.scrollToItem()
      this.scrollToBottom()
    },

    sendMessage() {
      if (this.messageInput.length === 0) {
        return;
      }
      if ((this.messageInput || '').length > 80) {
        this.$message({
          type: 'error',
          message: '消息太长了'
        })
        return
      }
      console.log('will send: ' + this.messageInput)
      //console.log(this.selectedRoom)
      //console.log(this.chatRooms)
      //console.log(this.chatRooms[this.selectedRoom])


      this.ws.send(JSON.stringify({
        roomId: this.roomNow.id,
        type: 2,
        mes: this.messageInput,
        mes_type: 'A'
      }));
      this.messageInput = ''
      this.scrollToBottom()
    },


    scrollToBottom() {
      this.$nextTick(() => {
        setTimeout(() => {
          let chatroom = document.getElementsByClassName("messages")[0]
          chatroom.scrollTop = chatroom.scrollHeight;
          console.log("scrolling")
        }, 100); // 延迟时间
      })
    },
    //
    // scrollToTop() {
    //   console.log("show selectedroom")
    //   console.log(this.selectedRoom)
    //   this.$nextTick(() => {
    //     setTimeout(() => {
    //       let chatroom= document.getElementsByClassName("chatroom")[0]
    //       console.log(chatroom.scrollTop)
    //       chatroom.scrollTop = 0;
    //       console.log("scrolling to top")
    //     }, 50); // 延迟时间
    //   })
    // },

    scrollToTop() {
      this.$nextTick(() => {
        const chatroom = this.$refs.chatroom;
        if (chatroom) {
          chatroom.scrollTop = 0;
        }
      });
    },

    //
    // scrollToItem() {
    //   const element = this.$refs[this.selectedRoom];
    //   if (element) {
    //     element.scrollIntoView({ behavior: "smooth", block: "start" });
    //   }
    // },

    getDarkColor: topicSetting.getDarkColor,
    getTopicColor: topicSetting.getColor,
    getLinearGradient: topicSetting.getLinearGradient,
    getRadialGradient: topicSetting.getRadialGradient,

    openShowConfirmDeleteMember(item) {
      this.showConfirmDeleteMember = true
      this.tempSelectedMember = item
    },

    closeShowConfirmDeleteMember() {
      this.showConfirmDeleteMember = false
    },

    deleteMember() {
      console.log("delete member")
      console.log(this.tempSelectedMember)
      axios.post('/api/chat/deletePerson', {
        currentUserId: this.user.id,
        targetUserId: this.tempSelectedMember.userId,
        roomId: this.roomNow.id
      }).then((res) => {
        if (res.data.errcode === 0) {
          console.log(res)
          this.updateChatRooms().then(() => {
            this.$message({
              type: 'success',
              message: '移除成功！'
            })
            //this.roomNow.users = this.chatRooms.find(room => room.id === this.roomNow.id).users
            //try提示信息
            //let message = "kick " + this.tempSelectedMember.userName
            this.ws.send(JSON.stringify( {
              type: 3,
            }))
          })
        } else {
          this.$message({
            type:'error',
            message:'移除失败！'
          })
        }
      }).catch((err) => {
        console.error(err)
      })
      this.tempSelectedMember = ''
      this.showConfirmDeleteMember = false

    },

    openShowConfirmDeleteRoom() {
      this.showConfirmDeleteRoom = true
    },

    closeShowConfirmDeleteRoom() {
      this.showConfirmDeleteRoom = false
    },

    deleteRoom() {
      console.log("delete room")
      console.log(this.roomNow)
      axios.post('/api/chat/deleteRoom', {
        roomId: this.roomNow.id,
        currentUserId: this.user.id
      }).then((res) => {
        console.log(res.data)
        if (res.data.errcode === 1) {
          this.$message({
            type: 'error',
            message: "您不是群主！"
          });
        } else if (res.data.errcode === 0) {
          this.selectedRoom = null
          this.roomNow = null
          this.updateChatRooms().then(() => {
            this.$message({
              type: 'success',
              message: "群已解散！"
            });
            this.ws.send(JSON.stringify({
              type: 3,
            }));
          });


        }
      }).catch((err) => {
        console.error(err)
      })

      this.showConfirmDeleteRoom = false

    },

    openShowInviteMember() {
      console.log("open invite member")
      console.log(this.roomNow)
      this.showInviteMember = true
      this.tempSelectedMember = ''

      this.inRoomMember = this.roomNow.users
      let allProjectMember = this.projectMembers.map((item, index) => {
        return {
          userId: item.peopleId,
          userName: item.peopleName
        }
      })
      this.notInRoomMember = allProjectMember.filter(member => {
        return !this.inRoomMember.some(selectedMember => selectedMember.userId === member.userId);
      })
      console.log(this.notInRoomMember)
      console.log(this.tempSelectedMember)
    },

    selectInviteMember(item) {
      this.tempSelectedMember = item
    },

    inviteMember() {
      console.log("inviting member")
      console.log(this.tempSelectedMember)

      axios.post('/api/chat/addPerson', {
        targetUserId: this.tempSelectedMember.userId,
        roomId: this.roomNow.id
      }).then((res) => {
        if (res.data.errcode === 0) {
          this.updateChatRooms().then(() => {
            this.$message({
              type: 'success',
              message: '邀请成功！'
            })
            //this.roomNow.users = this.chatRooms.find(room => room.id === this.roomNow.id).users
          })
          //
          let message = "邀请了" + this.tempSelectedMember.userName + "加入聊天室"
          console.log('will send: ' + message)
          this.ws.send(JSON.stringify({
            roomId: this.roomNow.id,
            type: 2,
            mes: message,
            mes_type: 'D' //通知消息
          }));
          // this.ws.send(JSON.stringify( {
          //   type: 3,
          // }))
        }
      }).catch((err) => {
        console.error(err)
      })
      this.showInviteMember = false
    },

    openShowConfirmLeaveRoom() {
      this.showConfirmLeaveRoom = true
    },

    closeShowConfirmLeaveRoom() {
      this.showConfirmLeaveRoom = false
    },

    leaveRoom() {
      console.log("leave room")
      axios.post('/api/chat/exitRoom', {
        currentUserId: this.user.id,
        roomId: this.roomNow.id
      }).then((res) => {
        console.log(res)
        if (res.data.errcode === 1) {
          this.$message({
            type: 'error',
            message: '您是群主，无法退出群聊！'
          })
        } else if (res.data.errcode === 0) {
          this.selectedRoom = null
          this.roomNow = null
          this.updateChatRooms().then(() => {
            this.$message({
              type: 'success',
              message: "已退出群聊！"
            });
            this.ws.send(JSON.stringify({
              type: 3,
            }));
          })
        }
      }).catch((err) => {
        console.error(err)
      })
      this.showConfirmLeaveRoom = false

    }
  }
};
</script>

<style>
.chatroom_ {
  height: 75vh;
  display: flex;
  flex-direction: column;
}

.chatroom {
  height: 67vh;
  display: flex;
  flex-direction: column;
}

.memberList {
  max-height: 60vh;
  flex-direction: column;
  overflow-y: auto;
}

.messages {
  flex: 1;
  overflow-y: auto;
}

.chatroom_box {
  height:80vh;
  flex: 1;
  overflow-y: auto;
}

.background {
  --color-background: pink;
}

.input {
  display: flex;
  align-items: center;
  padding: 10px;
  background-color: #f0f0f0;
}

.input input {
  flex: 1;
  padding: 8px;
  border-radius: 20px;
  border: none;
  outline: none;
  margin-right: 10px;
}

.input button {
  padding: 8px 20px;
  border-radius: 20px;
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
}

.notifyMessage {
  border-radius: 10px 25px;
  background-color: #f0f0f0;
}

.message {
  color:white ;
}
</style>
