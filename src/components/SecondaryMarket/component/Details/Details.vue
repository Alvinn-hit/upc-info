<template>
    <div class="Details second-market box-center">
        <header-section>
            <p>{{title}}</p>
        </header-section>
        <div class="body box-center">
            <manager-header :userData="managerUserData" :managerPrice="managerData.managerPrice"></manager-header>
            <hr style="border: 1px solid #EBEBEB;border-bottom: none">
            <p class="banner">{{managerData.managerDetail}}</p>
            <div v-for="item in managerData.imgUrl" class="banner">
                <img :src="item" alt="图片加载失败" class="descimg box-center" v-if="item !== ''">
            </div>
        </div>
        <reply-box class="second-market"></reply-box>
        <div class="discuss-list box-center second-market" v-for="item in discussList">
            <img :src="item.ybHeader" alt="图片加载失败">
            <div>
                <span>{{item.ybName}}</span>
                <p>{{item.detail}}</p>
                <hr style="border: 1px solid #EBEBEB;border-bottom: none;text-align: right;">
            </div>
        </div>

        <div style="height: 8rem"></div>
        <div class="footer box-center second-market" v-if="showDiscuss">
            <label>
                <input type="text" v-model="discussDetail" @key.enter="addDiscuss">
                <button @click="addDiscuss">提交</button>
            </label>
        </div>
        <div class="footer clear second-market box-center" v-if="!showDiscuss">
            <ul>
                <li class="float-left" @click="addCollection" v-if="!isCollected"><img :src="collectionOff"
                                                                                       alt="图片加载失败">&nbsp;收藏
                </li>
                <li class="float-left" @click="addCollection" v-if="isCollected"><img :src="collectionOn" alt="图片加载失败">&nbsp;收藏
                </li>
                <li class="float-left" @click="changeDiscuss"><img :src="discuss" alt="图片加载失败">&nbsp;评论</li>
                <li class="float-left text-center">
                    <button @click="showUserProfile">我想要</button>
                </li>
            </ul>
        </div>
        <div class="mask" @click="closeProfile" v-show="showProfile">
            <transition name="showProfile">
                <user-profile class="user-profile" :profile="publisherProfile" v-if="showProfile"></user-profile>
            </transition>
        </div>
    </div>
</template>

<script>
    import HeaderSection from '../../common-component/HeaderSection.vue'
    import ManagerHeader from '../../common-component/ManagerHeader.vue'
    import ReplyBox from '../../common-component/ReplyBox.vue'
    import UserProfile from '../../common-component/UserProfile.vue'
    import updateData from '../../../../common/mixins/UpdateData'
    import { marketFetch, yibanAuth } from '../../config/fetchUtil'
    import loading from '../../../../common/mixins/loading'
    import fetchVq from '../../../../common/mixins/fetchVq'
    import findManagerById from '../../fetch/findManagerById'
    import addCollection from '../../fetch/addCollection'
    import addDiscuss from '../../fetch/addDiscuss'
    import userData from '../../../../common/mixins/userData'
    import fetchDiscussList from '../../fetch/fetchDiscussList'

    /** @namespace $route.params.articleId */
    /** @namespace managerData */
    export default {
        name: 'Details',
        mixins: [updateData, loading(marketFetch), fetchVq(yibanAuth), userData],

        data () {
            return {
                managerData: {},
                title: '加载中...',
                managerUserData: [],
                managerList: [],
                color: '#004073',
                collectionOn: require('../../media/collection-on.png'),
                collectionOff: require('../../media/collection-off.png'),
                discuss: require('../../media/discuss.png'),
                showDiscuss: false,
                discussDetail: '',
                discussList: [],
                isCollected: false,
                showProfile: false,
                publisherProfile: {}
            }
        },
        created () {
            this.routeUpdate(parseInt(this.$route.params.articleId))
                .then(() => {
                    this.fetch.getTextData('/secondhand/collention/iscollection', {
                        articleid: this.$route.params.articleId
                    }).then(r => {
                        this.isCollected = r === 'true'
                    })
                })
            this.fetchDiscussList()
        },
        watch: {
            $route (to, from) {
                this.routeUpdate(parseInt(to.params.articleId))
            }
        },
        components: {
            HeaderSection,
            ManagerHeader,
            ReplyBox,
            UserProfile
        },

        methods: {
            routeUpdate (id) {
                const localData = this.getLocalData(id)
                if (localData === null) {
                    return this.fetchWithSave(id)
                } else {
                    return new Promise((resolve) => {
                        this.managerData = localData.managerData
                        this.managerUserData = localData.managerUserData
                        resolve()
                    })
                }
            },
            fetchWithSave (id) {
                return this.findManagerById(id).then(data => {
                    if (data.code !== 0) {
                        this.managerData = data.managerData
                        this.managerUserData = data.userData
                        this.managerList.push({
                            managerData: this.managerData,
                            managerUserData: this.managerUserData
                        })
                        console.log('managerData', this.managerData)
                        this.title = this.managerData.name
                    } else {
                        this.title = '加载中...'
                    }
                })
            },
            getLocalData (id) {
                for (let i = 0; i < this.managerList.length; i++) {
                    let item = this.managerList[i]
                    if (item.managerData.id === id) {
                        return item
                    }
                }
                return null
            },
            addCollection () {
                addCollection(this.$route.params.articleId, this).then(data => {
                    if (data === false) {
                        this.isCollected = true
                        alert('收藏成功')
                    } else {
                        alert('请勿重新收藏')
                    }
                })
            },
            addDiscuss () {
                addDiscuss(this.$route.params.articleId, this.discussDetail, this).then(data => {
                    this.discussList.unshift(data)
                    this.changeDiscuss()
                })
            },
            changeDiscuss () {
                this.showDiscuss = !this.showDiscuss
            },
            fetchDiscussList () {
                fetchDiscussList(this.$route.params.articleId, this)
                    .then(discussList => {
                        this.discussList = discussList
                    })
            },
            showUserProfile () {
                this.fetch.getJsonData('/second/user/otherinfo', {
                    userid: this.managerUserData.userId
                }).then(userInfo => {
                    this.$set(this, 'publisherProfile', {
                        avatar: userInfo.ybhead,
                        qq: userInfo.qq ? userInfo.qq : '无',
                        wchat: userInfo.wchat ? userInfo.wchat : '无',
                        phone: userInfo.phone ? userInfo.phone : '无',
                        email: userInfo.email ? userInfo.email : '无'
                    })
                    this.showProfile = true
                })
            },
            closeProfile () {
                this.showProfile = false
            },
            findManagerById
        }
    }
