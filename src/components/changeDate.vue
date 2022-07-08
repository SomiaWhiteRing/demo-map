<template>
  <div class="hello">
    <!-- <button @click="changeYear">用力刷新</button> -->
    <div ref="timeline" class="timeline"/>
  </div>
</template>

<script>
import echarts from 'echarts'
export default {
  name: 'HelloWorld',
  props: {
    years: Array
  },
  data () {
    return {
      playState: true
    }
  },
  mounted () {
    this.drawTimeline()
  },
  methods: {
    changeYear (year) {
      this.$emit('changeYear', year, this.playState)
    },
    drawTimeline () {
      const timeLine = echarts.init(this.$refs.timeline)
      const optionTimeline = {
        timeline: {
          data: this.years,
          axisType: 'category',
          autoPlay: true,
          // show: false,
          playInterval: 315576000000,
          left: '10%',
          right: '10%',
          bottom: '3%',
          width: '80%',
          //  height: null,
          label: {
            normal: {
              textStyle: {
                color: '#ddd'
              }
            },
            emphasis: {
              textStyle: {
                color: '#fff'
              }
            }
          },
          symbolSize: 10,
          lineStyle: {
            color: '#555'
          },
          checkpointStyle: {
            borderColor: '#777',
            borderWidth: 2
          },
          controlStyle: {
            showNextBtn: true,
            showPrevBtn: true,
            normal: {
              color: '#666',
              borderColor: '#666'
            },
            emphasis: {
              color: '#aaa',
              borderColor: '#aaa'
            }
          }
        }
      }
      timeLine.setOption(optionTimeline)
      window.addEventListener('resize', () => { timeLine.resize() })

      timeLine.on('timelineChanged', (params) => { // 将年份的变更传达给父组件
        console.log(timeLine)
        this.$emit('changeYear', this.years[params.currentIndex])
      })
      timeLine.on('timelinePlayChanged', (params) => { // 将播放状态的变更传达给父组件
        // console.log(params)
        this.playState = params.playState
        this.$emit('changePlay', params.playState)
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.hello{
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.timeline{
  position: absolute;
  z-index: 10;
  width: calc(100vw - 16px);
  // height: calc(100vh - 16px);
  height: 100px;
  bottom: 3%;
}
</style>
