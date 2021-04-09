<template>
  <div ref="wrapEle" id="app" class="box" :style="wrapStyle">
    <drag-item v-for="item in list" :wrap-size="wrapSize" :range="range" :outer.sync="item.data" :key="item.id"></drag-item>
  </div>
</template>

<script>
import DragItem from './dragItem'

export default {
  name: 'App',
  components: {
    DragItem
  },
  data () {
    return {
      list: [
        {
          id: 1,
          data: [0, 0, 4, 4]
        },
        {
          id: 2,
          data: [5, 0, 4, 4]
        },
        {
          id: 3,
          data: [0, 6, 4, 4]
        },
        {
          id: 3,
          data: [13, 6, 4, 4]
        }
      ],
      originSize: [0, 0],
      wrapStyle: {},
      range: [],
      minX: 0,
      minY: 0
    }
  },
  computed: {
    wrapSize () {
      return [this.originSize[0] - this.minX, this.originSize[1] - this.minY]
    }
  },
  watch: {
    originSize () {
      const r = 40
      const xRemainder = []
      const yRemainder = []
      for (let i = r; i < r + 10; i++) {
        xRemainder.push(this.originSize[0] % i)
      }
      for (let i = r; i < r + 10; i++) {
        yRemainder.push(this.originSize[1] % i)
      }
      this.minX = Math.min(...xRemainder)
      this.minY = Math.min(...yRemainder)
      this.range = [xRemainder.indexOf(this.minX) + r, yRemainder.indexOf(this.minY) + r]
      this.wrapStyle = {
        width: this.originSize[0] - this.minX + 'px',
        height: this.originSize[1] - this.minY + 'px'
      }
    }
  },
  mounted () {
    const el = this.$refs.wrapEle
    this.originSize = [el.clientWidth, el.clientHeight]
  }
}
</script>

<style lang="less">
body {
  background: #f2f2f2;
  width: 100vw;
  height: 100vh;
  margin: 0;
}
.box {
  position: relative;
  height: 600px;
  margin: 30px;
  background: #e8eced;
  border: 2px solid #e8eced;
}
</style>
