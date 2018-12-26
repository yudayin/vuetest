<template>
  <transition name="detail">
    <div class="food" v-show="showFlag" ref="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image"/>
          <div class="back" @click="hideFoodDetail">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="nowPrice">¥{{food.price}}</span><span class="oldPrice" v-show="food.oldPrice">¥{{food.oldPrice}}</span>
          </div>
        </div>
        <div class="cartcontrol-wrapper">
          <cartcontrol :food="food" v-on:cart-add="detailAddCart"></cartcontrol>
        </div>
        <transition name="fade">
          <div class="buy" @click.stop.prevent="addFirstFood" v-show="!food.count || food.count===0">加入购物车</div>
        </transition>
      </div>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
import Vue from 'vue'
import BScroll from 'better-scroll'
import cartcontrol from '../cartcontrol/cartcontrol'
export default {
  props: {
    food: {
      type: Object
    }
  },
  data () {
    return {
      showFlag: false
    }
  },
  methods: {
    showFoodDetail () {
      this.showFlag = true
      this.$nextTick(() => {
        if (!this.scroll) {
          this.scroll = new BScroll(this.$refs.food, {
            click: true
          })
        } else {
          this.scroll.refresh()
        }
      })
    },
    hideFoodDetail () {
      this.showFlag = false
    },
    addFirstFood (event) {
      if (!event._constructed) {
        return
      }
//      console.log(event.target)
      this.$emit('first-add', event.target)
      Vue.set(this.food, 'count', 1)
    },
    detailAddCart (target) { // 跟add关联的addFood方法
      this.$emit('first-add', target) // 触发当前实例food上的事件add(在goods组件上绑定在food组件的add方法)
    }
  },
  components: {
    BScroll,
    cartcontrol
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .food
    position fixed
    left 0
    top 0
    bottom 48px
    z-index 30
    width 100%
    background #f3f5f7
    transform translate3d(0, 0, 0)
    &.detail-enter-active, &.detail-leave-active
      transition: all .2s linear
      transform translate3d(0, 0, 0)
    &.detail-enter, &.detail-leave-active
      transform translate3d(100%, 0, 0)
    .image-header
      position relative
      width 100%
      height 0
      padding-top 100%
      img
        position: absolute
        left 0
        top 0
        width 100%
        height 100%
      .back
        position absolute
        top 10px
        left 0
        .icon-arrow_lift
          display block
          padding 10px
          font-size 20px
          color #fff
    .content
      padding 18px
      .title
        line-height 14px
        margin-bottom 8px
        font-size 14px
        font-weight 700
        color rgb(7, 17, 27)
      .detail
        padding-bottom 18px
        height 10px
        line-height 10px
        font-size 0
        .sell-count, .rating
          font-size 10px
          color rgb(147, 153, 159)
        .sell-count
          margin-right 12px
      .price
        font-weight: 700
        line-height: 24px
        .nowPrice
          margin-right: 8px
          font-size: 14px
          color: rgb(240, 20, 20)
        .oldPrice
          text-decoration: line-through
          margin-right: 8px
          font-size: 10px
          color: rgb(147, 153, 159)
    .cartcontrol-wrapper
      position absolute
      right 12px
      bottom 12px
    .buy
      position absolute
      right 18px
      bottom 18px
      z-index 10
      height 24px
      line-height 24px
      padding 0 12px
      box-sizing border-box
      border-radius 12px
      font-size 10px
      color #fff
      background rgb(0, 160, 220)
      opacity: 1 //加入动画,一个是为了体验,另外一个是为了延迟触发隐藏,避免小球抛物线动画出现问题
      &.fade-enter-active, &.fade-leave-active
        transition: all 0.2s
      &.fade-enter, &.fade-leave-active
        opacity: 0
        z-index: -1

  //
</style>
