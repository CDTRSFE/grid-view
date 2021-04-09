<template>
    <div :style="itemStyle" class="drag-item">
      <div :style="bgStyle" class="bg"></div>
        <div
          ref="dragEle"
          :style="comStyle"
          class="content"
        >
          <slot></slot>
          <div ref="scaleEle" class="scale-btn"></div>
        </div>
    </div>
</template>

<script>
export default {
  name: 'DragItem',
  props: {
    outer: {
      type: Array,
      default: () => []
    },
    range: {
      type: Array,
      required: true
    },
    wrapSize: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      ox: 0,
      oy: 0,
      x: 0,
      y: 0,
      drag: false,
      scale: false
    }
  },
  computed: {
    // 父级位置和尺寸
    itemP () {
      if (this.range[0] === 0 || this.range[1] === 0) return
      return {
        left: this.outer[0] * this.range[0],
        top: this.outer[1] * this.range[1],
        width: this.outer[2] * this.range[0],
        height: this.outer[3] * this.range[1]
      }
    },
    // 父级样式
    itemStyle () {
      return {
        left: this.itemP.left + 'px',
        top: this.itemP.top + 'px',
        width: this.itemP.width + 'px',
        height: this.itemP.height + 'px'
      }
    },
    // 内容块的位置
    comStyle () {
      return {
        left: (this.drag ? this.delta.x : 0) + 'px',
        top: (this.drag ? this.delta.y : 0) + 'px',
        width: (this.scale ? this.delta.x + this.itemP.width : this.itemP.width) + 'px',
        height: (this.scale ? this.delta.y + this.itemP.height : this.itemP.height) + 'px'
      }
    },
    // 灰色背景位置
    bgStyle () {
      return {
        left: this.change.left + 'px',
        top: this.change.top + 'px',
        width: this.change.width + 'px',
        height: this.change.height + 'px'
      }
    },
    // 拖拽距离
    delta () {
      return {
        x: this.x - this.ox,
        y: this.y - this.oy
      }
    },
    // 拖拽结束 父级位置需要变化的值
    change () {
      const result = {}

      if (this.drag) {
        result.x = Math.floor(Math.abs(this.delta.x / this.range[0])) * this.range[0] * (this.delta.x < 0 ? -1 : 1)
        result.y = Math.floor(Math.abs(this.delta.y / this.range[1])) * this.range[1] * (this.delta.y < 0 ? -1 : 1)

        if (result.x + this.itemP.left < 0) result.x = -this.itemP.left
        if (result.y + this.itemP.top < 0) result.y = -this.itemP.top
        if (result.x + this.itemP.left > this.wrapSize[0] - this.itemP.width) {
          result.x = this.wrapSize[0] - this.itemP.width - this.itemP.left
        }
        if (result.y + this.itemP.top > this.wrapSize[1] - this.itemP.height) {
          result.y = this.wrapSize[1] - this.itemP.height - this.itemP.top
        }
      }

      if (this.scale) {
        result.x = Math.ceil(this.delta.x / this.range[0]) * this.range[0]
        result.y = Math.ceil(this.delta.y / this.range[1]) * this.range[1]

        if (result.x + this.itemP.width < 3 * this.range[0]) result.x = 3 * this.range[0] - this.itemP.width
        if (result.y + this.itemP.height < 3 * this.range[1]) result.y = 3 * this.range[1] - this.itemP.height
        if (result.x + this.itemP.width > this.wrapSize[0] - this.itemP.left) {
          result.x = this.wrapSize[0] - this.itemP.left - this.itemP.width
        }
        if (result.y + this.itemP.height > this.wrapSize[1] - this.itemP.top) {
          result.y = this.wrapSize[1] - this.itemP.top - this.itemP.height
        }
      }

      return {
        left: this.drag ? result.x : 0,
        top: this.drag ? result.y : 0,
        width: this.scale ? result.x + this.itemP.width : this.itemP.width,
        height: this.scale ? result.y + this.itemP.height : this.itemP.height
      }
    }
  },
  mounted () {
    this.bindEvent()
  },
  beforeDestroy () {
    this.unbindEvent()
  },
  methods: {
    bindEvent () {
      const body = document.body
      body.addEventListener('mousedown', this.down)
      body.addEventListener('mousemove', this.move)
      body.addEventListener('mouseup', this.up)
    },
    unbindEvent () {
      const body = document.body
      body.removeEventListener('mousedown', this.down)
      body.removeEventListener('mousemove', this.move)
      body.removeEventListener('mouseup', this.up)
    },
    down (e) {
      if (this.drag) return this.setP()
      if (this.scale) return this.setSize()
      if (e.target === this.$refs.dragEle) {
        this.drag = true
      }
      if (e.target === this.$refs.scaleEle) {
        this.scale = true
      }
      if (e.target !== this.$refs.dragEle && e.target !== this.$refs.scaleEle) return
      console.log('down', this.drag, this.scale)
      this.ox = e.x
      this.oy = e.y
      this.x = e.x
      this.y = e.y
    },
    move (e) {
      if (!this.drag && !this.scale) return
      this.x = e.x
      this.y = e.y
    },
    up () {
      this.drag && this.setP()
      this.scale && this.setSize()
    },
    setP () {
      const arr = this.outer.slice()
      arr[0] += this.change.left / this.range[0]
      arr[1] += this.change.top / this.range[1]
      this.$emit('update:outer', arr)
      this.drag = false
      this.ox = 0
      this.oy = 0
      this.x = 0
      this.y = 0
    },
    setSize () {
      const arr = this.outer.slice()
      arr[2] = this.change.width / this.range[0]
      arr[3] = this.change.height / this.range[1]
      this.$emit('update:outer', arr)
      this.scale = false
      this.ox = 0
      this.oy = 0
      this.x = 0
      this.y = 0
    }
  }
}
</script>

<style scoped lang="less">
.drag-item {
  position: absolute;
  &:hover {
    z-index: 3;
  }
}
.bg {
  background: rgba(0, 0, 0, 0.1);
}
.content, .bg {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
}
.content {
  background: #fff;
  z-index: 2;
  box-sizing: border-box;
  border: 2px solid #e8eced;
}
.scale-btn {
  position: absolute;
  right: 0;
  bottom: 0;
  width: 30px;
  height: 30px;
  background: #ddd;
  cursor: se-resize;
}
</style>
