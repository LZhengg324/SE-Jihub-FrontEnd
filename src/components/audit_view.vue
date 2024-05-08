<template>
  <div style="display: flex; flex-direction: row; height: 100vh; ">
    <div style="height: 100vh; flex: 7">
      <iframe
          ref="showDiffHtml"
          src="/diff2html_ui.html"
          :style="{ 'min-width': '100%', 'min-height': '100%' }"
          @load="sendDataToIframe"
      ></iframe>
    </div>
    <div style="height: 80vh; flex: 3">
      <h3>审核意见: {{this.prtitle}}</h3>
      <p1>
        {{this.prdescription}}
      </p1>
      <el-form :model="form" ref="form">
        <el-form-item >
          <el-input v-model="form.comment" type="textarea" :disabled="PrToAudit_ReadOnly"></el-input>
        </el-form-item>
      </el-form>
      <el-button v-if="!PrToAudit_ReadOnly"  @click="turnBack" >取 消</el-button>
      <el-button v-if="!PrToAudit_ReadOnly" type="primary" @click="createNewComment()">不通过</el-button>
      <el-button v-if="!PrToAudit_ReadOnly" type="primary" @click="approvePr()">通过</el-button>
    </div>
  </div>

</template>


<script>
import Cookies from "js-cookie";
import {approvePullRequest, closePullRequest} from "@/api/user";

export default {
  name: "audit_view",
  data() {
    return {
      form: {
        name: "",
        comment: '',
      },
      PrToAudit: {},
      selectedRepo:{},
      PrToAudit_ReadOnly: false,
      prtitle:"",
      prdescription:"",
    }
  },
  inject: {
    user: {default: null},
    proj: {default: null},

  },
  mounted() {
  },
  created() {
    this.PrToAudit_ReadOnly = this.user.id !== this.proj.managerId ;
    this.form.comment = localStorage.getItem('comment');
    this.prtitle = localStorage.getItem('title');
    this.prdescription = localStorage.getItem("description");
    if(!this.PrToAudit_ReadOnly)
      this.PrToAudit = JSON.parse(Cookies.get("PrToAudit"));
    this.selectedRepo = JSON.parse(Cookies.get("selectedRepo"));

    if(!this.PrToAudit.isOpen)
      this.PrToAudit_ReadOnly = true;
  },
  methods: {
    sendDataToIframe() {
      const diffString = localStorage.getItem('diffString');
      // const diffString = Cookies.get('diffString');
      console.log("cbycby" + diffString);
      this.$refs.showDiffHtml.contentWindow.postMessage(
          diffString);
    },
    turnBack() {
      this.$router.back();
    },
    createNewComment() {
      closePullRequest({
        ghpr_id: this.PrToAudit.id,
        repo_id: this.selectedRepo.id,
        comment: this.form.comment,
      }).then((res) => {
        alert('您拒绝了一个PullRequest并提出修改意见！');
      }).catch((err) => {
        alert('哦不，好像失败了！');
      })
    },
    approvePr() {
      approvePullRequest({
        ghpr_id: this.PrToAudit.id,
        repo_id: this.selectedRepo.id,
        comment: this.form.comment,
      }).then((res) => {
        alert('您同意了一个PullRequest');
      }).catch((err) => {
        alert('哦不，好像失败了！');
      })
    },

  }
}
</script>


<style scoped>

</style>
