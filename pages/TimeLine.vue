<template>
    <the-main-body>        
        <template slot='data'>
            <base-time-line-node v-for="tlnode in timeLineNodes" :todo="tlnode" :key="tlnode.id" @refresh="refresh">
                
            </base-time-line-node>
        </template>
        <template slot='more'>
            <ul class="pager">
                <li>
                    <small v-if="page == 0">总共{{ totalNum }}条数据。(●ˇ∀ˇ●)</small>
                    <small v-else>下拉加载更多</small>
                </li>
            </ul>
        </template>
    </the-main-body>
</template>

<script>
    import TheMainBody from '../components/TheMainBody'
    import BaseTimelineNode from '../components/BaseTimelineNode'
    import _ from 'lodash'

    export default {
        components: {
            "the-main-body": TheMainBody,
            "base-time-line-node": BaseTimelineNode
        },
        computed: {
            // 所有的时光节点
            timeLineNodes () {
                return this.$store.state.data;
            },
            // 当前页码，如果为0，表示没有更多的数据了
            page () {
                return this.$store.state.page;
            },
            // 当前页最大数据数量
            count () {
                return this.$store.state.num;    
            },
            // 当前页面总共有多少条数据
            totalNum () {
                return this.$store.state.count;
            }
        },
        methods: {
            // 鼠标滑动到底部，触发事件
            scrollBottom: _.debounce(function() {
                // 滚动条滚动时，距离顶部的距离
                var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
                // 可视区的高度
                var windowHeight = document.documentElement.clientHeight || document.body.clientHeight;
                // 滚动条的总高度
                var scrollHeight = document.documentElement.scrollHeight || document.body.scrollHeight;
                // 滚动条到达底部
                if (scrollTop + windowHeight >= scrollHeight) {
                    // 自动加载更多
                    this.getMore();
                }              
            }, 500),
            // 刷新页面
            refresh () {
                // 初始化各种数据
                let _this = this;
                _this.$store.commit('set', [
                    {
                        key: 'page',
                        value: 1
                    },
                    {
                        key: 'data',
                        value: []
                    },
                    {
                        key: 'count',
                        value: 0
                    }
                ]);
                // 添加对鼠标的监听事件
                window.addEventListener('scroll', _this.scrollBottom);
                // 重新请求数据   
                _this.getMore();
            },
            // 向服务器请求数据，默认一次请求10个
            getMore () {
                let _this = this;
                if (_this.page != 0) {
                    _this.$http
                        .get('/api/timelines', {
                            params: {
                                page: _this.page,
                                count: _this.count
                            }
                        })
                        .then(function (response) {
                            // 成功
                            // 提交数据
                            var newData = response.data.results;
                            _this.$store.commit('addData', {
                                content: newData
                            });                        
                            _this.$store.commit('set', [
                                {
                                    key: 'page',
                                    // 如果新数据少于本页面的最大数据，说明没有更多的数据了
                                    value: newData.length == _this.count ? _this.page + 1 : 0
                                },
                                {
                                    key: 'count',
                                    // 计数，总共的数据量
                                    value: _this.totalNum +  newData.length
                                }
                            ])                                               
                        })
                        .catch(function (error) {
                            // 返回404错误，说明没有更多数据了
                            if (error.message == '404') {
                                // 发生错误
                                _this.$store.commit('set', [
                                    {
                                        key: 'page',
                                        value: 0
                                    }
                                ]) 
                            }  
                        })
                } else {
                    // 移除鼠标监听事件
                    window.removeEventListener('scroll', this.scrollBottom);
                }
            }
        },
        mounted: function () {
            // 页面组件成功挂载后，初始化页面数据
            this.getMore();     
            // 添加对鼠标的监听事件
            window.addEventListener('scroll', this.scrollBottom);              
        },
        destroyed: function () {
            // 移除对鼠标的监听事件
            window.removeEventListener('scroll', this.scrollBottom);
        }
    }
</script>

<style scoped>
    small {
        color: grey;
        font-style: italic;
    }
</style>