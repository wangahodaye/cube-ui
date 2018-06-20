<template>
  <div class="cube-scroll-nav" :class="{'cube-scroll-nav_side': sideStyle}">
    <cube-sticky ref="sticky" :pos="scrollY" @change="stickyChangeHandler">
      <cube-scroll
        ref="scroll"
        :scroll-events="scrollEvents"
        :options="options"
        :data="data"
        @scroll="scrollHandler"
        @scroll-end="scrollEndHandler"
        @pulling-down="onPullingDown"
        @pulling-up="onPullingUp">
        <slot name="prepend"></slot>
        <div class="cube-scroll-nav-main">
          <cube-sticky-ele ref="navBarEle" ele-key="cube-scroll-nav-bar">
            <slot name="bar" :txts="barTxts" :labels="labels" :current="currentSticky">
              <cube-scroll-nav-bar
                :direction="barDirection"
                :txts="barTxts"
                :labels="labels"
                :current="currentSticky" />
            </slot>
          </cube-sticky-ele>
          <cube-sticky
            ref="pageSticky"
            :offset="pageStickyOffset"
            :pos="scrollY"
            @change="pageStickyChangeHandler">
            <div class="cube-scroll-nav-panels">
              <slot></slot>
            </div>
            <template slot="fixed" v-if="!sideStyle">
              <div></div>
            </template>
          </cube-sticky>
        </div>
        <template v-if="$slots.pullup || $scopedSlots.pullup" slot="pullup" slot-scope="props">
          <slot name="pullup"
            :pullUpLoad="props.pullUpLoad"
            :isPullUpLoad="props.isPullUpLoad">
          </slot>
        </template>
        <template v-if="$slots.pulldown || $scopedSlots.pulldown" slot="pulldown" slot-scope="props">
          <slot name="pulldown"
            :pullDownRefresh="props.pullDownRefresh"
            :pullDownStyle="props.pullDownStyle"
            :beforePullDown="props.beforePullDown"
            :isPullingDown="props.isPullingDown"
            :bubbleY="props.bubbleY">
          </slot>
        </template>
      </cube-scroll>
    </cube-sticky>
  </div>
</template>

<script type="text/ecmascript-6">
  import scrollMixin from '../../common/mixins/scroll'

  import CubeSticky from '../sticky/sticky.vue'
  import CubeStickyEle from '../sticky/sticky-ele.vue'
  import CubeScroll from '../scroll/scroll.vue'
  import CubeScrollNavBar from '../scroll-nav-bar/scroll-nav-bar.vue'

  const DIRECTION_H = 'horizontal'
  const DIRECTION_V = 'vertical'

  const COMPONENT_NAME = 'cube-scroll-nav'
  const EVENT_PULLING_DOWN = 'pulling-down'
  const EVENT_PULLING_UP = 'pulling-up'
  const EVENT_CHANGE = 'change'

  export default {
    name: COMPONENT_NAME,
    mixins: [scrollMixin],
    props: {
      data: {
        type: Array
      },
      sideStyle: {
        type: Boolean,
        default: false
      },
      current: {
        type: [String, Number],
        default: ''
      }
    },
    data() {
      return {
        scrollEvents: ['scroll', 'scroll-end'],
        scrollY: 0,
        labels: [],
        barTxts: [],
        stickyCurrent: this.current,
        active: this.current,
        pageStickyOffset: 0
      }
    },
    computed: {
      currentSticky() {
        const active = this.active
        const label = this.labels[0]
        return active || label
      },
      barDirection() {
        return this.sideStyle ? DIRECTION_V : DIRECTION_H
      }
    },
    watch: {
      current(newVal) {
        this.stickyCurrent = newVal
        this.active = newVal
        this.jumpTo(newVal)
      },
      active(newVal) {
        this.$emit(EVENT_CHANGE, newVal)
      }
    },
    beforeCreate() {
      this.isScrollNav = true
    },
    created() {
      this.navBar = null
      this.panels = []
    },
    mounted() {
      this.$nextTick(() => {
        if (this.sideStyle) {
          const el = this.$refs.pageSticky.$el.querySelector('.cube-sticky-fixed')
          this.$refs.scroll.$el.appendChild(el)
        }
        this.refresh()
        this.jumpTo(this.current)
      })
    },
    methods: {
      refresh() {
        this.navBar && this.navBar.refresh()
        this.$refs.sticky.refresh()
        this.$refs.pageSticky.refresh()
        this.pageStickyOffset = this.sideStyle ? 0 : this.$refs.navBarEle.$el.offsetHeight
      },
      setBar(bar) {
        this.navBar = bar
      },
      barChange(label) {
        this.active = label
        // waiting touchend
        this.$nextTick(() => {
          this.jumpTo(label)
        })
      },
      jumpTo(label, time = 300) {
        if (!label) {
          return
        }
        const labelEl = this.$refs.sticky.$el.querySelector(`.cube-scroll-nav-panel[data-panel-id="${label}"]`)
        if (labelEl) {
          this._jumping = true
          const offset = this.pageStickyOffset
          this.$refs.scroll.scrollToElement(labelEl, time, 0, this.sideStyle ? (offset + 1) : (-offset + 1))
        }
      },
      pageStickyChangeHandler(current) {
        if (current === -1) {
          current = this.labels[0]
        }
        this.stickyCurrent = current
        if (this._jumping) {
          return
        }
        this.active = current
      },
      stickyChangeHandler(current) {
        this.$nextTick(() => {
          this.navBar && this.navBar.refresh()
        })
      },
      scrollHandler(pos) {
        this.scrollY = -pos.y
        if (!this._jumping) {
          this.active = this.stickyCurrent
        }
      },
      scrollEndHandler() {
        this._jumping = false
      },
      forceUpdate() {
        this.$refs.scroll.forceUpdate()
      },
      onPullingUp() {
        this.$emit(EVENT_PULLING_UP)
      },
      onPullingDown() {
        this.$emit(EVENT_PULLING_DOWN)
      },
      addPanel(panel) {
        this.panels.push(panel)
        this.labels.push(panel.label)
        this.barTxts.push(panel.txt)
      },
      removePanel(panel) {
        const i = this.panels.indexOf(panel)
        this.panels.splice(i, 1)
        this.labels.splice(i, 1)
        this.barTxts.splice(i, 1)
      }
    },
    components: {
      CubeSticky,
      CubeStickyEle,
      CubeScroll,
      CubeScrollNavBar
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .cube-scroll-nav
    position: relative
    height: 100%
    overflow: hidden
    .cube-sticky-content
      height: 100%
  .cube-scroll-nav-main
    overflow: hidden
    > .cube-sticky
      position: static
  .cube-scroll-nav_side
    > .cube-sticky
      display: flex
      > .cube-scroll-wrapper
        flex: 1
      > .cube-sticky-fixed
        position: relative
        height: 100%
        order: -1
    .cube-scroll-nav-main
      > .cube-sticky-ele
        float: left
      > .cube-sticky
        overflow: hidden
</style>