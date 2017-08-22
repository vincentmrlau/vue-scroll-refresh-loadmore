<template>
  <div
    class="vincent-scroll"
    ref="vincentScroll"
    @touchstart="start"
    @touchmove="move"
    @touchend="end"
  >
    <div ref="vincentScrollInner">
      <p class="vincent-scroll-header" :style="headerStyle">{{headerText}}</p>
      <div>
        <slot></slot>
      </div>
      <p class="vincent-scroll-footer" :style="footerStyle">{{footerText}}</p>
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      pulldownText: {
        type: String,
        default: 'Pull down to refresh'
      },
      onPulldownText: {
        type: String,
        default: 'Release to refresh'
      },
      pulldownRefreshingText: {
        type: String,
        default: 'loading...'
      },
      scrollRefreshingText: {
        type: String,
        default: 'loading more...'
      },
      headerMaxHeight: {
        type: Number,
        default: 50
      },
      footerMaxHeight: {
        type: Number,
        default: 30
      },
      refreshThreshold: {
        type: Number,
        default: 0.5
      },
      loadMoreThreshold: {
        type: Number,
        default: 10
      },
      headerRefreshDisabled: {
        type: Boolean,
        default: false
      },
      footerLoadDisabled: {
        type: Boolean,
        default: false
      }
    },
    data () {
      return {
        /*
        * 0 - before pull down
        * 1 - pulling down
        * 2 - enough to release
        * 3 - refreshing
        * 4 - finish refresh
        * */
        headerStatus: 0,
        /*
        * 0 - before pull down
        * 1 - loading
        * 2 - finish
        * */
        footerStatus: 0,
        headerHeight: 0,
        footerHeight: 0,
        headerAnimate: false,
        touchY: 0,
        moveY: 0,
        scrollTemp: undefined
      }
    },
    computed: {
      headerText () {
        switch (this.headerStatus) {
          case 0:
            return this.$props.pulldownText
          case 1:
            return this.$props.pulldownText
          case 2:
            return this.$props.onPulldownText
          case 3:
            return this.$props.pulldownRefreshingText
          default:
            return this.$props.pulldownText
        }
      },
      footerText () {
        return this.$props.scrollRefreshingText
      },
      headerStyle () {
        let style = {}
        if (this.headerAnimate) {
          style.transition = 'all 0.2s'
        }
        if (this.headerHeight >= this.headerMaxHeight) {
          this.headerHeight = this.headerMaxHeight
        }
        style.height = this.headerHeight + 'px'
        style.lineHeight = this.headerHeight + 'px'
        return style
      },
      footerStyle () {
        let style = {}
        style.transition = 'all 0.2s'
        style.height = this.footerHeight + 'px'
        style.lineHeight = this.footerHeight + 'px'
        return style
      }
    },
    methods: {
      start (e) {
        this.$data.headerAnimate = false
        if (this.$data.headerStatus === 3) {
          // refreshing
          return
        } else {
          this.$data.touchY = e.touches[0].pageY
        }
      },
      move (e) {
        if (e.touches === undefined) {
          return
        }
        this.$data.moveY = e.touches[0].pageY - this.$data.touchY
        if (this.$data.moveY < -20) {
          if (this.$props.footerLoadDisabled) {
            return
          }
          let scrollTop = this.$refs.vincentScroll.scrollTop
          let innerHeight = this.$refs.vincentScrollInner.offsetHeight
          if (this.$data.footerStatus === 1) {
            // loading
            this.$data.scrollTemp = scrollTop
            return
          } else {
            if (this.$data.footerStatus === 1) {
              // loading
              return
            }
            if (innerHeight - this.$refs.vincentScroll.offsetHeight <= scrollTop + this.$props.loadMoreThreshold) {
              this.$data.footerHeight = this.$props.footerMaxHeight
              this.$data.footerStatus = 1
              this.$emit('on-loadmore')
            }
            return
          }
        } else {
          if (this.$refs.vincentScroll.scrollTop === 0 ||
            this.$data.headerStatus !== 1 ||
            this.$data.headerStatus !== 2) {
            if (this.$data.headerStatus === 3) {
              // refreshing
              return
            } else {
              // pulling down
              this.$data.headerStatus = 1
              if (this.$data.moveY > 0) {
                this.$data.headerHeight = this.$data.moveY
                if (this.$data.headerHeight >= this.$props.refreshThreshold * this.$props.headerMaxHeight) {
                  this.$data.headerStatus = 2
                } else {
                  this.$data.headerStatus = 1
                }
                let seft = this
                this.$nextTick(() => {
                  seft.$refs.vincentScroll.scrollTop = 0
                })
                return
              } else {
                return
              }
            }
          } else {
            return
          }
        }
      },
      end () {
        if (this.$data.moveY > 0) {
          switch (this.$data.headerStatus) {
            case 0:
              return
            case 1:
              this.$data.headerAnimate = true
              this.$data.headerHeight = 0
              this.$data.headerStatus = 0
              this.$nextTick(() => {
                this.$refs.vincentScroll.scrollTop = 0
              })
              return
            case 2:
              this.$data.headerAnimate = true
              this.$data.headerHeight = this.$props.headerMaxHeight
              this.$data.headerStatus = 3
              this.$nextTick(() => {
                this.$refs.vincentScroll.scrollTop = 0
              })
              this.$emit('on-refresh')
              return
            case 3:
              return
          }
        } else {
          // pull up
        }
      },
      refreshDone () {
        this.$data.headerAnimate = true
        this.$data.headerHeight = 0
        this.$data.headerStatus = 4
        this.$nextTick(() => {
          this.$refs.vincentScroll.scrollTop = 0
        })
      },
      loadmoreDone () {
        this.$data.footerStatus = 3
        this.$data.footerHeight = 0
      }
    }
  }
</script>

<style>
  .vincent-scroll{
    position: absolute;
    width: 100%;
    height: 100%;
    overflow: auto;
    -webkit-overflow-scrolling: touch;
  }
  .vincent-scroll-header,.vincent-scroll-footer{
    text-align: center;
    padding: 0px;
    margin: 0px;
    overflow: hidden;
  }
</style>
