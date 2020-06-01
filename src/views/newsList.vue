<template>
    <div ref="wrap" style="flex:1">
        <!-- 新闻列表 -->
        <bui-header :title="title" :leftItem="leftItem" @leftClick="back">
        </bui-header>
        <div class="news-list" style="flex:1">
            <bui-dropload :hasRefresh="hasRefresh" :droploadStyle="droploadStyle" :refreshStyle="refreshStyle" :refreshTextStyle="refreshTextStyle" :refreshTextMap="refreshTextMap" :refreshIconStyle="refreshIconStyle" :loadingStyle="loadingStyle" :loadingTextStyle="loadingTextStyle" :loadingIconStyle="loadingIconStyle" :loadingTextMap="loadingTextMap" :hasLoading='hasLoading' @loading="loading" @refresh="refresh">
                <cell v-for="(item,index) in newsList" :key='index' @click='newsListItemEvent(item.action)'>
                    <div v-if='!item.more'>
                        <div class="flex-dr content-item" v-if='item.image !=""'>
                            <div class='content-item-left'>
                                <bui-image @click='newsListItemEvent(item.action)' placeholder='/image/ellipsis.png' :src="item.image" radius='10' width="100wx" height="73wx"></bui-image>
                            </div>
                            <div class='content-item-right flex-sb'>
                                <text class="item-right-text lines2 f28 c0 fw4">{{item.title}}</text>
                                <div class="item-right-date flex">
                                    <div class="date-origin flex">
                                        <text class="f24 c9">{{item.time}}</text>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div v-if='item.image ==""' class="flex-sb flex-dr flex-ac content-no-image">
                            <text class="f28 fw5 c0 lines1 flex10">{{item.title}}</text>
                            <text class="f24 c9 flex2 tr">{{item.time}}</text>
                        </div>
                    </div>
                    <div v-if='item.more' class="flex-sb flex-dr flex-jc pull">
                        <text v-if='pullMore' class="f28 fw5 c0">{{pullUpMore}}</text>
                    </div>
                </cell>
            </bui-dropload>
        </div>
        <div class="pre-loading" v-if='preLoading'>
            <div class="select-box">
                <bui-image src="/image/loding.gif" width="30wx" height="30wx"></bui-image>
            </div>
        </div>
    </div>
</template>

<script>
const link = weex.requireModule("LinkModule");
const linkapi = require("linkapi");
const dom = weex.requireModule("dom");
const storage = weex.requireModule('storage');
var storeFileIdUiDownload = '/store/getFile?fileId=${id}';
export default {
    data() {
        return {
            newsList: [],
            i18n: "",
            preLoading: true,
            pullMore: true,
            leftItem: {
                icon: 'ion-chevron-left',
            },
            total: 10,
            title: '品高快报',
            droploadStyle: {
                backgroundColor: '#fff'
            },
            refreshStyle: {
                backgroundColor: '#ccc'
            },
            refreshTextMap: {
                start: '准备下拉',
                move: '可以释放啦',
                success: '刷新完成',
                fail: '刷新失败啦'
            },
            refreshStyle: {
                backgroundColor: '#fff'
            },
            refreshTextStyle: {
                color: '#4da4fe',
            },
            refreshIconStyle: {
                color: '#4da4fe',
            },
            loadingStyle: {
                backgroundColor: '#fff'
            },
            loadingTextStyle: {
                color: '#4da4fe',
            },
            loadingIconStyle: {
                color: '#4da4fe',
            },
            loadingTextMap: {
                start: '向上拉加载更多',
                success: '加载完成',
                empty: '没有更多了',
                fail: '加载失败，请重试'
            },
            hasLoading: true,
            hasRefresh: true,
            pullUpMore: '上拉加载更多',
            urlParams: {}
        };
    },
    created() {
        this.$fixViewport();
        this.urlParams = this.resolveUrlParams(weex.config.bundleUrl)
        linkapi.getLanguage(res => {
            this.i18n = this.$window[res];
            this.title = this.urlParams.title ? decodeURI(this.urlParams.title) : this.$window[res].new
            this.refreshTextMap = {
                start: this.$window[res].start,
                move: this.$window[res].move,
                success: this.$window[res].success,
                fail: this.$window[res].fail
            }
            this.loadingTextMap = {
                start: this.$window[res].loadMoreStart,
                success: this.$window[res].loadMoreSuccess,
                empty: this.$window[res].loadMoreEmpty,
                fail: this.$window[res].loadMoreFail
            }
            this.pullUpMore = this.$window[res].pullUpMore
        });
    },
    mounted() {
        this.getNewsData(this.total)
    },
    methods: {
        loading(next) {
            this.pullMore = false
            setTimeout(() => {
                next && next();
            }, 200)
            this.total += 10;
            this.getNewsData(this.total)
        },
        refresh(next) {
            setTimeout(() => {
                next && next();
            }, 200)
            this.total = 10;
            this.getNewsData(this.total)
        },
        back: function () {
            // 返回上一个页面
            this.$pop();
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
        getNewsData(total) {
            link.getServerConfigs([], params => {
                linkapi.get({
                    url: params.comwidgetsUri + "/news/list",
                    data: {
                        limit: total
                    }
                }).then(res => {
                    this.preLoading = false
                    this.hasLoading = this.total < res.total
                    this.pullMore = this.total < res.total
                    if (res.code == 200) {
                        try {
                            let newsArr = [];
                            for (let index = 0; index < res.data.length; index++) {
                                const element = res.data[index];
                                let newsItemObj = {};
                                let action = this.getAction(element.action)
                                newsItemObj["action"] = action
                                newsItemObj["title"] = element.title;
                                newsItemObj["time"] = this.timestampToTime(element.publishTime);
                                newsItemObj["image"] = this.parseImgUrl(element.image, params.storeUri || params.storeUrl);
                                newsArr.push(newsItemObj);
                            }
                            newsArr.push({
                                title: '下拉加载更多',
                                more: true
                            })
                            this.newsList = newsArr;
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
            url = (uam + storeFileIdUiDownload).replace('${id}', url);
            return url;
        },
        error() {
            this.newsList = []
        }
    }
};
</script>

<style lang="css" src="../css/common.css"></style>
<style>
.news-list-title {
    height: 44wx;
    padding: 0 12wx;
    border-bottom: 1px solid #f2f2f2;
}

.content-item {
    margin: 13wx 11wx 13wx 12wx;
}

.content-item-left {
    width: 200px;
}

.content-item-right {
    margin-left: 10wx;
    flex: 1;
    flex-direction: column;
}

.item-right-text {
    flex: 1;
    color: rgb(165, 164, 164);
}

.item-right-date {
    padding-top: 10wx;
}

.date-origin {
    width: 200px;
}

.content-no-image {
    height: 40wx;
    margin: 9wx 11wx 5wx 12wx;
}

.pre-loading {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    justify-content: center;
    align-items: center;
}
.select-box {
    justify-content: center;
    align-items: center;
    border-radius: 10wx;
    width: 68wx;
    height: 68wx;
    background-color: rgba(0, 0, 0, 0.8);
}
.pull {
    height: 50px;
    line-height: 50px;
}
</style>