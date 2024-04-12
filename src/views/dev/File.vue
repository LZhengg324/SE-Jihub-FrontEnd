<script>
import axios from "axios";
import Cookies from 'js-cookie'
import * as CodeMirror from 'codemirror/lib/codemirror.js'
import "@/utils/cm-settings.js"
import topicSetting from "@/utils/topic-setting";
import {downloadStrAsFile} from "@/utils/file-system";

export default {
    name: "FileView",
    data() {
        return {
            tree: [],
            files: {
                html: 'mdi-language-html5',
                js: 'mdi-nodejs',
                json: 'mdi-code-json',
                md: 'mdi-language-markdown',
                pdf: 'mdi-file-pdf',
                png: 'mdi-file-image',
                txt: 'mdi-file-document-outline',
                xls: 'mdi-file-excel',
                vue: 'mdi-vuejs'
            },
            items: [],
            cmEditor: null,
            sheet: false,
            fileContent: '选择一个文件以查看其内容\n\n选择文件后，可以点击右上角”代码助手“，使用JiHub进行AI代码诊断、生成单元测试、下载文件',
            selectedText: '',
            fileTreeReady: false,
            fileContentReady: false,
        }
    },
    inject: {
        user: {default: null},
        proj: {default: null},
        repo: {default: null},
        branchName: {default: null}
    },
    methods: {
        downloadStrAsFile,
        getFileExt(obj, path) {
            if (obj['children'] === undefined) {
                obj['file'] = 'unknown'
                obj['path'] = path + '/' + obj['name']
                if (obj['name'].indexOf('.') !== -1) {
                    obj['file'] = obj['name'].split('.').pop()
                }
            } else {
                for (let i = 0; i < obj['children'].length; i++) {
                    this.getFileExt(obj['children'][i], path + '/' + obj['name'])
                }
            }
        },
        file2style() {
          if (this.tree.length === 0) {
            return 'null'
          }

          let file = this.tree[0]['file']
          if (file === 'html') {
            return 'htmlmixed'
          } else if (file === 'js') {
            return 'javascript'
          } else if (file === 'json') {
            return 'javascript'
          } else if (file === 'md') {
            return 'markdown'
          } else if (file === 'vue') {
            return 'vue'
          } else if (file === 'css') {
            return 'css'
          } else if (file === 'xml') {
            return 'xml'
          } else if (file === 'c' || file === 'cpp' || file === 'h') {
            return 'clike'
          } else if (file === 'py') {
            return 'python'
          } else {
            return 'null'
          }
        },
        onCursorActivity() {
          this.selectedText = this.cmEditor.getSelection()
        },
        diagSelected() {
          //如果文件长度大于Cookie最长长度，就不诊断了
          if (this.selectedText.length > 4096) {
            this.$message({
              type: 'error',
              message: '文件太长了，AI会罢工的！'
            })
            return
          }
          if (this.selectedText.length <= 10) {
            this.$message({
              type: 'error',
              message: '代码这么短，怎么诊断嘛'
            })
            return
          }
          Cookies.set('diag', this.selectedText)
          window.open('/user/ai/diagnosis', '_blank')
        },
        diagWholeFile() {
          //如果文件长度大于Cookie最长长度，就不诊断了
          if (this.fileContent.length > 4096) {
            this.$message({
              type: 'error',
              message: '文件太长了，AI会罢工的！'
            })
            return
          }
          Cookies.set('diag', this.fileContent)
          window.open('/user/ai/diagnosis', '_blank')
        },
        unitTestSelected() {
          //如果文件长度大于Cookie最长长度，就不诊断了
          if (this.selectedText.length > 4096) {
            this.$message({
              type: 'error',
              message: '文件太长了，AI会罢工的！'
            })
            return
          }
          if (this.selectedText.length <= 10) {
            this.$message({
              type: 'error',
              message: '代码这么短，怎么生成嘛'
            })
            return
          }
          Cookies.set('diag', this.selectedText)
          window.open('/user/ai/testdata', '_blank')
        },
        unitTestWholeFile() {
          //如果文件长度大于Cookie最长长度，就不诊断了
          if (this.fileContent.length > 4096) {
            this.$message({
              type: 'error',
              message: '文件太长了，AI会罢工的！'
            })
            return
          }
          Cookies.set('diag', this.fileContent)
          window.open('/user/ai/testdata', '_blank')
        },
        getTopicColor: topicSetting.getDarkColor,
        getRadialGradient: topicSetting.getRadialGradient,
        getLinearGradient: topicSetting.getLinearGradient,
    },
    created() {
        this.fileTreeReady = false;
        axios.post('/api/develop/getFileTree', {
            userId: this.user.id,
            projectId: this.proj.projectId,
            repoId: this.$route.params.repoid,
            branch: this.branchName
        }).then((res) => {
            if (res.data.errcode === 0) {
                console.log(res.data.data)
                this.items = res.data.data
                for (let i = 0; i < this.items.length; i++) {
                    this.getFileExt(this.items[i], '')
                }
            } else {
                alert('/api/develop/getFileTree errcode not 0: ' + res.data.message)
            }
        }).catch((err) => {
            alert('/api/develop/getFileTree error' + err)
            console.log(err);
        }).finally(() => {
            this.fileTreeReady = true;
        })
    },
    watch: {
        tree() {
            console.log('selected file change!')
            if (this.tree[0]['file'] !== undefined) {
                this.fileContentReady = false;
                this.cmEditor.setValue('正在努力拉取文件！\n\n选择文件后，可以点击右上角”代码助手“，使用JiHub进行AI代码诊断、生成单元测试、下载文件')
                this.cmEditor.setOption('mode', '')
                axios.post('/api/develop/getContent', {
                    userId: this.user.id,
                    projectId: this.proj.projectId,
                    repoId: this.$route.params.repoid,
                    branch: this.branchName,
                    path: this.tree[0]['path']
                }).then((res) => {
                    if (res.data.errcode === 0) {
                        console.log(res.data.data)
                        this.fileContent = res.data.data
                        this.cmEditor.setValue(this.fileContent)
                        this.cmEditor.setOption('mode', this.file2style())
                    } else {
                        alert('/api/develop/getFileContent errcode not 0: ' + res.data.message)
                    }
                }).catch((err) => {
                    alert('/api/develop/getFileContent error' + err)
                    console.log(err);
                }).finally(() => {
                    this.fileContentReady = true;
                })
            }
        }
    },
    mounted() {
      this.cmEditor = CodeMirror.fromTextArea(this.$refs.cm1, {
        theme: 'idea',
        lineNumbers: true,
        line: true,
        readOnly: true,
        lineWrapping: true
      });

      this.cmEditor.on('cursorActivity', this.onCursorActivity)
    }
}