</script>


<style scoped>
    .Details .body {
        width: 95%;
    }

    .Details .banner {
        margin: 1rem 1rem;
        color: #595959;
        height: auto;
    }

    .Details .banner .descimg {
        width: 100%;
        overflow-x: hidden;
    }

    .Details .footer {
        position: fixed;
        display: flex;
        flex-wrap: nowrap;
        justify-content: space-between;
        height: 75px;
        line-height: 75px;
        border-top: 1px solid #B2B1B1;
        bottom: 0;
        left: 0;
        right: 0;
        background-color: white;
    }

    .Details .footer ul {
        width: 100%;
        display: flex;
        justify-content: space-around;
    }

    .Details .footer li {
        width: 8rem;
        color: #767676;
        cursor: pointer;
        margin-left: 1rem;
        margin-right: 1rem;
    }

    .Details .footer img {
        vertical-align: middle;
        opacity: 0.5;
    }

    .Details .footer li button {
        width: 4rem;
        background: #FE1E1E;
        border: none;
        height: 2.2rem;
        font-size: 0.9rem;
        color: #FEE4EB;
        cursor: pointer;
    }

    .Details .footer label {
        width: 100%;
        border-bottom: 1px solid #B2B1B1;
        border-left: 1px solid #B2B1B1;
        border-right: 1px solid #B2B1B1;
        text-align: center;
        background-color: #D0D4DA;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .Details .footer input {
        width: 24rem;
        margin-left: 1rem;
        height: 1.5rem;
        font-size: 0.8rem;
        border: none;
    }

    .Details .footer label button {
        background: #BCBCBC;
        border-radius: 10px;
        margin-right: 0.5rem;
        border: none;
        color: white;
        height: 1.5rem;
        width: 5rem;
    }

    .Details .discuss-list {
        margin: 1rem auto;
        padding: 0 1.2rem;
        max-width: 95%;
        display: flex;
        overflow-x: hidden;
    }

    .Details .discuss-list img {
        width: 2rem;
        height: 2rem;
        border-radius: 1rem;
        vertical-align: top;
    }

    .Details .discuss-list div {
        display: inline-block;
        margin-left: 0.7rem;
        width: calc(100%-1rem);
    }

    .Details .discuss-list p {
        margin-top: 1rem;
        word-wrap: break-word;
        word-break: break-all;
    }

    .Details .discuss-list a {
        display: inline-block;
        margin-top: 0.6rem;
        font-size: 0.8rem;
        font-weight: 600;
    }

    .Details .discuss-list a:hover {
        color: blue;
        border-bottom: 1px solid blue;
        margin-bottom: -1px;
    }

    .Details .mask {
        position: fixed;
        top: 0%;
        height: 100%;
        width: 100%;
        background: #4040407f
    }

    .Details .user-profile {
        z-index: 900;
        position: fixed;
        top: 20%;
        width: 90%;
        transform-origin: center;
    }

     @keyframes show{
        0%{
            transform: scale(0.1);
        }
        100%{
            transform: scale(1);
        }
    }

    .showProfile-enter-active{
        animation: show .2s;
    }
    .showProfile-leave-active{
        animation: show .2s reverse;
    }
</style>
