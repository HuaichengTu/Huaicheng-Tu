# 生日聊天页面 🎂
## 页面代码
以下是聊天页面的核心HTML代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=100%, initial-scale=1">
    <title>与屠淮城聊天中</title>
    <link href="css/main.min.css" rel="stylesheet" type="text/css">
    <link rel="Shortcut Icon" href="favicon.ico"/>
    <link rel="preload" href="img/love-the-girl.jpg" as="image">
    <link rel="preload" href="img/news-wuhanfeiyan.jpg" as="image">
    <link rel="preload" href="img/separate.jpeg" as="image">
    <link rel="preload" href="img/in-sichuan.jpeg" as="image">
    <link rel="preload" href="img/lucky-me.jpg" as="image">
    <link rel="preload" href="img/breakfast.jpg" as="image">
    <link rel="preload" href="img/exercise-together.jpg" as="image">
    <link rel="preload" href="img/travel.jpg" as="image">
    <link rel="preload" href="img/foot.jpg" as="image">
    <link rel="preload" href="img/marry-me.png" as="image">
    <link rel="preload" href="img/kiss-my-princess.png" as="image">
    <!-- 新增：头像样式 + 修复文字显示不全问题 -->
    <style>
        /* 重置消息行布局，解决重合核心问题 */
        .msg-row {
            margin: 12px 0 !important; /* 增大上下间距，避免重合 */
            padding: 0 16px !important;
            display: flex !important; /* 统一所有消息行的Flex布局 */
            align-items: flex-start !important;
            width: 100% !important;
            box-sizing: border-box !important;
        }
        /* 屠淮城（对方）消息：左对齐，显示头像 */
        .msg-row.msg-author {
            justify-content: flex-start !important;
        }
        /* 自己的消息：右对齐，隐藏头像 */
        .msg-row.msg-me {
            justify-content: flex-end !important;
        }
        /* 头像样式（仿微信） */
        .msg-avatar {
            width: 40px;
            height: 40px;
            border-radius: 6px;
            margin-right: 8px;
            overflow: hidden;
            flex-shrink: 0;
            display: block;
        }
        .msg-avatar img {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        /* 自己的消息行隐藏头像占位 */
        .msg-row.msg-me .msg-avatar {
            display: none !important;
            margin-right: 0 !important;
        }
        /* 消息框样式：修复文字显示不全 + 兼容图片消息 */
        .msg {
            max-width: 70% !important;
            padding: 8px 12px !important;
            border-radius: 10px !important;
            word-wrap: break-word !important; /* 强制换行 */
            word-break: break-all !important; /* 兼容长单词/链接换行 */
            margin: 0 !important; /* 清除原有外边距，避免重合 */
            box-sizing: border-box !important;
            white-space: normal !important; /* 取消强制单行，允许换行 */
            overflow: visible !important; /* 内容超出时显示完整 */
            line-height: 1.4 !important; /* 优化行高，提升可读性 */
        }
        /* 图片消息专用样式：保留尺寸适配 */
        .msg.has-image {
            padding: 0 !important; /* 图片消息去掉内边距 */
            overflow: hidden !important;
        }
        .msg.has-image img {
            width: 100% !important;
            height: 100% !important;
            object-fit: cover !important;
        }
        /* 兼容原有动画样式 */
        .msg-bounce-in-left, .msg-bounce-in-right {
            display: inline-block !important;
        }
    </style>
</head>
<body>
    <div id="mobile" :class="{ 'has-prompt': hasPrompt }">
        <div id="mobile-head">
            <div id="mobile-head-title">与屠淮城聊天中</div>
        </div>
        <div id="mobile-body">
            <div id="mobile-body-bg"></div>
            <div id="mobile-body-content">
                <div id="mock-msg-row" class="msg-row">
                    <div id="mock-msg" class="msg" v-html="latestMsgContent"></div>
                </div>
                <!-- 核心修改：1. 修复文字显示 2. 区分图片/文字消息样式 -->
                <div class="msg-row"
                    v-for="(msg, index) in messages"
                    :key="index"
                    :class="msg.author === 'author' ? 'msg-author' : 'msg-me'">
                    <!-- 屠淮城的头像：仅对方消息显示 -->
                    <div v-if="msg.author === 'author'" class="msg-avatar">
                        <img src="img/avatar-tuhuai.jpg" alt="屠淮城头像">
                    </div>
                    <!-- 空头像占位：保证自己消息行布局对称，避免偏移 -->
                    <div v-else class="msg-avatar"></div>
                    <!-- 关键修复：仅图片消息应用固定宽高，文字消息不限制 -->
                    <div class="msg"
                        :class="[
                            'msg-bounce-in-' + (msg.author === 'author' ? 'left': 'right'),
                            msg.isImage ? 'has-image' : ''
                        ]"
                        :style="msg.isImage && msg.width && msg.height ? 
                                {width: msg.width - 26 + 'px', height: msg.height - 18 + 'px'} : 
                                {}"
                        v-html="msg.content"></div>
                </div>
            </div>
        </div>
        <div id="mobile-foot">
            <div id="prompt">
                <div id="prompt-head">
                    <div class="say-something">我想说……</div>
                    <a href="javascript:;" class="close-btn"
                        v-on:click="togglePrompt(false)"></a>
                </div>
                <div id="prompt-body">
                    <ul class="responses" v-if="lastDialog">
                        <li v-for="res in lastDialog.responses">
                            <a href="javascript:;" v-on:click="respond(res)">{{ res.content }}</a>
                        </li>
                    </ul>
                    <div class="next-topic"
                        v-if="!lastDialog || !lastDialog.responses">
                        <ul class="topics">
                            <li v-for="topic in nextTopics">
                                <a href="javascript:;" v-on:click="ask(topic)">{{ topic.brief }}</a>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
            <div id="input-hint" class="say-something"
                v-on:click="togglePrompt(true)"
                :class="{'clickable': !isTyping }">
                <span v-if="!isTyping">我想说……</span>
                <span v-if="isTyping">屠淮城正在输入中</span>
            </div>
        </div>
        <div id="prompt-bg" v-on:click="togglePrompt(false)"></div>
    </div>

    <script src="//cdn.bootcss.com/zepto/1.2.0/zepto.min.js"></script>
    <script src="//cdn.bootcss.com/vue/2.2.6/vue.min.js"></script>

    <!-- 重写打字时间逻辑，覆盖原有js中的配置 -->
    <script>
        // 先等待原有JS加载完成，再修改打字时长
        window.onload = function() {
            // 核心修改：延长屠淮城打字显示时间（单位：毫秒）
            // 原默认一般是1000-2000ms，这里改为5000ms（5秒），可根据需要调整
            Vue.prototype.TYPING_DURATION = 5000; // 全局定义打字时长
            // 如果原有JS用的是其他变量名，补充兼容
            if (window.app && window.app.$data) {
                // 额外增加消息发送间隔（每条消息之间的停顿）
                window.app.$data.TYPING_INTERVAL = 6000; // 6秒
            }
        };
    </script>
    <script src="js/index-min.js"></script>
</body>
</html>
