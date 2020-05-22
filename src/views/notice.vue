<template>
    <div ref="wrap">
        <!-- 公告 -->
        <div class="notice-main" :class="[noticeArray.length == 0? 'notice-no' : '']">
            <div ref='notice' class="notice-content">
                <div class="notice flex" v-for="(item, index) in noticeArray" :key='index' @click='noticeEvent(item.action)'>
                    <text class="f35" v-bind:style="{'color': themeColor}">{{i18n.Notice}}</text>
                    <div class="notice-lines"></div>
                    <text class="lines1 f28 fw4 flex1">{{item.title}}</text>
                </div>
            </div>
        </div>
        <div class='no-notice flex-ac flex-jc' v-if='noticeArray.length == 0 && isShow'>
            <div class="flex-dr flex-ac">
                <bui-image src="/image/sleep.png" width="20wx" height="20wx"></bui-image>
                <text class="f26 c51 fw4 pl15 center-height">{{isError?i18n.NoneData:i18n.ErrorLoadData}}</text>
            </div>
        </div>
    </div>
</template>

<script>
const link = weex.requireModule("LinkModule");
const linkapi = require('linkapi');
const animation = weex.requireModule('animation')
const dom = weex.requireModule("dom");
const storage = weex.requireModule('storage');
export default {
    data() {
        return {
            channel: new BroadcastChannel('WidgetsMessage'),
            i18n: '',
            noticeArray: [],
            timeout: null,
            refre: false,
            isError: true,
            isShow: false,
            themeColor: '',
            urlParams: {}
        }
    },
    created() {
        this.$fixViewport();
        linkapi.getLanguage((res) => {
            this.i18n = this.$window[res]
        })
        linkapi.getThemeColor(res => {
            this.themeColor = res.background_color;
        })
        this.urlParams = this.resolveUrlParams(weex.config.bundleUrl)
    },
    mounted() {
        var that = this
        this.channel.onmessage = (event) => {
            if (event.data.action === 'RefreshData') {
                this.refre = true
                this.getNoticeData()
            }
        }
        this.getStorage(function () {
            that.getNoticeData()
        })
    },
    methods: {
        getStorage(callback) {
            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
            storage.getItem('bulletinJLocalData1' + ecode + pageId, res => {
                console.log('公告数据', res);
                if (res.result == 'success' && !(res.data == 'undefined') && res.data) {
                    try {
                        var data = JSON.parse(res.data)
                        this.isShow = true
                        this.isError = true
                        this.notice(data)
                        clearInterval(this.timeout)
                        this.setIntervalEvent()
                        this.broadcastWidgetHeight()
                    } catch (error) {
                        callback()
                    }
                } else {
                    callback()
                }
            })
        },
        noticeEvent(url) {
            if (!url) return
            linkapi.openLinkBroswer("", url);
        },
        UrlAddParam(url, name, value) {
            var r = url;
            if (r != null && r != 'undefined' && r != "") {
                value = encodeURIComponent(value);
                var reg = new RegExp("(^|)" + name + "=([^&]*)(|$)");
                var tmp = name + "=" + value;
                if (url.match(reg) != null) {
                    r = url.replace(eval(reg), tmp);
                } else {
                    if (url.match("[?]")) {
                        r = url + "&" + tmp;
                    } else {
                        r = url + "?" + tmp;
                    }
                }
            }
            return r;
        },
        getUrl(actionParams) {
            if (!actionParams) return {};
            var sPkey = /\s/;
            var us = actionParams.split(sPkey);
            var params = {};
            for (var i = 0; i < us.length; i++) {
                var u = us[i];
                var idx = u.indexOf('=');
                if (idx > -1) {
                    params[u.substring(0, idx).trim()] = u.substring(idx + 1, u.length).trim();
                }
            }
            var url = params.url;
            delete params.url;
            for (var key in params) {
                url = this.UrlAddParam(url, key, params[key]);
            }
            return url;
        },
        getAction(action) {
            if (!action) return ''
            if (typeof action === 'string') {
                action = action.replace(/[\r\n]/g, ' ')
                try {
                    action = JSON.parse(action)
                } catch (e) {
                    action = eval('(' + action + ')');
                }
            }
            var pcAction = action.pc || action.android;
            if (pcAction.indexOf('[OpenUrl]') === 0) {
                return this.getUrl(pcAction);
            } else {
                return ''
            }
        },
        getNoticeData() {
            link.getServerConfigs([], (params) => {
                linkapi.get({
                    url: params.comwidgetsUri + '/notice/list',
                    data: {
                        limit: 8,
                        eCode: this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
                    }
                }).then((res) => {
                    try {
                        this.broadcastWidgetHeight()
                        this.noticeArray = []
                        if (res.code == 200) {
                            try {
                                this.isError = true
                                this.isShow = true
                                let noticeArr = []
                                for (let index = 0; index < res.data.length; index++) {
                                    const element = res.data[index];
                                    let noticeObj = {}
                                    let action = this.getAction(element.action)
                                    noticeObj["action"] = action
                                    noticeObj['title'] = element.title
                                    noticeArr.push(noticeObj)
                                }
                                let pageId = this.urlParams.userId ? this.urlParams.userId : ''
                                let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
                                storage.setItem('bulletinJLocalData1' + ecode + pageId, JSON.stringify(noticeArr))
                                this.notice(noticeArr)
                                // 数据不为空 并且 数据与上次不一致 并且需要刷新后
                                if (this.noticeArray.length != 0 && this.refre) {
                                    clearInterval(this.timeout)
                                    this.setIntervalEvent()
                                } else if (!this.refre) {
                                    this.setIntervalEvent()
                                }
                            } catch (error) {
                                this.error()
                            }
                        }
                    } catch (err) {
                        this.error()
                    }
                }, (err) => {
                    this.error()
                })
            }, (err) => {
                this.error()
            });
        },
        error() {
            this.isError = false
            this.isShow = true
            this.broadcastWidgetHeight()
        },
        resolveUrlParams(url) {
            // let url = weex.config.bundleUrl;
            if (!url) return {};
            url = url + "";
            var index = url.indexOf("?");
            if (index > -1) {
                url = url.substring(index + 1, url.length);
            }
            var pairs = url.split("&"),
                params = {};
            for (var i = 0; i < pairs.length; i++) {
                var pair = pairs[i];
                var indexEq = pair.indexOf("="),
                    key = pair,
                    value = null;
                if (indexEq > 0) {
                    key = pair.substring(0, indexEq);
                    value = pair.substring(indexEq + 1, pair.length);
                }
                params[key] = value;
            }
            return params;
        },
        notice(item) {
            let newFirstItem = item[0],
                newLastItem = item[item.length - 1],
                noticeArr = item;
            noticeArr.unshift(newLastItem)
            noticeArr.push(newFirstItem)
            this.noticeArray = noticeArr
        },
        setIntervalEvent() {
            try {
                let noticeArrLength = this.noticeArray.length,
                    noticeItem = this.$refs.notice,
                    index = 1
                if (this.refre) {
                    this.animationNotice(this.$refs.notice, 0, 0)
                    index = 0
                    this.refre = false
                }
                this.timeout = setInterval(() => {
                    let noticeArrLength = this.noticeArray.length
                    index++
                    if (noticeArrLength - 1 == index) {
                        index = 1
                        this.animationNotice(noticeItem, 0, 0)
                    }
                    this.animationNotice(noticeItem, index, 500)
                }, 3000);
            } catch (err) {
                this.error()
            }
        },
        animationNotice(noticeItem, index, duration) {
            try {
                animation.transition(noticeItem, {
                    styles: {
                        transform: 'translateY(-' + 88 * index + 'px)',
                    },
                    duration: duration,
                    timingFunction: 'ease-in-out',
                    needLayout: false,
                    delay: 0
                }, function () { })
            } catch (error) {
                this.error()
            }
        },
        getComponentRect(_params) {
            dom.getComponentRect(this.$refs.wrap, (ret) => {
                this.channel.postMessage({
                    widgetHeight: ret.size.height,
                    id: _params.id
                });
            });
        },
        broadcastWidgetHeight() {
            let _params = this.$getPageParams();
            // 防止高度通知失败
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 200)
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 1200)
        }
    }
}
</script>

<style lang="css" src="../css/common.css"></style>
<style>
.notice-main {
    height: 88px;
    overflow: hidden;
}
.no-notice {
    height: 88px;
    background-color: #fff;
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

.notice-el-text {
    width: 500px;
    color: rgba(77, 76, 79, 1);
}

.notice-content {
    background-color: #fff;
    transform: translateY(-88px);
}

.no-notice-content {
    height: 88px;
}

.center-height {
    line-height: 40px;
}
.notice-no {
    height: 0;
}
</style>