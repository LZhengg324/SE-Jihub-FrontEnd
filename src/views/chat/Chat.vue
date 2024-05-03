<template>
  <v-container class="chatroom_">
    <h1>聊天室</h1>
    <v-row>
        <v-col cols="3" class="chatroom_">
          <v-list style="overflow-y: auto;">
            <v-toolbar>
              <v-toolbar-title>历史聊天室</v-toolbar-title>
              <v-spacer></v-spacer>
              <v-btn icon  small :color="getDarkColor(user.topic)">
                <v-icon>mdi-magnify</v-icon>
                <!-- 创建新的聊天室 }}-->
              </v-btn>
              <v-btn icon ripple small :color="getDarkColor(user.topic)" @click="openSelectMemberDialog">
                <v-icon>mdi-plus-circle</v-icon>
              </v-btn>
            </v-toolbar>

            <v-list-item-group v-model="selectedRoom">
              <v-list-item two-line v-for="item in chatRooms" :key="item.id" @click="selectToRoom(item)">
<!--                <v-list-item two-line v-for="item in chatRooms" :key="item.id" @click="selectToRoom(item)">-->

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
        </v-col>

      <v-col cols="6">
          <!-- 聊天内容区域 -->
        <v-card class="chatroom_">
          <v-card-title v-if="this.chatRooms.length !== 0">
            {{ this.chatRooms[this.selectedRoom].title }}
          </v-card-title>
        <div class="messages">

          <v-card-text>
            <v-list dense>
<!--              <v-list-item v-for="message in chatRooms[selectedRoom].history" :key="message.id">-->
                <v-list-item v-for="message in messages" :key="message.id" >
                  <v-list-item-avatar size="30px" style="align-self: flex-start;">
                    <v-img :src="getIdenticon(message.from, 50, 'user')"></v-img>
                  </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title >
                    {{ message.from }}
                  </v-list-item-title>
                  <v-list-item-subtitle style="overflow: visible; white-space: normal;">
                    {{ message.content }}
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
      </v-col>

      <!-- 创建新的聊天室 }}-->
      <v-col cols="3" class="chatroom_">
        <v-list style="overflow-y: auto;">
          <v-toolbar>
            <v-toolbar-title>聊天室成员</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn icon ripple small :color="getDarkColor(user.topic)" @click="openSelectMemberDialog">
              <v-icon>mdi-plus-circle</v-icon>
            </v-btn>
            <v-btn icon  small :color="getDarkColor(user.topic)">
              <v-icon>mdi-minus</v-icon>
              <!-- 创建新的聊天室 }}-->
            </v-btn>

          </v-toolbar>

          <v-list-item-group v-model="selectedRoom">
            <v-list-item two-line v-for="item in chatRooms" :key="item.id" @click="selectToRoom(item)">
              <!--                <v-list-item two-line v-for="item in chatRooms" :key="item.id" @click="selectToRoom(item)">-->

              <v-list-item-content>
                <v-list-item-title style="font-weight: bold">
                  <span :style="'color: ' + getDarkColor(user.topic) ">{{ item.title }}</span>
                  <span class="float-end grey--text">{{item.desc}}</span>
                </v-list-item-title>
                <v-list-item-subtitle>
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

          <v-list max-height="80%" scrollable>
            <template v-for="item in deselectedMembers">
              <v-list-item>
                <v-chip @click="() => {
                    selectedMembers.push(item)
                    deselectedMembers.splice(deselectedMembers.indexOf(item), 1)
                  }">
                  <v-avatar left><v-img :src="getIdenticon(item.peopleName, 50, 'user')" ></v-img></v-avatar>
                  {{ item.peopleName }}
                </v-chip >
              </v-list-item>
            </template>
          </v-list>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn v-if="selectedMembers.length === 1 " :color="getRadialGradient(user.topic)" block ripple @click="createChatRoom">创建私聊！</v-btn>
          <v-btn v-else-if="selectedMembers.length > 1 " :color="getRadialGradient(user.topic)" block ripple @click="createChatRoom">创建群聊！</v-btn>

        </v-card-actions>

      </v-card>

    </v-dialog>



  </v-container>





</template>

<script>
import topicSetting from "@/utils/topic-setting";
import getIdenticon from "@/utils/identicon";
import ChatMessage from "@/views/chat/ChatMessage.vue";
import ChatMessageLeft from "@/views/chat/ChatMessageLeft.vue";
import axios from "axios";


