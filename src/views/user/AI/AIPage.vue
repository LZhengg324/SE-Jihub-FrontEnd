<script>

import diagnosis from "@/views/user/AI/Diagnosis.vue";
import TestData from "@/views/user/AI/TestData.vue";
import Diagnosis from "@/views/user/AI/Diagnosis.vue";
import topicSetting from "@/utils/topic-setting";
import Cookies from "js-cookie";

export default {
  components: {TestData, Diagnosis},

  data() {
    return {
      selected: "diagnosis",
    }

  },

  inject: {
    user: {default: null},
    proj: {default: null}
  },

  mounted() {
    this.selectTabFromUrl()

  },

  methods: {
    selectTabFromUrl()
    {
      const tabHash = window.location.hash;

      if (tabHash === '#tabdiagnosis') {
        this.selected = "diagnosis"
      } else if (tabHash === '#tabtestData') {
        this.selected = "testData"
      }
    },
    // test() {
    //   window.open('/user/ai#tabdiagnosis', '_blank')
    // },
    getTopicColor: topicSetting.getColor,
    getDarkColor: topicSetting.getDarkColor,
    getRadialGradient: topicSetting.getRadialGradient
  }

}

</script>

<template>

  <v-card>
<!--    <v-btn @click="test()"> this </v-btn>-->
    <div>
        <v-tabs v-model="selected">
          <v-tab href="#diagnosis" class="icon-with-text">
            <v-icon :color="getDarkColor(user.topic)">mdi-atom-variant</v-icon>
            代码诊断
          </v-tab>

          <v-tab href="#testData" class="icon-with-text">
            <v-icon :color="getDarkColor(user.topic)">mdi-palette-outline</v-icon>
            生成测试数据
          </v-tab>
          </v-tabs>
    </div>
      <v-tabs-items v-model="selected">
        <v-tab-item value="diagnosis">
          <diagnosis/>
        </v-tab-item>
        <v-tab-item value="testData">
          <TestData/>
        </v-tab-item>
      </v-tabs-items>
  </v-card>



</template>

<style scoped>

.icon-with-text{
  display: flex;
}

.icon-with-text .v-icon{
  margin-right: 12px
}

</style>