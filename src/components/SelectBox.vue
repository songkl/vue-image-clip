<template>
  <div ref="cropWrap" class="crop-wrap">
    <div class="shadow-box" :style="recStyle">
      <img :src="img" class="shadow-img" :style="imgStyle">
    </div>
    <div ref="cropBox" class="crop-box" :class="showBox ? 'show': ''" :style="recStyle">
      <span ref="lt" type="drag-lt" class="drag-point point-lt"></span>
      <span ref="lb" type="drag-lb" class="drag-point point-lb"></span>
      <span ref="rt" type="drag-rt" class="drag-point point-rt"></span>
      <span ref="rb" type="drag-rb" class="drag-point point-rb"></span>
    </div>
  </div>
</template>

<script>
const touch = typeof(window) && typeof(window.ontouchstart)!='undefined'
const up_event_name = touch?'touchend':'mouseup'
const down_event_name = touch?'touchstart':'mousedown'
const move_event_name = touch?'touchmove':'mousemove'

  export default {
    props: {
      ratio: {},
      img: {},
      srcSize: {}
    },
    data () {
      return {
        rec: {
          w: 0, h: 0, l: 0, t: 0
        },
        pl: 0,
        pt: 0,
        action: '',
        actionPoint: {x: 0, y: 0},
        referPoint: {x: 0, y: 0},
        $rec: null
      }
    },
    computed: {
      showBox () {
        return this.rec.w && this.rec.h
      },
      imgStyle () {
        return `width:${this.srcSize.w}px;height:${this.srcSize.h}px;top:${-this.rec.t}px;left:${-this.rec.l}px;`
      },
      recStyle () {
        return `width:${this.rec.w}px;height:${this.rec.h}px;left:${this.rec.l}px;top:${this.rec.t}px;`
      }
    },
    mounted () {

      window.addEventListener(up_event_name, this.disableDrag)
      window.addEventListener(move_event_name, this.updateRec)


     this.$refs.cropWrap.addEventListener(down_event_name, this.wrapMouseDown)
     this.$refs.cropBox.addEventListener(down_event_name, this.boxMouseDown)

     this.$refs.lt.addEventListener(down_event_name, this.pointMouseDown)
     this.$refs.lb.addEventListener(down_event_name, this.pointMouseDown)
     this.$refs.rt.addEventListener(down_event_name, this.pointMouseDown)
     this.$refs.rb.addEventListener(down_event_name, this.pointMouseDown)

    },
    beforeDestroy () {
      window.removeEventListener(up_event_name, this.disableDrag)
      window.removeEventListener(move_event_name, this.updateRec)


     this.$refs.cropWrap.removeEventListener(down_event_name, this.wrapMouseDown)
     this.$refs.cropBox.removeEventListener(down_event_name, this.boxMouseDown)

     this.$refs.lt.removeEventListener(down_event_name, this.pointMouseDown)
     this.$refs.lb.removeEventListener(down_event_name, this.pointMouseDown)
     this.$refs.rt.removeEventListener(down_event_name, this.pointMouseDown)
     this.$refs.rb.removeEventListener(down_event_name, this.pointMouseDown)

    },
    methods: {
      getLeft (el) {
        let left = el.offsetLeft
        let parent = el.offsetParent

        while (parent) {
          left += parent.offsetLeft
          parent = parent.offsetParent
        }
        return left
      },
      getTop (el) {
        let top = el.offsetTop
        let parent = el.offsetParent

        while (parent) {
          top += parent.offsetTop
          parent = parent.offsetParent
        }
        return top
      },
      initAction (name, x, y) {
        this.action = name
        this.pl = this.getLeft(this.$el)
        this.pt = this.getTop(this.$el)
        this.actionPoint = {x, y}
        this.referPoint = {x: this.rec.l, y: this.rec.t}

        if (name === 'drag-lt') {
          this.referPoint = {x: this.rec.l + this.rec.w, y: this.rec.t + this.rec.h}
        } else if (name === 'drag-lb') {
          this.referPoint = {x: this.rec.l + this.rec.w, y: this.rec.t}
        } else if (name === 'drag-rt') {
          this.referPoint = {x: this.rec.l, y: this.rec.t + this.rec.h}
        } else if (name === 'drag-rb') {
          this.referPoint = {x: this.rec.l, y: this.rec.t}
        }
      },
      pointMouseDown (event) {
        let type = event.target.getAttribute('type')

        let e = event

        if(touch){
            e = event.touches[0]
        }

        this.initAction(type, e.pageX, e.pageY)
        event.stopPropagation()
      },
      boxMouseDown (event) {

        let e = event

        if(touch){
           e = event.touches[0]
        }

        this.initAction('move', e.pageX, e.pageY)
        event.stopPropagation()
      },
      wrapMouseDown (event) {
        if (this.rec.w && this.rec.h) {
          return
        }

        let e = event

        if(touch){
            e = event.touches[0]
        }

        this.initAction('cross', e.pageX, e.pageY)
        this.rec = {
          w: 0,
          h: 0,
          l: e.pageX - this.pl,
          t: e.pageY - this.pt
        }
        event.stopPropagation()
      },
      disableDrag () {
        if (this.action) {
          this.action = ''
          this.$emit('selectEnd')
        }
      },
      clearRec () {
        this.action = ''
        this.rec = {w: 0, h: 0, l: 0, t: 0}
      },
      updateRec (event) {
        if (!this.action) {
          return
        }

        let e = event

        if(touch){
            e = event.touches[0]
        }

        const elWidth = this.$el.offsetWidth
        const elHeight = this.$el.offsetHeight
        const dx = e.pageX - this.actionPoint.x
        const dy = e.pageY - this.actionPoint.y
        const x = e.pageX
        const y = e.pageY
        let w = 0
        let h = 0
        let t = 0
        let l = 0

        if (dx === 0 && dy === 0) {
          return
        }

        if (this.action === 'move') {
          t = dy + this.referPoint.y
          l = dx + this.referPoint.x

          if (t <= 0) {
            t = 0
          } else if (t + this.rec.h >= elHeight) {
            t = elHeight - this.rec.h
          }

          if (l <= 0) {
            l = 0
          } else if (l + this.rec.w >= elWidth) {
            l = elWidth - this.rec.w
          }

          this.rec.l = l
          this.rec.t = t
        } else if (this.action === 'cross') {
          if (dx > 0 && dy > 0) {
            w = dx + this.rec.l >= elWidth ? elWidth - this.rec.l : dx
            h = w / this.ratio

            if (h + this.rec.t > elHeight) {
              h = elHeight - this.rec.t
              w = h * this.ratio
            }
            this.rec.w = w
            this.rec.h = h
          } else if (dx > 0 && dy < 0) {
            w = dx + this.referPoint.x >= elWidth ? elWidth - this.referPoint.x : dx
            h = w / this.ratio

            if (h >= this.referPoint.y) {
              h = this.referPoint.y
              w = h * this.ratio
            }

            this.rec.t = this.referPoint.y - h
            this.rec.w = w
            this.rec.h = h
          } else if (dx < 0 && dy < 0) {
            w = dx + this.referPoint.x <= 0 ? this.referPoint.x : -dx
            h = w / this.ratio

            if (h >= this.referPoint.y) {
              h = this.referPoint.y
              w = h * this.ratio
            }

            this.rec.t = this.referPoint.y - h
            this.rec.l = this.referPoint.x - w
            this.rec.w = w
            this.rec.h = h
          } else if (dx < 0 && dy > 0) {
            w = dx + this.referPoint.x <= 0 ? this.referPoint.x : -dx
            h = w / this.ratio

            if (h + this.referPoint.y >= elHeight) {
              h = elHeight - this.referPoint.y
              w = h * this.ratio
            }

            this.rec.l = this.referPoint.x - w
            this.rec.w = w
            this.rec.h = h
          }
        } else if (this.action === 'drag-lt' || this.action === 'drag-rt'
          || this.action === 'drag-lb' || this.action === 'drag-rb') {
          w = x - (this.referPoint.x + this.pl)
          h = y - (this.referPoint.y + this.pt)
          if (w < 0 && h < 0) {
            w = w * -1 >= this.referPoint.x ? this.referPoint.x : w * -1
            h = w / this.ratio

            if (h >= this.referPoint.y) {
              h = this.referPoint.y
              w = h * this.ratio
            }
            this.rec.l = this.referPoint.x - w
            this.rec.t = this.referPoint.y - h
          } else if (w < 0 && h > 0) {
            w = w * -1 >= this.referPoint.x ? this.referPoint.x : w * -1
            h = w / this.ratio

            if (h >= elHeight - this.referPoint.y) {
              h = elHeight - this.referPoint.y
              w = h * this.ratio
            }
            this.rec.l = this.referPoint.x - w
            this.rec.t = this.referPoint.y
          } else if (w > 0 && h < 0) {
            w = w >= elWidth - this.referPoint.x ? elWidth - this.referPoint.x : w
            h = w / this.ratio

            if (h >= this.referPoint.y) {
              h = this.referPoint.y
              w = h * this.ratio
            }
            this.rec.l = this.referPoint.x
            this.rec.t = this.referPoint.y - h
          } else if (w > 0 && h > 0) {
            w = w >= elWidth - this.referPoint.x ? elWidth - this.referPoint.x : w
            h = w / this.ratio

            if (h >= elHeight - this.referPoint.y) {
              h = elHeight - this.referPoint.y
              w = h * this.ratio
            }
            this.rec.l = this.referPoint.x
            this.rec.t = this.referPoint.y
          }

          this.rec.w = w
          this.rec.h = h
        }
        this.$emit('selectChange')
      }
    }
  }
