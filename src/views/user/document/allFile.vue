<template>
  <v-card style="position: relative;">
    <v-toolbar
        dark
        :src="getUrl(user.topic)"
    >
      <v-btn
          icon
          dark
          @click="close"
      >
        <v-icon>mdi-close</v-icon>
      </v-btn>
      <v-toolbar-title>共享文档</v-toolbar-title>
    </v-toolbar>


    <v-data-table
        :headers="headers"
        :items="showCollect ? collectDocList : documentData"
        :single-expand="true"
        class="elevation-1"
        item-key="token"
        :search="search"
        show-expand
        :expanded.sync="expanded"
        :custom-filter="filterOnlyCapsText"
        style="position:absolute;left:3%;width:94%;height: 70%;top:14%"
    >

      <template v-slot:top>
        <div style="width: 100%; height: 10%; position: relative; align-items: center;">
          <v-text-field
              v-model="search"
              label="请输入文档名称进行查询"
              class="mx-4"
              append-icon="mdi-magnify"
              style="height:40px; width: 35%; display: inline-flex"
          ></v-text-field>
          <v-chip class="ma-2" v-if="!showCollect" text-color="blue" @click="showCollect=true"> 全部 </v-chip>
          <v-chip class="ma-2" v-else @click="showCollect=false" > 全部 </v-chip>
          <v-chip class="ma-2" v-if="!showCollect"  @click="showCollect=true"> 星标 </v-chip>
          <v-chip class="ma-2" v-else text-color="blue" @click="showCollect=false"> 星标 </v-chip>

          <v-btn
              style="
                top: 20%;
                right: 2%;
                height: 60%;
                width: 10%;
                position: absolute;
              "
              depressed
              :color="getTopicColor(user.topic)"
              @click="newDocumentForm.name = '';newDocumentForm.intro = '';dialog1 = true;"
          ><strong>创建文档</strong>
          </v-btn>

        </div>
      </template>
      <template v-slot:no-data>
        <div></div>
      </template>
      <template v-slot:no-results>
        <div style="text-align: center;">
          <img src="../../../assets/search.png" height="150px" width="150px"/>
        </div>
        <div style="font-size:20px;font-weight: bold">
          没有找到数据
        </div>
      </template>
      <template v-slot:[`item.name`] = "{item}" >
        <v-icon>mdi-text-box-outline</v-icon>
        <a @click="enterPad(item.token)" style="position:relative;left:2%;top:3%;">{{ item.name }}</a>
      </template>
      <template v-slot:expanded-item="{ headers, item }">
        <td :colspan="headers.length">
          {{ item.info }}
        </td>
      </template>
      <template v-slot:[`item.creatorName`]="{ item }">
        <v-chip class="accent-1" color="white">
          <v-avatar left><v-img :src="getIdenticon(item.creatorName, 50, 'user')" ></v-img></v-avatar>
          {{ item.creatorName }}
        </v-chip>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon @click="favorPad(item)" v-if="!isInFavor(item)" class="mr-2">mdi-star-outline</v-icon>
        <v-icon @click="unFavorPad(item)" v-else class="mr-2">mdi-star</v-icon>
        <v-icon @click="openShowConfirmDelete(item.token)" class="mr-2">mdi-delete-outline</v-icon>
      </template>
    </v-data-table>

    <v-dialog v-model="dialog1" hide-overlay width="1000px">
      <v-card
          width="1000px"
          style="position: relative;"
      >

        <v-card-title>
          共享文档
        </v-card-title>

        <v-icon @click="dialog1 = false" style="position: absolute;right: 1%;top:1%;">mdi-close</v-icon>

        <v-card-text>
          <v-form>
            <v-text-field
                v-model="newDocumentForm.name"
                :counter="10"
                label="文档名称"
                style="width:80%;position: relative;left:2%"
                prepend-icon="mdi-map-marker"
            ></v-text-field>
            <v-textarea
                v-model="newDocumentForm.intro"
                label="文档简介"
                style="width:80%;position: relative;left:2%"
                prepend-icon="mdi-comment"
            ></v-textarea>
          </v-form>
        </v-card-text>
        <v-card-actions style="position:absolute;right:0%;bottom: 0%;">
          <v-btn
              text
              color="primary"
              width="70px"
              style="float: right"
              @click="dialog1 = false;"
          >取消</v-btn>
          <v-btn
              text
              :color="getTopicColor(user.topic)"
              width="70px"
              style="float: right"
              @click="checkNameIntro()"
          >确认</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialog2" hide-overlay width="1000px" style="height: 600px;">
      <v-card
          width="1000px"
          height="600px"
          style="position: relative;"
      >

        <v-card-title>
          共享文档
        </v-card-title>

        <v-card-actions class="justify-end" style="position: absolute;bottom: 0%;right: 0%;">
          <v-btn
              text
              color="primary"
              width="70px"
              style="float: right"
              @click="dialog1 = true;"
          >返回</v-btn>
          <v-btn
              text
              :color="getTopicColor(user.topic)"
              width="70px"
              style="float: right"
              @click="dialog1 = false;createPad()"
          >确认</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="editDialog1" hide-overlay width="1000px">
      <v-card
          width="1000px"
          style="position: relative;"
      >

        <v-card-title>
          共享文档
        </v-card-title>

        <v-icon @click="editDialog1 = false" style="position: absolute;right: 1%;top:1%;">mdi-close</v-icon>

        <v-card-text>
          <v-form>
            <v-text-field
                v-model="editDocumentForm.name"
                :counter="10"
                label="文档名称"
                style="width:80%;position: relative;left:2%"
                prepend-icon="mdi-map-marker"
            ></v-text-field>
            <v-textarea
                v-model="newDocumentForm.intro"
                label="文档简介"
                style="width:80%;position: relative;left:2%"
                prepend-icon="mdi-comment"
            ></v-textarea>
          </v-form>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-dialog v-model="editDialog2" hide-overlay width="1000px" style="height: 600px;">
      <v-card
          width="1000px"
          height="600px"
          style="position: relative;"
      >

        <v-card-title>
          共享文档
        </v-card-title>

        <v-card-actions class="justify-end" style="position: absolute;bottom: 0%;right: 0%;">
          <v-btn
              text
              color="primary"
              width="70px"
              style="float: right"
              @click="editDialog2 = false;editDialog1 = true;"
          >返回</v-btn>
          <v-btn
              text
              :color="getTopicColor(user.topic)"
              width="70px"
              style="float: right"
              @click="editDialog2 = false;editDialog1 = false"
          >确认</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="showConfirmDelete" hide-overlay max-width="300" persistent>
      <v-card>
        <v-card-title> 确认删除文档 </v-card-title>
        <v-card-text> 删除后，无法撤销操作！ </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green" class="white--text" @click="closeShowConfirmDelete()">
            取消
          </v-btn>
          <v-btn color="red" class="white--text" @click="deletePad()">
            确定
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </v-card>
</template>

