<template>
    <div ref="wrap">
        <!-- 公告 -->
        <div class="notice-main">
            <div ref='notice' class="notice-content">
                <div class="notice flex" v-for="(item, index) in noticeOldArr" :key='index' @click='noticeEvent(item.action)'>
                    <text class="c255 f35">{{i18n.Notice}}</text>
                    <div class="notice-lines"></div>
                    <text class="lines1 notice-text f28 fw4">{{item.title}}</text>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    const link = weex.requireModule("LinkModule");
    const linkapi = require('linkapi');
    const animation = weex.requireModule('animation')
    const dom = weex.requireModule("dom");
    export default {
        data() {
            return {
                channel: new BroadcastChannel('WidgetsMessage'),
                i18n: '',
                noticeOldArr: []
            }
        },
        methods: {
            noticeEvent(url) {
                linkapi.openLinkBroswer("", url);
            },
            getNoticeData() {
                link.getServerConfigs([], (params) => {
                    linkapi.get({
                        url: params.comwidgetsUri + '/notice/list',
                    }).then((res) => {
                        if (res.code == 200) {
                            let noticeArr = []
                            for (let index = 0; index < res.data.length; index++) {
                                const element = res.data[index];
                                let noticeObj = {}
                                let action = JSON.parse(element.action)
                                noticeObj['action'] = action.mobile_web ? action.mobile_web : action.web
                                noticeObj['title'] = element.title
                                noticeArr.push(noticeObj)
                            }
                            this.notice(noticeArr)
                            this.broadcastWidgetHeight()
                        }
                    }, (err) => {
                        this.error()
                    })
                }, (err) => {
                    this.error()
                });
            },
            error() {
                this.broadcastWidgetHeight()
            },
            notice(item) {
                let index = 1,
                    noticeItem = this.$refs.notice,
                    newFirstItem = item[0],
                    newLastItem = item[item.length - 1],
                    noticeArr = item;
                noticeArr.unshift(newLastItem)
                noticeArr.push(newFirstItem)
                this.noticeOldArr = noticeArr
                let noticeArrLength = this.noticeOldArr.length
                setInterval(() => {
                    index++
                    if (noticeArrLength - 1 == index) {
                        index = 1
                        this.animationNotice(noticeItem, 0, 0)
                    }
                    this.animationNotice(noticeItem, index, 500)
                }, 3000);
                this.broadcastWidgetHeight()
            },
            animationNotice(noticeItem, index, duration) {
                animation.transition(noticeItem, {
                    styles: {
                        transform: 'translateY(-' + 88 * index + 'px)',
                    },
                    duration: duration,
                    timingFunction: 'ease-in-out',
                    needLayout: false,
                    delay: 0
                }, function () { })
            },
            broadcastWidgetHeight() {
                let _params = this.$getPageParams();
                setTimeout(() => {
                    dom.getComponentRect(this.$refs.wrap, (ret) => {
                        this.channel.postMessage({
                            widgetHeight: ret.size.height,
                            id: _params.id
                        });
                    });
                }, 100)
            }
        },
        created() {
            linkapi.getLanguage((res) => {
                this.i18n = this.$window[res]
            })
        },
        mounted() {
            this.channel.onmessage = (event) => {
                if (event.data.action === 'RefreshData') {
                    this.getNoticeData()
                }
            }
            this.getNoticeData()
        }
    }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
    .notice-main {
        height: 88px;
        overflow: hidden;
    }

    .notice {
        height: 88px;
        background-color: #fff;
        padding: 0 24px;
    }

    .notice-lines {
        width: 1px;
        height: 44px;
        margin: 0 20px;
        background: rgba(216, 216, 216, 1);
    }

    .notice-text {
        width: 600px;
        color: rgba(77, 76, 79, 1);
    }

    .notice-content {
        background-color: #fff;
        transform: translateY(-88px);
    }

    .no-notice-content {
        height: 88px;
    }
</style>