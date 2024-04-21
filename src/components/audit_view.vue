<template>
    <iframe
        ref="showDiffHtml"
        src="/diff2html_ui.html"
        width="100%"
        height="100%"
        @load="sendDataToIframe"
    ></iframe>
</template>

<script>
import Cookies from "js-cookie";
export default {
  name: "audit_view",
  data() {
    return {
      selectedRepo:{},
      pr:{},
      repo:'',
      fromBranch:'',
      toBranch:'main'
    }
  },
  methods:{
    sendDataToIframe(){
      const diffString = Cookies.get('diffString');
      // console.log();
      this.$refs.
      showDiffHtml.contentWindow.postMessage(
          diffString);
      Cookies.delete('diffString');

    }
  }
}
</script>

<style scoped>

</style>