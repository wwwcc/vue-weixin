<template>
 <!-- 进入对话框里的页面 -->
    <div class="dialogue">
        <!-- 头部 -->
        <header id="wx-header">
            <!-- 进入对话框将右上角的图标显示 -->
            <div class="other">
                <!-- 进入群对话框的页面，即右上角的群图标 -->
                <router-link :to="{path:'/wechat/dialogue/dialogue-info',query: { msgInfo: msgInfo}}" tag="span" class="iconfont icon-chat-group" v-show="$route.query.group_num&&$route.query.group_num!=1"></router-link>
                <!-- 进入个人对话框的页面路径，即右上角的单人图标 -->
                <router-link :to="{path:'/wechat/dialogue/dialogue-detail',query: { msgInfo: msgInfo}}" tag="span" class="iconfont icon-chat-friends" v-show="$route.query.group_num==1"></router-link>
            </div>
            <div class="center">
                <router-link to="/" tag="div" class="iconfont icon-return-arrow">
                    <span>微信</span>
                </router-link>
                <span>{{pageName}}</span>
                <!-- 显示群里的人数 -->
                <span class="parentheses" v-show='$route.query.group_num&&$route.query.group_num!=1'>{{$route.query.group_num}}</span>
            </div>
        </header>
        <!-- section显示的是进入对话页面的中间部分，即把对话内容显示出来 -->
        <section class="dialogue-section clearfix" v-on:click="MenuOutsideClick">
            <div class="row clearfix" v-for="item in msgInfo.msg">
                <!-- 对话的人头像 -->
                <img :src="item.headerUrl" class="header">
                <!-- 对话人的内容 -->
                <p class="text" v-more>{{item.text}}</p>
            </div>
            <!-- 手机端长按出现的功能 -->
            <span class="msg-more" id="msg-more">
                <ul>
                    <li>复制</li>
                    <li>转发</li>
                    <li>收藏</li>
                    <li>删除</li>
                </ul>
            </span>
        </section>
        <footer class="dialogue-footer">
            <div class="component-dialogue-bar-person">
                <!-- 键盘打字的功能 -->
                <span class="iconfont icon-dialogue-jianpan" v-show="!currentChatWay" v-on:click="currentChatWay=true"></span>
                <!-- 控制低端显示点击会转换成按住说话的那个图标 -->
                <span class="iconfont icon-dialogue-voice" v-show="currentChatWay" v-on:click="currentChatWay=false"></span>
                <div class="chat-way" v-show="!currentChatWay">
                    <div class="chat-say" v-press>
                        <span class="one">按住 说话</span>
                        <span class="two">松开 结束</span>
                    </div>
                </div>
                <div class="chat-way" v-show="currentChatWay">
                    <!-- 中间的像input框的那个输入框 -->
                    <input class="chat-txt" type="text" v-on:focus="focusIpt" v-on:blur="blurIpt"/>
                </div>
                <!-- 右边的笑脸，表情 -->
                <span class="expression iconfont icon-dialogue-smile"></span>
                <!-- 最右边的加号，（即点击加号出现相册，拍摄，视频聊天等功能的操作页面 ）-->
                <span class="more iconfont icon-dialogue-jia"></span>
                <!-- 控制手机端的消息记录 -->
                <div class="recording" style="display: none;" id="recording">
                    <div class="recording-voice" style="display: none;" id="recording-voice">
                        <div class="voice-inner">
                            <div class="voice-icon"></div>
                            <div class="voice-volume">
                                <span></span>
                                <span></span>
                                <span></span>
                                <span></span>
                                <span></span>
                                <span></span>
                                <span></span>
                                <span></span>
                                <span></span>
                            </div>
                        </div>
                        <!-- 将输入框切换到按住说话时弹出的框 -->
                        <p>手指上划,取消发送</p>
                    </div>
                    <!-- 弹出上面的框之后，手指上划会出现下面这个弹出框 -->
                    <div class="recording-cancel" style="display: none;">
                        <div class="cancel-inner"></div>
                        <p>松开手指,取消发送</p>
                    </div>
                </div>
            </div>
        </footer>
    </div>