export default {
  components: {ChatMessageLeft, ChatMessage},
  inject: {
    user: {default: null},
    proj: {default: null}
  },

  created() {
    this.getProjectMember()
    this.updateChatRooms()
    //this.initWS(1)
  },
  data() {

    return {

      selectedRoomIndex:0,
      selectedRoom:0,
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
    };
  },
  methods: {
    getIdenticon,

    updateChatRooms() {
      console.log('updating chat rooms...')
      let tempWS = {}
      let tempHistory = {}
      this.chatRooms.forEach((item, index) => {
        if (item.ws !== null) {
          tempWS[item.id] = item.ws
          tempHistory[item.id] = item.history
        }
      })


      axios.post('/api/chat/discussions', {
        projectId: this.proj.projectId,
        currentUserId: this.user.id
      }).then((res) => {
        console.log(res.data)
            if (res.data.errcode === 0) {
              this.chatRooms = res.data.data.discussions.map((item, index) => {
                return {
                  id: item.roomId,
                  title: item.roomName,
                  desc: item.outline,
                  users: item.users,
                  history: tempHistory[item.id] || [],
                  unread: 0,
                }
              })

              console.log(this.chatRooms)

            } else {
              throw new Error('get discussion list failure with non 0 errcode (' + res.data.errcode + ')')
            }

      }).catch((err) => {
        this.chatRooms = []
        console.error(err)
        this.$message({
          type: 'error',
          message: 'get discussion list failure with error: ' + err
        })
      }).finally(() => {
            // 重新分配ws
            this.chatRooms.forEach((item, index) => {
              if (tempWS[item.id] !== undefined) {
                item.ws = tempWS[item.id]
                tempWS[item.id] = undefined
                item.history = tempHistory[item.id]
              } else {
                item.ws = this.initWS(item.id)
                console.log(item)
                this.getChatHistory(item)
              }
            })

        // 关闭多余的ws
        for (const [key, value] of Object.entries(tempWS)) {
          if (value !== undefined) {
            value.close()
          }
        }
      })
    },

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
        }).finally( () => {
          this.updateChatRooms()
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
        }).finally(() => {
          this.updateChatRooms()
        })
      }
      this.selectedMembers = []
      this.deselectedMembers = []
      this.createRoomName=''
      this.createRoomDesc=''
      this.createNewChatSheet = false
    },

    getChatHistory(room) {
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
          this.scrollToBottom()
        }
      })
    },


    initWS(rid) {
      console.log('initWS: connecting to ws://127.0.0.1:8000/ws/chat/' + this.user.id.toString() + '/' + rid.toString())
      const socket = new WebSocket('ws://127.0.0.1:8000/ws/chat/' + this.user.id.toString() + '/' + rid.toString());

      const onopen = (e) => {
        console.log('socket opened')
      }

      socket.onopen = onopen

      const onmessage = (fromName, fromId, content, time) => {
        // 找到this.chatRooms中id为rid的room
        let room = this.chatRooms.find((item, index) => {
          return item.id === rid
        })
        if (room === undefined) {
          console.error('room not found')
        } else {
          room.history.push({
            from: fromName,
            type: 'group',
            content: content,
            time: time
          })
          //this.chatRooms[this.selectedRoom].history.push({user:this.user.name, content:this.messageInput})
          // if (this.messageServiceAvailable && document.visibilityState === 'hidden' && fromName !== this.user.name) {
          //   const notification = new Notification(`来自讨论室 ${room.title} 的一条新消息`, {
          //     body: `${fromName}: ${content}`,
          //     icon: getIdenticon(fromName, 100, 'user')
          //   })
          // }
          //this.messages.push({user: fromName, content: content})
          console.log("显示聊天记录")
          console.log(this.messages)
          //this.scrollToBottom()
        }
      }

      socket.onmessage = function (event) {
        console.log('Message from server ', event.data);
        var data = JSON.parse(event.data)
        onmessage(data.senderName, data.senderId, data.content, data.time)
      };

      socket.onerror = function (event) {
        console.error('WebSocket error observed:', event)
      }

      socket.onclose = function (e) {
        console.log('Socket is closed.', e.reason);
      }

      return socket
    },

    selectToRoom(room) {
      this.getChatHistory(room)
      //this.scrollToTop()
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
      this.chatRooms[this.selectedRoom].ws.send(JSON.stringify({
        sender: this.user.id,
        type: 1,
        message: this.messageInput
      }));
      // this.messages.push({user:this.user.name, content:this.messageInput})

      this.messageInput = ''
      //this.scrollToTop()
      this.scrollToBottom()
    },


    scrollToBottom() {
      this.$nextTick(() => {
        setTimeout(() => {
          let chatroom = document.getElementsByClassName("messages")[0]
          chatroom.scrollTop = chatroom.scrollHeight;
          console.log("scrolling")
        }, 50); // 延迟时间
      })
    },

    getDarkColor: topicSetting.getDarkColor,
    getTopicColor: topicSetting.getColor,
    getLinearGradient: topicSetting.getLinearGradient,
    getRadialGradient: topicSetting.getRadialGradient
  }
};
</script>

<style>
.chatroom_ {
  height: 80vh;
  display: flex;
  flex-direction: column;
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

</style>
