<template>
    <div>
        <van-nav-bar
            title="优惠劵"
            left-arrow
            :fixed="true"
            :placeholder="true"
            @click-left="prev"
        />

        <div class="top">
            <span :class="{active: isActive == 1}" @click="changeCoupon(1)">未使用</span>
            <span :class="{active: isActive == 2}" @click="changeCoupon(2)">己使用/己过期</span>
        </div>
        <div class="top-placeholder"></div>

        <div class="list-wrap">
            <van-empty v-if="isEmpty" :image="emptyImage" :description="emptyDescription" />
            <van-list
                v-if="!isEmpty"
                v-model="loading"
                :finished="finished"
                finished-text="没有更多了"
                @load="onLoad"
            >

                <div class="list-box">
                    <div class="list-item" v-for="(item, index) in list" :key="index">
                        <div class="l">
                            <span>￥<i>{{item.amount}}</i></span>
                            <span v-if="item.price > 0">满{{item.price}}可用</span>
                            <span v-else>无门槛</span>
                        </div>
                        <div class="m">
                            <span>{{item.name}}</span>
                            <span>到期：{{ item.end_time }}</span>
                        </div>
                        <div class="r">
                            <span @click="$router.push('/ucenter/goods')">立即使用</span>
                        </div>
                    </div>
                </div>

            </van-list>
        </div>

    </div>
</template>

<script>
import { NavBar } from 'vant';
import { List,Empty,Toast } from 'vant';
export default {
    name: 'UCoupon',
    components: {
        [NavBar.name]: NavBar,
        [List.name]: List,
        [Empty.name]: Empty
    },
    data() {
        return {
            list: [],
            loading: false,
            finished: false,
            isActive:1,
            page: 1,
            isEmpty: false,
            emptyImage: "search",
            emptyDescription: "您还没有优惠劵哦",
        };
    },
    created() {

    },
    methods: {
        prev() {
            this.$tools.prev();
        },
        changeCoupon(index){
            this.page = 1;
            this.isActive = index;
            this.list = [];
            this.loading = true;
            this.onLoad();
        },
        onLoad() {
            this.isEmpty = false;
            let emptyImage = this.$request.domain() + 'static/images/coupon-empty.png';
            this.$request.get("/ucenter/coupon",{
                type: this.isActive,
                page: this.page
            }).then(result=>{
                if(result.data.list == undefined && this.page == 1){
                    this.isEmpty = true;
                    this.emptyImage = emptyImage;
                    this.emptyDescription = "您还没有优惠劵哦";
                } else if(result.status == 1){
                    this.list = this.list.concat(result.data.list);
                    this.loading = false;
                    this.page++;
                }else if(result.status == -1){
                    if(result.data == undefined && this.page == 1){
                        this.isEmpty = true;
                        this.emptyImage = emptyImage;
                        this.emptyDescription = "您还没有优惠劵哦";
                    }else{
                        this.loading = false;
                        this.finished = true;
                    }
                }
            }).catch((error)=>{
                this.isEmpty = true;
                this.emptyImage = "network";
                this.emptyDescription = "网络出错，请检查网络是否连接";
            });
        },
    },
}
</script>

<style lang="scss" scoped>
.top-placeholder { height: 45px; width: 100% }
.top{
    width: 100%;
    position: fixed;
    display: flex;
    flex-wrap: nowrap;
    flex-direction: row;
    height: 45px;
    line-height: 45px;
    text-align: center;
    background-color: #fff;
    font-size: 14px;
    color: #666;
    border-bottom: 1px solid #eee;
    span{
        width: 50%;
    }
    span:first-child{
        width: 49%;
        border-right: 1px solid #eee;
    }
    .active{
        color: #e72a2a;
    }
}
.list-wrap{
    width: 100%;
    margin-top: 10px;
    .list-item{
        width: 92%;
        height: 80px;
        border-radius: 5px;
        background-color: #fff;
        margin:  0 auto;
        margin-bottom: 10px;
        font-size: 13px;
        .l{
            width: 30%;
            height: 80px;
            float: left;
            span{
                color: red;
                display: block;
                text-align: center;
                height: 30px;
                line-height: 20px;
            }
            span:first-child{
                font-size: 16px;
                height: 50px;
                line-height: 60px;
                color: #ce3232;
                i{
                    font-size: 25px;
                    font-style: normal;
                    font-weight: bold;
                }
            }

        }
        .m{
            width: 47%;
            height: 80px;
            float: left;
            span{
                display: block;
            }
            span:first-child{
                padding-top: 10px;
                line-height: 25px;
            }
            span:last-child{
                height: 25px;
                line-height: 25px;
            }
        }
        .r {
            width: 22%;
            height: 80px;
            line-height: 80px;
            float: right;
            border-left: 1px solid #eee;
            text-align: center;
            span{
                padding: 5px 5px;
                background-color: #d91010;
                color: #fff;
                font-size: 12px;
                border-radius: 5px;
            }
        }
    }

}
</style>
