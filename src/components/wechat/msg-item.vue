 <template>
    <!--消息列表组件 数据交互频繁--><!-- 微信消息列表 -->
    <!--进入 dialogue 页面，携带参数 mid(msg的id) name(是个人就是对方名字，是群组就是群名) group_num -->
    <li :class="{'item-hide':deleteMsg}">
        <!--自定义指令 v-swiper 用于对每个消息进行滑动处理-->
        <router-link :to="{ path: '/wechat/dialogue', query: { mid: item.mid,name:item.group_name||(item.user[0].remark||item.user[0].nickname),group_num:item.user.length}}" tag="div" class="list-info" v-swiper v-on:click.native="toggleMsgRead($event,'enter')">
            <div class="header-box">
                <!--未读并且未屏蔽 才显示新信息数量,一個對話的新消息泡泡-->
                <i class="new-msg-count" v-show="!read&&!item.quiet">{{item.msg.length}}</i>
                <!--未读并且屏蔽 只显示小红点-->
                <i class="new-msg-dot" v-show="!read&&item.quiet"></i>
                <!--如果是私聊，只显示一个头像； 如果是群聊，则显示多个头像，flex 控制样式-->
                <!-- 判断是群还是个人，然后显示图像 -->
                <div class="header" :class="[item.type=='group'?'multi-header':'']">
                    <!-- user追踪到store.js中的user,然后在getUserInfo方法，之后又到contact.js中的这个方法 -->
                    <img v-for="userInfo in item.user" :src="userInfo.headerUrl" :key="userInfo.wxid">
                </div>
            </div>
            <div class="desc-box">
                <!--使用过滤器 fmtDate 格式化时间-->
                <div class="desc-time">{{item.msg[item.msg.length-1].date | fmtDate('hh:ss')}}</div>
                <div class="desc-author" v-if="item.type=='group'">{{item.group_name}}</div>
                <!--如果没有备注（remark）好友，则显示微信昵称-->
                <div class="desc-author" v-else>{{item.user[0].remark||item.user[0].nickname}}</div>
                 <!-- 在页面上显示对话最后对话的结束消息，不进入对话框里 -->
                <div class="desc-msg">
                    <!-- 显示是否消息免打扰 -->
                    <div class="desc-mute iconfont icon-mute" v-show="item.quiet">
                    </div>
                    <!-- 不进入对话框里，显示群里最后是谁在说话，即人名 -->
                    <span v-show="item.type=='group'">{{item.msg[item.msg.length-1].name}}:</span>
                    <!-- 将最后人说的话显示在页面上 -->
                    <span>{{item.msg[item.msg.length-1].text}}</span>
                </div>
            </div>
        </router-link>
        <!-- 手机端长按的滑动效果 -->
        <div class="operate-box">
            <div class="operate-unread" v-if="read" v-on:click="toggleMsgRead">标为未读</div>
            <div class="operate-read" v-else v-on:click="toggleMsgRead">标为已读</div>
            <div class="operate-del" v-on:click="deleteMsgEvent">删除</div>
        </div>
    </li>
</template>
<script>
    export default {
        props: ["item"],
        data() {
            return {
                read: this.item.read,
                deleteMsg: false
            }
        },
        methods: {
            //切换消息未读/已读状态
            toggleMsgRead(event, status) {
                if (status === 'enter') {
                    if (this.read) {
                        return ''
                    }
                    this.read = true
                } else {
                    this.read = !this.read
                }
                if (!this.item.quiet) {
                    if (this.read) {
                        this.$store.commit('minusNewMsg')
                    } else {
                        this.$store.commit('addNewMsg')
                    }
                }

                event.target.parentNode.parentNode.firstChild.style.marginLeft = 0 + "px"
            },
            deleteMsgEvent() {
                this.deleteMsg = true
                if (!this.item.quiet) {
                    if (!this.read) {
                        this.$store.commit('minusNewMsg')
                    }
                }
            }
        },
        // 参考 https://vuefe.cn/v2/guide/custom-directive.html
        directives: {
            swiper: {
                bind: function(element, binding) {
                    var isTouchMove, startTx, startTy
                    element.addEventListener('touchstart', function(e) {
                        var touches = e.touches[0]
                        startTx = touches.clientX
                        startTy = touches.clientY
                        isTouchMove = false;
                    }, false);
                    element.addEventListener('touchmove', function(e) {
                        var touches = e.changedTouches[0],
                            endTx = touches.clientX,
                            endTy = touches.clientY,
                            distanceX = startTx - endTx,
                            distanceY = startTy - endTy;
                        if (distanceX < 0) { //右滑
                            if (Math.abs(distanceX) >= Math.abs(distanceY)) {
                                if (Math.abs(distanceX) > 20) {
                                    element.style.transition = "0.3s"
                                    element.style.marginLeft = 0 + "px"
                                }
                            }
                        } else { //左滑
                            if (Math.abs(distanceX) >= Math.abs(distanceY)) {
                                if (distanceX < 156 && distanceX > 20) {
                                    e.preventDefault()
                                    element.style.transition = "0s"
                                    element.style.marginLeft = -distanceX + "px"
                                    isTouchMove = true
                                }
                            }
                        }
                        // e.preventDefault()
                    }, false);
                    element.addEventListener('touchend', function(e) {
                        if (!isTouchMove) {
                            return;
                        }
                        var touches = e.changedTouches[0],
                            endTx = touches.clientX,
                            endTy = touches.clientY,
                            distanceX = startTx - endTx,
                            distanceY = startTy - endTy,
                            isSwipe = false
                        if (Math.abs(distanceX) >= Math.abs(distanceY)) {
                            if (distanceX < 0) {
                                return;
                            }
                            if (Math.abs(distanceX) < 60) {
                                isSwipe = true
                                element.style.transition = "0.3s"
                                element.style.marginLeft = 0 + "px"
                            } else {
                                element.style.transition = "0.3s"
                                element.style.marginLeft = "-156px"
                            }
                        }
                    }, false);
                }
            }
        }
    }
   
</script>