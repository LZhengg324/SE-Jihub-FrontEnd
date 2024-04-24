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
      <el-form  :model="form" ref="form">
        <el-form-item label="审核意见：">
          <el-input v-model="form.comment" type="textarea"></el-input>
        </el-form-item>
      </el-form>
      <el-button @click="turnBack">取 消</el-button>
      <el-button type="primary" @click="createNewComment">确 定</el-button>
    </div>
  </div>

</template>


<script>
import Cookies from "js-cookie";

export default {
  name: "audit_view",
  data() {
    return {
      selectedRepo: {},
      pr: {},
      repo: '',
      fromBranch: '',
      toBranch: 'main',
      form:{
        name:"",
        comment:'',
      },

    }
  },
  mounted() {
  },
  methods: {
    sendDataToIframe() {
      const diffString = Cookies.get('diffString');
      this.$refs.showDiffHtml.contentWindow.postMessage(
          diffString);
      Cookies.delete('diffString');
    },
    turnBack(){
      this.$router.back();
    },
    createNewComment(){

    }
  }
}
</script>


<style scoped>

</style>
