<!-- 风险类型分布雷达图 -->
<template>
  <div class='com-container'>
    <div class='com-chart' ref='radar_ref'></div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
export default {
  data () {
    return {
      chartInstance: null,
      allData: null
    }
  },
  created () {
    // 在组件创建完成之后 进行回调函数的注册
    this.$socket.registerCallBack('radarData', this.getData)
  },
  mounted () {
    this.$socket.send({
      action: 'getData',
      socketType: 'radarData',
      chartName: 'radar',
      value: ''
    })
    this.initChart()
    // this.getData()
    window.addEventListener('resize', this.screenAdapter)
    this.screenAdapter()
  },
  destroyed () {
    window.removeEventListener('resize', this.screenAdapter)
    this.$socket.unRegisterCallBack('radarData')
  },
  methods: {
    getData (ret) {
      this.allData = ret
      // console.log(this.allData)
      this.updateChart()
    },
    initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.radar_ref, this.theme)
      const initOption = {
        title: {
          text: '▎风险类型分布',
          left: 20,
          top: 20
        },
        tooltip: {
          // 避免tool被边缘遮挡
          confine: true
        },
        radar: {
          radius: '60%',
          center: ['50%', '55%'],
          splitArea: {
            areaStyle: {
              opacity: '0.5'
            }
          }
        },
        series: [
          {
            type: 'radar',
            lineStyle: {
              color: 'rgba(17,150,238,0.9)'
            },
            areaStyle: {
              color: 'rgba(17,150,238,0.5)'
            }
          }
        ]
      }
      this.chartInstance.setOption(initOption)
    },
    updateChart () {
      // 处理图表需要的数据
      const data = this.allData.data
      const indicator = this.allData.indicator
      const seriesArr = {
        type: 'radar',
        label: {
          position: 'center'
        },
        data: [
          {
            name: '',
            value: data[0].value,
            itemStyle: {
            }
          }
        ]
      }
      const dataOption = {
        series: seriesArr,
        radar: {
          indicator: indicator
        },
        tooltip: {
          formatter: function (params) {
            let result = ''
            for (let i = 0; i < indicator.length; i++) {
              result += indicator[i].name + ' : ' + params.data.value[i] + '<br/>'
            }
            return result
          }
        }
      }
      this.chartInstance.setOption(dataOption)
    },
    screenAdapter () {
      const titleFontSize = this.$refs.radar_ref.offsetWidth / 100 * 3.6
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: titleFontSize
          }
        },
        series: [
          {
            radius: this.titleFontSize * 6,
            type: 'radar',
            label: {
              fontSize: titleFontSize / 2
            }
          }
        ]
      }
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    }
  },
  computed: {
    ...mapState(['theme'])
  },
  watch: {
    theme () {
      console.log('主题切换了')
      this.chartInstance.dispose() // 销毁当前的图表
      this.initChart() // 重新以最新的主题名称初始化图表对象
      this.screenAdapter() // 完成屏幕的适配
      this.updateChart() // 更新图表的展示
    }
  }
}
</script>

<style lang='less' scoped>
</style>