<script>
import {showPersonList, userDocList, userCollectDocList, addDocToCollect, delDocFromCollect, userCreateDoc,
  userEditDocOther, userDelDoc, userEditDocContent, userReleaseDocLock, userGetDocLock, isDocLocked} from "@/api/user"
import getIdenticon from "@/utils/identicon";
import topicSetting from "@/utils/topic-setting";
import Cookies from "js-cookie"
import axios from "axios";

export default {
  inject: {'user': {default: null},
    'selectedProj': {default: null},
  },
  data() {
    return {
      showCollect: false,
      showConfirmDelete: false,
      selectedPad: null,
      doc: '',
      textList: {},
      expanded: [],
      addGroup: [1],
      deleteGroup: [1],
      deleteFlag: false,
      search: '',
      dialog1: false,
      dialog2: false,
      editDialog1: false,
      editDialog2: false,
      value: '',
      tab: '',
      searchNot: '',
      isCollectArr: [],
      item: [],
      headers: [
        {
          text: "文档名称",
          sortable: false,
          value: "name",
        },
        {text: "", value: "space", sortable: false},
        {text: "", value: "space", sortable: false},
        {text: "创建者", value: "creatorName", sortable: false},
        {text: "", value: "space", sortable: false},
        {text: "", value: "actions", sortable: false},
        {text: "", value: "data-table-expand"},
      ],
      html: "",
      blogInfo: {
        blogMdContent: '',
        blogContent: ''
      },
      documentData: [
        {
          creatorId: 12,
          creatorName: "test2",
          info: "test",
          name: "2",
          token: "117167294241403835"
        }
      ],
      peopleCanWrite: [
        {
          'name': 'szx',
          'id': 1,
          'email': '123'
        },
        {
          'name': 'ghy',
          'id': 2,
          'email': '123'
        },
      ],
      peopleCanNotWrite: [
        {
          'name': 'lr',
          'id': 3,
          'email': '123'
        },
        {
          'name': 'zhy',
          'id': 4,
          'email': '123'
        },
      ],
      allPeople: [],
      newDocumentForm: {
        name: '',
        intro: '',
        people: [],
      },
      editDocumentForm: {
        name: '',
        intro: '',
        people: [],
        id: "",
      },
      toolbars: {
        bold: true, // 粗体
        italic: true, // 斜体
        header: true, // 标题
        underline: true, // 下划线
        strikethrough: true, // 中划线
        mark: true, // 标记
        superscript: true, // 上角标
        subscript: true, // 下角标
        quote: true, // 引用
        ol: true, // 有序列表
        ul: true, // 无序列表
        link: true, // 链接
        imagelink: true, // 图片链接
        code: false, // code
        table: true, // 表格
        fullscreen: true, // 全屏编辑
        readmodel: true, // 沉浸式阅读
        htmlcode: true, // 展示html源码
        help: true, // 帮助
        /* 1.3.5 */
        undo: true, // 上一步
        redo: true, // 下一步
        trash: true, // 清空
        save: true, // 保存（触发events中的save事件）
        /* 1.4.2 */
        navigation: true, // 导航目录
        /* 2.1.8 */
        alignleft: true, // 左对齐
        aligncenter: true, // 居中
        alignright: true, // 右对齐
        /* 2.2.1 */
        subfield: true, // 单双栏模式
        preview: true, // 预览
      },
      collectDocList: [],
    }
  },
  created() {
    this.getPad()
    this.getFavorPads()
    showPersonList({projectId: this.selectedProj.projectId, userId: this.user.id}).then(
        res => {
          console.log(res);
          this.allPeople = res['data']['data'];
          console.log("allPeople");
          console.log(this.allPeople);
        }
    );
  },
  methods: {
    getIdenticon,
    enterPad(token) {
      const apiUrl = this.$router.resolve({path: "/api/pad/enterPad"}).href
      axios.post(apiUrl, {
        userId: this.user.id,
        token: token
      }).then((res) => {
        if (res.data.errcode === 0) {
          window.open(res.data.data.padUrl, '_blank');
        }
      })
    },
    gotoCollect() {
      this.isCollect = true;
      this.getFavorPads();
    },
    gotoAll() {
      this.isCollect = false;
      this.getPad()
    },
    checkNameIntro() {
      console.log("check");
      if (this.newDocumentForm.name.trim() === "") {
        this.$message({
          type: "error",
          message: "共享文档名称不能为空！",
        });
      } else {
        if (this.documentData === undefined) {
          this.dialog1 = false;
          return
        }
        for (let i = 0; i < this.documentData.length; i++) {
          if (this.documentData[i].name.trim() === this.newDocumentForm.name.trim()) {
            this.$message({
              type: 'error',
              message: '已存在同名文档！'
            })
            return;
          }
        }
        this.dialog1 = false;
        this.createPad()
      }
    },

    isInFavor(item) {
      return this.collectDocList.find((file) => {
        return file.token === item.token
      })
    },
    favorPad(item) {
      const apiUrl = this.$router.resolve({path: "/api/pad/favorPad"}).href
      axios.post(apiUrl, {
        userId: this.user.id,
        token: item.token
      }).then((res) => {
        if (res.data.errcode === 0) {
          this.$message.success("收藏成功！")
          this.collectDocList.push(item)
        } else {
          this.$message.error("收藏失败")
        }
      })
    },
    unFavorPad(item) {
      const apiUrl = this.$router.resolve({path: "/api/pad/unFavorPad"}).href
      axios.post(apiUrl, {
        userId: this.user.id,
        token: item.token
      }).then((res) => {
        if (res.data.errcode === 0) {
          this.$message.success("取消收藏成功！")
          let i = 0;
          for (i in this.collectDocList) {
            console.log(i)
            if (this.collectDocList[i].name === item.name) {
              break;
            }
          }
          this.collectDocList.splice(i, 1)
          if (this.isCollect) {
            this.getFavorPads()
          }
        } else {
          this.$message.error("取消收藏失败")
        }
      })
    },
    getFavorPads() {
      const apiUrl = this.$router.resolve({path: "/api/pad/getFavorPads"}).href
      axios.post(apiUrl, {
        userId: this.user.id,
        projectId: this.selectedProj.projectId
      }).then((res) => {
        console.log("collect doct")
        this.collectDocList = res.data.data.pads
        console.log(this.collectDocList)
        // this.documentData = this.collectDocList
      })
    },

    createPad() {
      const apiUrl = this.$router.resolve({path: "/api/pad/createPad"}).href
      axios.post(apiUrl, {
        userId: this.user.id,
        projectId: this.selectedProj.projectId,
        name: this.newDocumentForm.name,
        info: this.newDocumentForm.intro,
      }).then((res) => {
        if (res.data.token !== null) {
          this.$message.success("创建文档成功！")
          this.getPad()
        } else {
          this.$message.error("创建文档失败")
        }
      })
    },

    getPad() {
      const apiUrl = this.$router.resolve({path: "/api/pad/getPads"}).href
      axios.post(apiUrl, {
        projectId: this.selectedProj.projectId
      }).then((res) => {
        console.log("document data")
        this.documentData = res.data.data.pads
        console.log(this.documentData)
      })
    },

    openShowConfirmDelete(token) {
      this.showConfirmDelete = true;
      this.selectedPad = token
    },

    closeShowConfirmDelete() {
      this.showConfirmDelete = false;
      this.selectedPad = null
    },
    deletePad() {
      const apiUrl = this.$router.resolve({path: "/api/pad/deletePad"}).href
      axios.post(apiUrl, {
        token: this.selectedPad,
        userId: this.user.id
      }).then((res) => {
        if (res.data.errcode === 0) {
          this.showConfirmDelete = false
          this.$message.success("删除文档成功！")
          //this.documentData = this.documentData.filter(doc => doc.token !== token);
          this.getPad()
          this.getFavorPads()
        } else if (res.data.errcode === 1) {
          this.$message.error("删除文档失败，您没有权限")
        } else {
          this.$message.error("删除失败，请联系管理员")
        }
      })

    },
    close() {
      this.$emit('close');
    },
    open() {
      this.$emit('open');
    },
    closeDialog() {
      console.log("closeDialog");
      this.dialog1 = false;
      this.dialog2 = false;
    },
    filterOnlyCapsText(value, search, item) {
      var s = item["name"];
      return (
          s != null &&
          search != null &&
          typeof s === "string" &&
          s.toString().toLocaleUpperCase().indexOf(search.toLocaleUpperCase()) !==
          -1
      );
    },
    getTopicColor: topicSetting.getColor,
    getDarkColor: topicSetting.getDarkColor,
    getLinearGradient: topicSetting.getLinearGradient,
    getRadialGradient: topicSetting.getRadialGradient,
    getUrl: topicSetting.getUrl
  },
}
</script>

<style>

</style>