</script>

<template>
  <v-container>
      <v-row>
          <v-col :cols="fileContentReady ? 2 : 3">
              <h2>文件树</h2>
              <v-card :style="getLinearGradient(user.topic)" min-height="calc(100vh - 300px)" max-height="calc(100vh - 300px)" class="overflow-y-auto">
                <v-treeview
                    v-if="fileTreeReady"
                    :items="items"
                    activatable
                    :active.sync="tree"
                    item-key="name"
                    open-on-click
                    dense
                    return-object
                >
                  <template v-slot:prepend="{ item, open }">
                    <v-icon v-if="!item.file" :color="getTopicColor(user.topic)">
                      {{ open ? 'mdi-folder-open' : 'mdi-folder' }}
                    </v-icon>
                    <v-icon v-else :color="getTopicColor(user.topic)">
                      {{ files[item.file] !== undefined ? files[item.file] : 'mdi-file-document' }}
                    </v-icon>
                  </template>
                </v-treeview>
                <v-skeleton-loader
                    v-else
                    type="list-item-three-line@5"
                    class="mt-2"
                ></v-skeleton-loader>
              </v-card>
          </v-col>
          <v-col :cols="fileContentReady ? 7 : 9">
            <h2>
              <v-scroll-y-transition>
                <span>
                  {{
                    !fileTreeReady ? '文件树加载中，稍侯...' :
                        tree.length === 0 ? '请选择一个文件' :
                            !fileContentReady ? '正在努力拉取文件' : ''
                  }}
                </span>
              </v-scroll-y-transition>
              <v-scroll-y-transition>
                <a v-ripple
                   v-if="fileContentReady"
                   :style="'text-decoration: none; color: ' + getTopicColor(user.topic)"
                   :href="'https://github.com/' + repo.user + '/' + repo.repo + '/blob/' + branchName + '/' + tree[0]['path']"
                   target="_blank">
                  {{
                    !fileTreeReady ? '文件树加载中，稍侯...' :
                        tree.length === 0 ? '请选择一个文件' :
                            !fileContentReady ? '正在努力拉取文件' :
                                repo.repo + tree[0]['path']
                  }}
                </a>
              </v-scroll-y-transition>
