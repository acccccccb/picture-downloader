<template>
  <div>
    <div class="titlebar">
      <div class="title">{{config.name}} - {{config.version}}</div>
      <div class="titlebarBtnGroup">
        <el-button @click="top" :type="isTop?'danger':'primary'" size="mini" style="padding:4px 8px;"><span class="fas fa-thumbtack"></span></el-button>
        <el-button @click="hide" type="primary" size="mini" style="padding:4px 8px;"><span class="fas fa-angle-down"></span></el-button>
        <el-button @click="quit" type="primary" size="mini" style="padding:4px 8px;"><span class="fas fa-times"></span></el-button>
      </div>
    </div>
    <el-row :gutter="20" class="main">
      <el-col :span="24">
        <el-tabs v-model="activeName">
          <el-tab-pane label="图片下载器">
            <Spider></Spider>
          </el-tab-pane>
            <el-tab-pane label="关于">
                <About></About>
            </el-tab-pane>
        </el-tabs>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  let config = require('../../package.json');
  import Spider from './Index/Spider'
  import About from './About/About'
    export default {
      name: 'Index',
        components:{
            Spider,
            About,
        },
        data(){
          return {
              config:config,
              activeName:'0',
              isTop:false,
          }
        },
        created(){
            ipcRenderer.on('windowReplay', (event, arg) => {
                if(arg === 'isTop') {
                    this.isTop = true;
                }
                if(arg === 'isNotTop') {
                    this.isTop = false;
                }
            })
        },
        methods:{
          top:function(){
              ipcRenderer.send('window', 'top');
          },
          hide:function(){
              ipcRenderer.send('window', 'hide');
              let option = {
                  title: "最小化至托盘图标",
                  body: "应用程序已经最小化至托盘图标，双击即可显示。",
                  silent:true,
                  timeoutType:'normal',
              };
              // 创建通知并保存
              let Notification = new electron.remote.Notification(option);
              Notification.show();
          },
          quit:function(){
              ipcRenderer.send('window', 'quit');
          },
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .main {
    box-sizing: border-box;
    width:100%;
    padding:0;
    margin-left:0!important;
    margin-right:0!important;
  }
  .titlebar {
    box-sizing: border-box;
    width:100%;
    height:34px;
    background:dodgerblue;
    color:#fff;
    line-height:34px;
    padding-left:10px;
    padding-right:10px;

  }
  .titlebar .title {
    float:left;
    text-shadow: 1px 1px rgba(0,0,0,.3);
    -webkit-app-region: drag;
    -webkit-user-select: none;
    width:650px;
  }
  .titlebar .titlebarBtnGroup {
    float:right;
  }
</style>
