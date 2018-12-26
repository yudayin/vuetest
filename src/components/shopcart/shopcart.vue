<template>
  <div>
  <div class="shopcart">
    <div class="content" @click="toggleList">
      <div class="content-left">
        <div class="logo-wrapper">
          <div class="logo" :class="{'highlight':totalCount>0}">
            <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
          </div>
          <div class="num" v-show="totalCount>0">{{totalCount}}</div>
        </div>
        <div class="price" :class="{'highlight':totalPrice >0}">¥{{totalPrice}}</div>
        <div class="desc">另需配送费¥{{deliveryPrice}}元</div>
      </div>
      <div class="content-right" @click.stop.prevent="pay">
        <div class="pay" :class="payClass">
          {{payDesc}}
        </div>
      </div>
    </div>
    <div class="ball-container">
      <!--eslint-disable-next-line-->
      <div v-for="(ball,index) in balls">
        <transition name="dropdown" @before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter">
          <div v-show="ball.show" :key="index" class="ball">
            <div class="inner inner-hook"></div>
          </div>
        </transition>
      </div>
    </div>
    <transition name="fold">
      <div class="shopcart-list" v-show="listShow">
        <!--<transition name="fold">-->
        <div class="list-header">
          <h1 class="title">购物车</h1>
          <span class="empty" @click="clearCart">清空</span>
        </div>
        <div class="list-content" ref="listContent">
          <ul>
            <!--eslint-disable-next-line-->
            <li class="food" v-for="(food,index) in selectFoods">
              <span class="name">{{food.name}}</span>
              <div class="price">
                <span>¥{{food.price * food.count}}</span>
              </div>
              <div class="cartcontrol-wrapper">
                <cartcontrol :food="food"></cartcontrol>
              </div>
            </li>
          </ul>
        </div>
        <!--</transition>-->
      </div>
    </transition>
  </div>
  <transition name="fade">
     <!--listShow表示当list详情列表显示的时候mask才显示 -->
    <div class="list-mask" v-show="listShow" @click="hideList()"></div>
  </transition>
  </div>
</template>