<!--              <a style="float: right" v-if="fileContentReady" @click="sheet = !sheet" :style="'color: ' + getTopicColor(user.topic)" v-ripple>代码助手</a>-->
            </h2>

              <v-card max-height="calc(100vh - 300px)" min-height="calc(100vh - 300px)">
                <textarea ref="cm1" v-model='fileContent' style="height: calc(100vh - 300px); width: 100%"></textarea>
              </v-card>
          </v-col>

          <v-col cols="3" v-if="fileContentReady">
            <h2 :style="'text-decoration: none; color: ' + getTopicColor(user.topic)">代码助手</h2>
            <v-card max-height="calc(100vh - 300px)" min-height="calc(100vh - 300px)" class="overflow-y-auto overflow-x-hidden">
              <v-card-title :style="getLinearGradient(user.topic)"><strong>欢迎来到代码助手</strong></v-card-title>
              <v-divider></v-divider>
              <v-card-title>单元测试</v-card-title>
              <v-card-text>JiHub可以对您选中的代码，或是整个文件生成单元测试</v-card-text>
<!--              <v-card-title>代码助手</v-card-title>-->
<!--              <v-card-text>-->
<!--                <v-container fluid>-->
<!--                  <v-row>-->
<!--                    <v-spacer></v-spacer>-->
<!--                    <v-col cols="12" sm="12" md="10" lg="6" xl="6">-->
<!--                      <v-textarea label="选中的代码" outlined v-model="selectedText" class="need-mono" rows="20"></v-textarea>-->
<!--                    </v-col>-->
<!--                    <v-spacer></v-spacer>-->
<!--                  </v-row>-->
<!--                </v-container>-->
<!--              </v-card-text>-->
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn width="" outlined :color="getTopicColor(user.topic)" @click="unitTestSelected"><v-icon>mdi-check</v-icon>对选中代码</v-btn>
              </v-card-actions>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn width="" outlined :color="getTopicColor(user.topic)" @click="unitTestWholeFile"><v-icon>mdi-check</v-icon>对整个文件</v-btn>
              </v-card-actions>
              <v-divider></v-divider>
              <v-card-title>代码诊断</v-card-title>
              <v-card-text>如果您阅读此代码比较困难，JiHub也很乐意对您选中的代码，或是整个文件进行代码诊断</v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn width="" outlined :color="getTopicColor(user.topic)" @click="diagSelected"><v-icon>mdi-code-braces</v-icon>对选中代码</v-btn>
              </v-card-actions>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn width="" outlined :color="getTopicColor(user.topic)" @click="diagWholeFile"><v-icon>mdi-code-braces</v-icon>对整个文件</v-btn>
              </v-card-actions>
              <v-divider></v-divider>
              <v-card-text>或者如果您想在GitHub操作，或查看源文件？</v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn width="" outlined :color="getTopicColor(user.topic)" link :href="'https://github.com/' + repo.user + '/' + repo.repo + '/blob/' + branchName + '/' + tree[0]['path']" target="_blank"><v-icon>mdi-github</v-icon>在GitHub浏览</v-btn>
              </v-card-actions>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn width="" outlined :color="getTopicColor(user.topic)" @click="() => downloadStrAsFile(selectedText, user.name + '\'s-clip' + '-' + this.tree[0]['name'])"><v-icon>mdi-download</v-icon>下载选中代码</v-btn>
              </v-card-actions>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn width="" outlined :color="getTopicColor(user.topic)" @click="() => downloadStrAsFile(fileContent, this.tree[0]['name'])"><v-icon>mdi-download</v-icon>下载整个文件</v-btn>
              </v-card-actions>

              <v-row style="height: 5rem"></v-row>
            </v-card>
          </v-col>
      </v-row>
  </v-container>

</template>

<style>
.CodeMirror {
  height: calc(100vh - 300px);
}

.need-mono {
  font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>