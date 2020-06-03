<template>
    <div ref="wrap">
        <!-- 新闻列表 -->
        <div class="news-list">
            <div class="news-list-title flex" v-bind:style="{'height': $isIPad ? '44wx': '88px'}">
                <div class="title flex">
                    <span class="line" v-bind:style="{'background-color': themeColor}"></span>
                    <text class="c0 f30">{{i18n.new}}</text>
                </div>
                <text class="f24 c153 fw4 pl20 pt10 pb10" @click='newsMoreEvent()'>{{i18n.All}}</text>
            </div>
            <div v-if='isShow'>
                <div v-if='newsList.length != 0'>
                    <div v-for="(item,index) in newsList" :key='index' @click='newsListItemEvent(item.action)'>
                        <div class="flex-dr" v-if='item.image !=""' :class="[ $isIPad ? 'marWx' : 'marPx']">
                            <div class='content-item-left'>
                                <bui-image @click='newsListItemEvent(item.action)' placeholder='/image/ellipsis.png' :src="item.image" radius='10' width="200px" height="73wx"></bui-image>
                            </div>
                            <div class='content-item-right flex-sb'>
                                <text class="lines2 f28 c0 fw4">{{item.title}}</text>
                                <text class="lines1 f26 c9 fw4">{{item.brief}}</text>
                                <div class="flex">
                                    <div class="date-origin flex">
                                        <text class="f24 c9">{{item.time}}</text>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div v-if='item.image ==""' class="flex-sb flex-dr flex-ac" v-bind:style="{'height': $isIPad ? '40wx' : '80px'}" :class="[ $isIPad ? 'nmarWx' : 'nmarPx']">
                            <text class="f28 fw5 c0 lines1 flex10">{{item.title}}</text>
                            <text class="f24 c9 flex2 tr">{{item.time}}</text>
                        </div>
                    </div>
                </div>
                <div class="no-content flex-ac flex-jc" v-bind:style="{'height': $isIPad ? '200wx': '400px'}" v-else>
                    <div class="flex-dr">
                        <text class="f26 c51 fw4 pl15 center-height">{{isError?i18n.NoneData:i18n.ErrorLoadData}}</text>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
const link = weex.requireModule("LinkModule");
const linkapi = require("linkapi");
const dom = weex.requireModule("dom");
const storage = weex.requireModule('storage');
var uamFileIdUiDownload = '/ui/download?fileId=${id}&access=anonymous';
export default {
    data() {
        return {
            newsList: [],
            channel: new BroadcastChannel("WidgetsMessage"),
            i18n: "",
            isShow: false,
            isError: true,
            themeColor: '',
            ecode: '',
            urlParams: {}
        };
    },
    created() {
        this.$fixViewport();
        linkapi.getLanguage(res => {
            this.i18n = this.$window[res];
        });
        linkapi.getThemeColor(res => {
            this.themeColor = res.background_color;
        })
        this.$isIPad = this.$isIPad()
        this.urlParams = this.resolveUrlParams(weex.config.bundleUrl)
    },
    mounted() {
        var that = this
        this.channel.onmessage = event => {
            if (event.data.action === "RefreshData") {
                this.getNewsData();
            }
        };
        this.getStorage(function () {
            that.getNewsData()
        })
    },
    methods: {
        newsMoreEvent() {
            let runApp = {
                appCode: 'bingo_newsList',
                data: {
                    title: this.i18n.new
                }
            }
            // 打开应用的方式
            linkapi.runApp(runApp)
            // link.launchLinkService(["[OpenApp] \n appCode=bingo_newsList"], (res) => { }, (err) => { });
        },
        // 新闻列表
        newsListItemEvent(url) {
            if (url) {
                linkapi.openLinkBroswer("", url);
            } else {
                this.$toast('地址不存在');
            }
        },
        timestampToTime(timestamp) {
            var date = new Date(timestamp);
            var Y = date.getFullYear() + "-";
            var M =
                (date.getMonth() + 1 < 10
                    ? "0" + (date.getMonth() + 1)
                    : date.getMonth() + 1) + "-";
            var D = date.getDate() + " ";
            var h = date.getHours() + ":";
            var m = date.getMinutes() + ":";
            var s = date.getSeconds();
            return M + D;
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
        getStorage(callback) {
            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
            storage.getItem('newListJLocalData202062' + ecode + pageId, res => {
                if (res.result == 'success') {
                    var data = JSON.parse(res.data)
                    this.isShow = true
                    this.isError = true
                    this.newsList = data;
                    this.broadcastWidgetHeight()
                } else {
                    callback()
                }
            })
        },
        getNewsData() {
            link.getServerConfigs([], params => {
                let urlParams = this.resolveUrlParams(weex.config.bundleUrl)
                linkapi.get({
                    url: params.comwidgetsUri + "/news/list",
                    data: {
                        limit: 5,
                        eCode: urlParams.ecode ? urlParams.ecode : 'localhost'
                    }
                }).then(res => {
                    this.isShow = true
                    this.isError = true
                    if (res.code == 200) {
                        try {
                            let newsArr = [];
                            for (let index = 0; index < res.data.length; index++) {
                                const element = res.data[index];
                                let newsItemObj = {};
                                let action = this.getAction(element.action)
                                newsItemObj["action"] = action
                                newsItemObj["brief"] = element.brief
                                newsItemObj["title"] = element.title;
                                newsItemObj["time"] = this.timestampToTime(element.publishTime);
                                newsItemObj["image"] = this.parseImgUrl(element.image, params.uamUri || params.uamUrl);
                                newsArr.push(newsItemObj);
                            }
                            this.newsList = newsArr;
                            this.broadcastWidgetHeight();
                            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
                            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
                            storage.setItem('newListJLocalData202062' + ecode + pageId, JSON.stringify(newsArr))
                        } catch (error) {
                            this.error();
                        }
                    }
                }, err => {
                    this.error();
                }
                );
            }, err => {
                this.error();
            });
        },
        parseImgUrl(url, uam) {
            if (!url) return url;
            if (url.match(/^https?:\/\//g)) {
                return url
            }
            if (url.startsWith("store://")) {
                url = url.replace("store://", "");
            }
            url = (uam + uamFileIdUiDownload).replace('${id}', url);
            return url;
        },
        error() {
            this.isError = false
            this.isShow = true
            this.broadcastWidgetHeight();
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
        }
    }
};
</script>

<style lang="css" src="../css/common.css"></style>
<style>
.news-list {
    background-color: #fff;
    padding-bottom: 10px;
}

.line {
    width: 5px;
    height: 36px;
    margin-right: 12px;
}

.news-list-title {
    height: 44wx;
    padding: 0 12wx;
    border-bottom: 1px solid #f2f2f2;
}

.content-item-left {
    width: 200px;
}

.content-item-right {
    position: relative;
    margin-left: 10wx;
    flex: 1;
    flex-direction: column;
}

.date-origin {
    width: 200px;
}

.no-content {
    flex: 1;
}

.marWx {
    margin: 6wx 5wx 6wx 5wx;
}
.marPx {
    margin: 12px 10px 12px 10px;
}
.nmarWx {
    margin: 9wx 11wx 5wx 12wx;
}
.nmarPx {
    margin: 18px 22px 10px 24px;
}
</style>