</template>
<script>
export default {
  data() {
    return {
      pageName: this.$route.query.name,
      currentChatWay: true, //ture为键盘打字 false为语音输入
      timer: null
      // sayActive: false // false 键盘打字 true 语音输入
    };
  },
  beforeRouteEnter(to, from, next) {
    next(vm => {
      vm.$store.commit("setPageName", vm.$route.query.name);
    });
  },
  computed: {
    msgInfo() {
      for (var i in this.$store.state.msgList.baseMsg) {
        if (this.$store.state.msgList.baseMsg[i].mid == this.$route.query.mid) {
          return this.$store.state.msgList.baseMsg[i];
        }
      }
    }
  },
//   注册局部指令
  directives: {
    //   按压下“按住说话这个框”话语时绑定的事件
    press: {
      inserted(element, binding) {
        var recording = document.querySelector(".recording"),
          recordingVoice = document.querySelector(".recording-voice"),
          recordingCancel = document.querySelector(".recording-cancel"),
          startTx,
          startTy;

        element.addEventListener(
          "touchstart",
          function(e) {
            // 用bind时，vue还没插入到dom,故dom获取为 undefine，用 inserted 代替 bind,也可以开个0秒的定时器
            element.className = "chat-say say-active";
            recording.style.display = recordingVoice.style.display = "block";
            // console.log('start')
            var touches = e.touches[0];
            startTx = touches.clientX;
            startTy = touches.clientY;
            e.preventDefault();
          },
          false
        );
        element.addEventListener(
          "touchend",
          function(e) {
            /*var touches = e.changedTouches[0];
                        var distanceY = startTy - touches.clientY;
                        if (distanceY > 50) {
                            console.log("取消发送信息");
                        }else{
                            console.log("发送信息");
                        }*/

            element.className = "chat-say";
            recordingCancel.style.display = recording.style.display = recordingVoice.style.display =
              "none";
            // console.log('end')
            e.preventDefault();
          },
          false
        );
        element.addEventListener(
          "touchmove",
          function(e) {
            var touches = e.changedTouches[0],
              endTx = touches.clientX,
              endTy = touches.clientY,
              distanceX = startTx - endTx,
              distanceY = startTy - endTy;

            if (distanceY > 50) {
              element.className = "chat-say";
              recordingVoice.style.display = "none";
              recordingCancel.style.display = "block";
            } else {
              element.className = "chat-say say-active";
              recordingVoice.style.display = "block";
              recordingCancel.style.display = "none";
            }
            // 阻断事件冒泡 防止页面被一同向上滑动
            e.preventDefault();
          },
          false
        );
      }
    },
    more: {
      bind(element, binding) {
        var startTx, startTy;
        element.addEventListener(
          "touchstart",
          function(e) {
            var msgMore = document.getElementById("msg-more"),
              touches = e.touches[0];
            startTx = touches.clientX;
            startTy = touches.clientY;

            clearTimeout(this.timer);
            this.timer = setTimeout(() => {
              // 控制菜单的位置
              msgMore.style.left =
                (startTx - 18 > 180 ? 180 : startTx - 18) + "px";
              msgMore.style.top = element.offsetTop - 33 + "px";
              msgMore.style.display = "block";
              element.style.backgroundColor = "#e5e5e5";
            }, 500);
          },
          false
        );
        element.addEventListener(
          "touchmove",
          function(e) {
            var touches = e.changedTouches[0],
              disY = touches.clientY;
            if (Math.abs(disY - startTy) > 10) {
              clearTimeout(this.timer);
            }
          },
          false
        );
        element.addEventListener(
          "touchend",
          function(e) {
            clearTimeout(this.timer);
          },
          false
        );
      }
    }
  },
  methods: {
    // 解决输入法被激活时 底部输入框被遮住问题
    focusIpt() {
      this.timer = setInterval(function() {
        document.body.scrollTop = document.body.scrollHeight;
      }, 100);
    },
    blurIpt() {
      clearInterval(this.timer);
    },
    // 点击空白区域，菜单被隐藏
    MenuOutsideClick(e) {
      var container = document.querySelectorAll(".text"),
        msgMore = document.getElementById("msg-more");
      if (e.target.className === "text") {
      } else {
        msgMore.style.display = "none";
        container.forEach(item => (item.style.backgroundColor = "#fff"));
      }
    }
  }
};
</script>
<style>
@import "../../assets/css/dialogue.css";
.say-active {
  background: #c6c7ca;
}
</style>