</script>

<style scoped>
  .crop-wrap {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 10;
    cursor: crosshair;
  }

  .crop-box {
    position: absolute;
    display: none;
    top: 0;
    left: 0;
    width: 0;
    height: 0;
    cursor: move;
    border: 1px solid rgba(255,255,255,.2);
    z-index: 2;
    box-sizing: border-box;
  }

  .crop-box.show {
    display: block;
  }

  .drag-point {
    display: inline-block;
    width: 30px;
    height: 30px;
    max-width: 20%;
    max-height: 20%;
    border: 1px solid rgba(0,0,0,.5);
    position: absolute;
    box-sizing: border-box;
  }

  .point-lt {
    top: -2px;
    left: -2px;
    cursor: nw-resize;
    border-bottom-color: transparent;
    border-right-color: transparent;

    cursor: nw-resize;
  }

  .point-lb {
    left: -2px;
    bottom: -2px;
    border-top-color: transparent;
    border-right-color: transparent;
    cursor: sw-resize;
  }

  .point-rt {
    right: -2px;
    top: -2px;
    border-bottom-color: transparent;
    border-left-color: transparent;
    cursor: ne-resize;
  }

  .point-rb {
    right: -2px;
    bottom: -2px;
    border-top-color: transparent;
    border-left-color: transparent;
    cursor: se-resize;
  }

  .shadow-box {
    position: absolute;
    top: 0;
    left: 0;
    overflow: hidden;
    z-index: 1;
  }

  .shadow-img {
    position: absolute;
    top: 0;
    left: 0;
  }

  .shadow-box::selection, .shadow-img::selection {
    background-color: transparent;
  }
</style>