<script type="text/ecmascript-6">
import cartcontrol from '../cartcontrol/cartcontrol'
import BScroll from 'better-scroll'
export default {
  props: {
    selectFoods: {
      type: Array,
      default () {
        return []
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      balls: [
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        }
      ],
      dropBalls: [],
      foldFlag: true
    }
  },
  computed: {
    totalPrice () {
      let total = 0
      this.selectFoods.forEach((food) => {
        total += food.price * food.count
      })
      return total
    },
    totalCount () {
      let count = 0
      this.selectFoods.forEach((food) => {
        count += food.count
      })
      return count
    },
    payDesc () {
      if (this.totalPrice === 0) {
        return `¥${this.minPrice}元起送`
      } else if (this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice
        return `还差¥${diff}元起送`
      } else {
        return '去结算'
      }
    },
    payClass () {
      if (this.totalPrice < this.minPrice) {
        return 'noenough'
      } else {
        return 'enough'
      }
    },
    listShow () {
      if (!this.totalCount) {
        /* eslint-disable */
        this.foldFlag = false
        return false
      }
      let show = this.foldFlag
      if (show) {
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$refs.listContent, {
              click: true
            })
          } else {
            this.scroll.refresh()
          }
        })
      }
      return show
    }
  },
  methods: {
    drop (el) {
      // 触发一次事件就会将所有小球进行遍历
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i]
        if (!ball.show) { // 将false的小球放到dropBalls
          ball.show = true
          ball.el = el // 设置小球的el属性为一个dom对象
          this.dropBalls.push(ball)
          return // 此处return终结函数,这意味着每次触发事件,dropBalls内只增加一个ball
        }
      }
    },
    beforeEnter (el) { // 这里的el指的是小球的Dom,与drop(el)参数区分开
      let count = this.balls.length
      while (count--) {
        let ball = this.balls[count]
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect() // 获取小球的相对于视口的位移(小球高度)
          let x = rect.left - 32
          let y = -(window.innerHeight - rect.top - 22) // 计算出与小球初始位置垂直方向上的距离，y轴向上为负
          el.style.display = '' // 清空display
          el.style.webkitTransform = `translate3d(0,${y}px,0)`
          el.style.transform = `translate3d(0,${y}px,0)`
          // 处理内层动画
          let inner = el.getElementsByClassName('inner-hook')[0] // 使用inner-hook类来单纯被js操作
          inner.style.webkitTransform = `translate3d(${x}px,0,0)`
          inner.style.transform = `translate3d(${x}px,0,0)`
        }
      }
    },
    enter (el, done) {
      /* eslint-disable */
      let rf = el.offsetHeight // 触发重绘html
      this.$nextTick(() => { // 让动画效果异步执行,提高性能
        el.style.webkitTransform = 'translate3d(0,0,0)'
        el.style.transform = 'translate3d(0,0,0)'
        // 处理内层动画
        let inner = el.getElementsByClassName('inner-hook')[0] // 使用inner-hook类来单纯被js操作
        inner.style.webkitTransform = 'translate3d(0,0,0)'
        inner.style.transform = 'translate3d(0,0,0)'
        el.addEventListener('transitionend', done) // Vue为了知道过渡的完成，否则无法进入到afterEnter中
      })
    },
    afterEnter (el) {
      let ball = this.dropBalls.shift() // 完成一次动画就删除一个dropBalls的小球，否则触发N次事件，dropBalls则有N个元素
      if (ball) {
        ball.show = false
        el.style.display = 'none' // 隐藏小球
      }
    },
    toggleList () {
      if (!this.totalCount) { // 购物车没有商品的时候不可点击
        return
      }
      // 当前是收起状态就展开，展开状态就收起
      this.foldFlag = !this.foldFlag
    },
    clearCart (){
      this.selectFoods.forEach((food)=>{
        food.count = ''
      })
    },
    hideList () {
      this.foldFlag = !this.foldFlag; // 点击mark层，购物车详情列表被收回
    },
    pay() { //点击去结算
      if (this.totalPrice < this.minPrice) {
        return;
      }
      window.alert(`支付￥${this.totalPrice}元`);
      //window.alert('支付' + this.totalPrice + '元');
    }

  },
  components: {
    cartcontrol,
    BScroll
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin"
  .shopcart
    position: fixed
    left: 0
    bottom: 0
    z-index: 50
    width: 100%
    height: 48px
    background: #000
    .content
      display: flex
      background: #141d27
      color: rgba(255, 255, 255, 0.4)
      .content-left
        flex: 1
        .logo-wrapper
          display: inline-block
          vertical-align: top
          position: relative
          top: -10px
          margin: 0 12px
          padding: 6px
          width: 56px
          height: 56px
          box-sizing: border-box
          border-radius: 50%
          background: #141d27
          .logo
            width: 100%
            height: 100%
            border-radius: 50%
            text-align: center
            background: #2b343c
            &.highlight
              background: rgb(0, 160, 220)
            .icon-shopping_cart
              line-height: 44px
              font-size: 24px
              color: #80858a
              &.highlight
                color: #fff
        .num
          position: absolute
          top: 0
          right: 0
          width: 24px
          height: 16px
          line-height: 16px
          text-align: center
          border-radius: 16px
          font-size: 9px
          font-weight: 700
          color: #fff
          background: rgb(240, 20, 20)
          box-shadow: 0 4px 8px rgba(0, 0, 0, 0.4)
        .price
          display: inline-block
          vertical-align: top
          margin-top: 12px
          line-height: 24px
          padding-right: 12px
          box-sizing: border-box
          border-right: 1px solid rgba(255, 255, 255, 0.1)
          font-size: 16px
          font-weight: 700
          &.highlight
            color: #fff
        .desc
          display: inline-block
          vertical-align: top
          margin: 12px 0 0 12px
          line-height: 24px
          font-size: 10px
      .content-right
        flex: 0 0 105px
        width: 105px
        .pay
          height: 48px
          line-height: 48px
          text-align: center
          font-size: 12px
          font-weight: 700
          background: #2b333b
          &.noenough
            background: #2b333b
          &.enough
            background: #00b43c
    .ball-container
      .ball
        position fixed
        left 32px
        bottom 22px
        z-index 1
        transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
        .inner
          width 16px
          height 16px
          border-radius 50%
          background rgb(0, 160, 220)
          transition all 0.4s linear
    .shopcart-list
      position absolute
      left 0
      top 0
      z-index -1
      width 100%
      transform translate3d(0, -100%, 0) // 整个列表相对于当前自身的高度做一个偏移
      &.fold-enter-active, &.fold-leave-active
        transition: all .5s linear
        transform translate3d(0, -100%, 0) // 每个表项相对于当前自身的高度做一个偏移
      &.fold-enter, &.fold-leave-active
        transform translate3d(0, 0, 0)
      .list-header
        height 40px
        line-height 40px
        padding 0 18px
        background #f3f5f7
        border-bottom 1px solid rgba(7, 17, 27, 0.1)
        .title
          float left
          font-size 14px
          color rgb(7, 17, 27)
        .empty
          float right
          font-size 12px
          color rgb(0, 160, 220)
      .list-content
        padding 0 18px
        max-height 201px
        overflow hidden
        background #fff
        .food
          position relative
          padding 12px 0
          box-sizing border-box
          border-1px(rgba(7, 17, 27, 0.1))
          .name
            line-height 24px
            font-size 14px
            color rgb(7, 17, 27)
          .price
            position absolute
            right 90px
            bottom 12px
            line-height 24px
            font-size 14px
            font-weight 700
            color rgb(240, 20, 20)
          .cartcontrol-wrapper
            position absolute
            right 0
            bottom 6px

  .list-mask
    position fixed
    top 0
    left 0
    width 100%
    height 100%
    z-index 40 //z-index要小于shopcart的index
    backdrop-filter blur(10px)
    -webkit-backdrop-filter blur(10px)
    opacity 1
    background rgba(7, 17, 27, 0.6)
    &.fade-enter-active, &.fade-leave-active
      opacity 1
      transition: all 0.5s //设置缓动效果
      background rgba(7, 17, 27, 0.6)
    &.fade-enter, &.fade-leave-active
      opacity 0
  //
</style>
