<template>
  <!--为echarts准备一个具备大小的容器dom-->
  <div id="main">
    <changeDate :years="years" @changeYear="changeYear" @changePlay="changePlay"/>
    <div id="myChart" class="map" />
    <div class="mapChoose">
      <span v-for="(item, index) in parentInfo" :key="item.code">
        <span class="title" @click="chooseArea(item, index)">{{
          item.cityName == "全国" ? "中国" : item.cityName
        }}</span>
        <span class="icon" v-show="index + 1 != parentInfo.length">></span>
      </span>
    </div>
  </div>
</template>
<script>
import echarts from 'echarts'
import changeDate from '@/components/changeDate'
// import map from '../../static/china.json'
export default {
  name: 'Sandbox',
  components: {
    changeDate
  },
  data () {
    return {
      // geoGpsMap: {
      //   1: [127.9688, 45.368],
      //   2: [116.4551, 40.2539],
      //   3: [109.1162, 34.2004],
      //   4: [113.12244, 23.009505],
      //   5: [87.9236, 43.5883],
      //   6: [91.11, 29.97]
      // },
      geoGpsMap: {}, // 起始点的中心坐标
      geoCoordMap: {
        // 台湾: [121.5135, 25.0308],
        // 黑龙江: [127.9688, 45.368],
        // 内蒙古: [110.3467, 41.4899],
        // 吉林: [125.8154, 44.2584],
        // 北京市: [116.4551, 40.2539],
        // 辽宁: [123.1238, 42.1216],
        // 河北: [114.4995, 38.1006],
        // 天津: [117.4219, 39.4189],
        // 山西: [112.3352, 37.9413],
        // 陕西: [109.1162, 34.2004],
        // 甘肃: [103.5901, 36.3043],
        // 宁夏: [106.3586, 38.1775],
        // 青海: [101.4038, 36.8207],
        // 新疆: [87.9236, 43.5883],
        // 西藏: [91.11, 29.97],
        // 四川: [103.9526, 30.7617],
        // 重庆: [108.384366, 30.439702],
        // 山东: [117.1582, 36.8701],
        // 河南: [113.4668, 34.6234],
        // 江苏: [118.8062, 31.9208],
        // 安徽: [117.29, 32.0581],
        // 湖北: [114.3896, 30.6628],
        // 浙江: [119.5313, 29.8773],
        // 福建: [119.4543, 25.9222],
        // 江西: [116.0046, 28.6633],
        // 湖南: [113.0823, 28.2568],
        // 贵州: [106.6992, 26.7682],
        // 云南: [102.9199, 25.4663],
        // 广东: [113.12244, 23.009505],
        // 广西: [108.479, 23.1152],
        // 海南: [110.3893, 19.8516],
        // 上海: [121.4648, 31.2891]
      }, // 全部省份的名称与中心坐标
      colors: [
        [
          '#1DE9B6',
          '#F46E36',
          '#04B9FF',
          '#5DBD32',
          '#FFC809',
          '#FB95D5',
          '#BDA29A',
          '#6E7074',
          '#546570',
          '#C4CCD3'
        ],
        [
          '#37A2DA',
          '#67E0E3',
          '#32C5E9',
          '#9FE6B8',
          '#FFDB5C',
          '#FF9F7F',
          '#FB7293',
          '#E062AE',
          '#E690D1',
          '#E7BCF3',
          '#9D96F5',
          '#8378EA',
          '#8378EA'
        ],
        [
          '#DD6B66',
          '#759AA0',
          '#E69D87',
          '#8DC1A9',
          '#EA7E53',
          '#EEDD78',
          '#73A373',
          '#73B9BC',
          '#7289AB',
          '#91CA8C',
          '#F49F42'
        ]
      ], // 展示的色彩列表
      colorIndex: 0, // 当前展示的色彩
      // province: ['福建', '广东', '上海', '台湾', '吉林'],
      province: [], // 省份的列表
      mapData: [], // 每个省下辖的数据
      barData: [], // 柱状图的参数
      barLegend: [], // 柱状图的名称
      craeteNum: 6, // 生成的案例起始点的数量
      years: ['2018', '2019', '2020', '2021', '2022'], // 年份表对应的年份
      year: '2018', // 当前timeline走到的年份
      // 引入Amap的部分
      geoJson: {
        features: []
      },
      parentInfo: [{
        cityName: '全国',
        code: 100000
      }],
      pointData: [] // 点的数据
    }
  },
  mounted () {
    this.getGeoJson(100000)
  },
  methods: {
    getGeoJson (adcode) {
      const that = this
      // eslint-disable-next-line no-undef
      AMapUI.loadUI(['geo/DistrictExplorer'], DistrictExplorer => {
        var districtExplorer = new DistrictExplorer()
        districtExplorer.loadAreaNode(adcode, function (error, areaNode) {
          if (error) {
            console.error(error)
            return
          }
          const Json = areaNode.getSubFeatures()
          // 如果是全国地图，则把福建置于最前
          if (adcode === 100000) {
            Json.unshift(Json[12])
            Json.splice(13, 1)
          }
          // console.log(Json)
          if (Json.length > 0) {
            that.geoJson.features = Json
          } else if (Json.length === 0) {
            that.geoJson.features = that.geoJson.features.filter(
              item => item.properties.adcode === adcode
            )
            if (that.geoJson.features.length === 0) return
          }
          // 如果是全国地图，则把福建置于第一个
          that.getMapData()
        })
      })
    },
    getMapData () {
      const pointData = []
      this.geoJson.features.forEach((j) => {
        pointData.push({
          name: j.properties.name,
          value: [j.properties.center[0], j.properties.center[1]],
          cityCode: j.properties.adcode
        })
        this.geoCoordMap[j.properties.name] = j.properties.center
      })
      this.pointData = pointData
      this.drawMap()
      this.createProvinceList()
      this.setData()
    },
    createProvinceList () {
      const craeteNum = Object.keys(this.geoCoordMap).length
      for (let k = 0; k < craeteNum; k++) {
        // const i = Math.floor(Math.random() * Object.keys(this.geoCoordMap).length)
        const i = k
        this.province.push(Object.keys(this.geoCoordMap)[i])
        this.geoGpsMap[k] = this.geoCoordMap[Object.keys(this.geoCoordMap)[i]]
      }
      this.mapData = JSON.parse(JSON.stringify(new Array(craeteNum).fill([])))
    },
    setData () {
      // console.log('this.geoCoordMap:', this.geoCoordMap)
      const craeteNum = Object.keys(this.geoCoordMap).length
      for (const key in this.geoCoordMap) {
        for (let k = 0; k < craeteNum; k++) {
          if ((Math.random() > 0.7 && this.province[k] !== key) || (this.mapData[k].length === 0 && this.province[k] !== key)) {
            this.mapData[k].push({
              province: this.province[k],
              name: key,
              value: this.randomNum(100, 300)
            })
          }
        }
      }
      // console.log('this.mapData', this.mapData)
      for (let i = 0; i < this.mapData.length; i++) {
        this.barData.push([])
        this.barLegend.push([])
        for (var j = 0; j < this.mapData[i].length; j++) {
          this.barData[i].push(this.mapData[i][j])
          this.barLegend[i].push(this.mapData[i][j].name)
        }
      }
    },
    drawMap (mapData, pointData, sum) {
      this.$nextTick(function () {
        // console.log(map)
        // console.log(this.geoJson)
        // this.$echarts.registerMap('map', map)
        this.$echarts.registerMap('map', this.geoJson)
        // 基于准备好的dom，初始化echarts实例
        const myChart = echarts.init(document.getElementById('myChart'))
        // 设置option
        const optionXyMap01 = {
          timeline: {
            data: this.province,
            axisType: 'category',
            autoPlay: true,
            show: false,
            playInterval: 5000,
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
          },
          baseOption: {
            animation: false,
            animationDuration: 1000,
            animationEasing: 'quadraticInOut',
            animationDurationUpdate: 1000,
            animationEasingUpdate: 'cubicInOut',
            grid: {
              right: '1%',
              top: '15%',
              bottom: '10%',
              width: '20%'
            },
            // tooltip: {
            //   trigger: 'axis', // hover触发器
            //   axisPointer: { // 坐标轴指示器，坐标轴触发有效
            //     type: 'shadow', // 默认为直线，可选为：'line' | 'shadow'
            //     shadowStyle: {
            //       color: 'rgba(150,150,150,0.1)' // hover颜色
            //     }
            //   }
            // },
            geo: {
              show: true,
              map: 'map',
              roam: true,
              zoom: 1.2,
              center: this.parentInfo.length === 1 ? ['118.83531246', '32.0267395887'] : false,
              label: {
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                normal: {
                  borderColor: 'rgba(147, 235, 248, 1)',
                  borderWidth: 1,
                  areaColor: {
                    type: 'radial',
                    x: 0.5,
                    y: 0.5,
                    r: 0.8,
                    colorStops: [{
                      offset: 0,
                      color: 'rgba(147, 235, 248, 0)' // 0% 处的颜色
                    }, {
                      offset: 1,
                      color: 'rgba(147, 235, 248, .2)' // 100% 处的颜色
                    }],
                    globalCoord: false // 缺省为 false
                  },
                  shadowColor: 'rgba(128, 217, 248, 1)',
                  // shadowColor: 'rgba(255, 255, 255, 1)',
                  shadowOffsetX: -2,
                  shadowOffsetY: 2,
                  shadowBlur: 10
                },
                emphasis: {
                  areaColor: '#389BB7',
                  borderWidth: 0
                }
              }
            }
          },
          options: []

        }
        // 推入数据
        for (var n = 0; n < this.province.length; n++) {
          optionXyMap01.options.push({
            backgroundColor: '#051b4a',
            title: [{
              // text: '地图',
              // subtext: '内部数据请勿外传',
              // left: 'center',
              // textStyle: {
              //   color: '#fff'
              // }
            },
            {
              id: 'statistic',
              text: this.year + '年' + this.province[n] + '流出情况',
              left: '75%',
              top: '3%',
              textStyle: {
                color: '#fff',
                fontSize: 24
              }
            }
            ],
            xAxis: {
              type: 'value',
              scale: true,
              position: 'top',
              min: 0,
              boundaryGap: false,
              splitLine: {
                show: false
              },
              axisLine: {
                show: false
              },
              axisTick: {
                show: false
              },
              axisLabel: {
                margin: 2,
                textStyle: {
                  color: '#aaa'
                }
              }
            },
            yAxis: {
              type: 'category',
              //  name: 'TOP 20',
              nameGap: 16,
              axisLine: {
                show: true,
                lineStyle: {
                  color: '#ddd'
                }
              },
              axisTick: {
                show: false,
                lineStyle: {
                  color: '#ddd'
                }
              },
              axisLabel: {
                interval: 0,
                textStyle: {
                  color: '#ddd'
                }
              },
              data: this.barLegend[n]
            },
            series: [
              // 散点图
              {
                // 文字和标志
                name: 'light',
                type: 'scatter',
                coordinateSystem: 'geo',
                data: this.convertData(this.mapData[n]),
                symbolSize: function (val) {
                  return val[2] / 10
                },
                label: {
                  normal: {
                    formatter: '{b}',
                    position: 'right',
                    show: true
                  },
                  emphasis: {
                    show: true
                  }
                },
                itemStyle: {
                  normal: {
                    color: this.colors[this.colorIndex][n]
                  }
                }
              },
              // 地图
              {
                type: 'map',
                map: 'map',
                geoIndex: 0,
                zoom: 1.3,
                // aspectScale: 0.75, // 长宽比
                showLegendSymbol: false, // 存在legend时显示
                label: {
                  normal: {
                    show: false
                  },
                  emphasis: {
                    show: false,
                    textStyle: {
                      color: '#fff'
                    }
                  }
                },
                roam: true,
                itemStyle: {
                  normal: {
                    areaColor: '#031525',
                    borderColor: '#FFFFFF'
                  },
                  emphasis: {
                    areaColor: '#2B91B7'
                  }
                },
                animation: false
              },
              // 散点的动画效果
              {
                //  name: 'Top 5',
                type: 'effectScatter',
                coordinateSystem: 'geo',
                data: this.convertData(this.mapData[n].sort(function (a, b) {
                  return b.value - a.value
                }).slice(0, 20)),
                symbolSize: function (val) {
                  return val[2] / 10
                },
                showEffectOn: 'render',
                rippleEffect: {
                  brushType: 'stroke'
                },
                hoverAnimation: true,
                label: {
                  normal: {
                    formatter: '{b}',
                    position: 'right',
                    show: true
                  }
                },
                itemStyle: {
                  normal: {
                    color: this.colors[this.colorIndex][n],
                    shadowBlur: 10,
                    shadowColor: this.colors[this.colorIndex][n]
                  }
                },
                zlevel: 1
              },
              // 迁移线的动画效果
              {
                type: 'lines',
                zlevel: 2,
                effect: {
                  show: true,
                  period: 4, // 箭头指向速度，值越小速度越快
                  trailLength: 0.02, // 特效尾迹长度[0,1]值越大，尾迹越长重
                  symbol: 'arrow', // 箭头图标
                  symbolSize: 3 // 图标大小
                },
                lineStyle: {
                  normal: {
                    color: this.colors[this.colorIndex][n],
                    width: 0.1, // 尾迹线条宽度
                    opacity: 0.5, // 尾迹线条透明度
                    curveness: 0.3 // 尾迹线条曲直度
                  }
                },
                data: this.convertToLineData(this.mapData[n], this.geoGpsMap[n])
              },
              // 柱状图
              {
                zlevel: 1.5,
                type: 'bar',
                symbol: 'none',
                itemStyle: {
                  normal: {
                    color: this.colors[this.colorIndex][n]
                  }
                },
                data: this.barData[n]
              }
            ]
          })
        }
        // 绘制图表
        this.$nextTick(() => {
          myChart.setOption(optionXyMap01, true)
          myChart.off('click')
          myChart.on('click', this.echartsMapClick)
          myChart.off('mouseover')
          myChart.on('mouseover', this.echartsMapMouseover)
        })
      })
    },
    echartsMapMouseover (params) { // 地图鼠标悬浮事件
      if (params.componentSubType !== 'map') return
      if (!this.province.includes(params.name)) return
      // console.log(this.province.indexOf(params.name))
      const index = this.province.indexOf(params.name)
      const myChart = echarts.init(document.getElementById('myChart'))
      myChart.dispatchAction({
        type: 'timelineChange',
        // 时间点的 index
        currentIndex: index
      })
    },
    echartsMapClick (params) { // 地图点击事件
      const areaList = this.pointData.map(item => item.name)
      if (!areaList.includes(params.name)) {
      // if (!params.data) {
        return
      }
      const data = this.pointData.find(item => item.name === params.name)
      if (
        this.parentInfo[this.parentInfo.length - 1].code === data.cityCode
      ) {
        return
      }
      this.parentInfo.push({
        cityName: data.name,
        code: data.cityCode
      })
      // console.log(data)
      this.clearMemories()
      this.getGeoJson(data.cityCode)
    },
    convertData (data) { // 绘制散点
      // console.log(data)
      const res = []
      // let sum = 0
      // console.log('drawing...', data)
      if (data.length === 0) {
        return res
      }
      for (let i = 0; i < data.length; i++) {
        const geoCoord = this.geoCoordMap[data[i].name]
        if (geoCoord) {
          res.push({
            name: data[i].name,
            value: geoCoord.concat(data[i].value)
          })
          // sum += data[i].value
        }
      }
      const geoCoord = this.geoCoordMap[data[0].province]
      res.push({
        name: data[0].province,
        // value: geoCoord.concat(sum)
        value: geoCoord.concat(300)
      })
      // console.log(res)
      return res
    },
    convertToLineData (data, gps) { // 绘制迁移线
      const res = []
      let sum = 0
      for (let i = 0; i < data.length; i++) {
        const dataItem = data[i]
        sum += dataItem.value
      }
      for (let i = 0; i < data.length; i++) {
        const dataItem = data[i]
        const fromCoord = this.geoCoordMap[dataItem.name]
        const toCoord = gps // 郑州
        //  const toCoord = geoGps[Math.random()*3];
        if (fromCoord && toCoord) {
          res.push([{
            coord: toCoord,
            value: dataItem.value
          }, {
            coord: fromCoord,
            value: sum
          }])
        }
      }
      return res
    },
    randomNum (minNum, maxNum) { // 随机数生成函数
      switch (arguments.length) {
        case 1:
          return parseInt(Math.random() * minNum + 1, 10)
        case 2:
          return parseInt(Math.random() * (maxNum - minNum + 1) + minNum, 10)
        default:
          return 0
      }
    },
    clearMemories () { // 清除内存
      this.mapData = []
      this.geoGpsMap = []
      this.geoCoordMap = {}
      this.barData = []
      this.barLegend = []
      this.province = []
      this.colorIndex = 0
      // console.log('=============clear memories=============')
    },
    async changeYear (year) { // 切换年份
      this.geoGpsMap = {}
      this.colorIndex = 0
      // province= ['福建' '广东' '上海' '台湾' '吉林']
      this.province = []
      this.mapData = []
      this.barData = []
      this.barLegend = []
      this.year = year
      await this.createProvinceList()
      await this.setData()
      await this.drawMap()
    },
    changePlay (event) { // 切换播放状态
      const myChart = echarts.init(document.getElementById('myChart'))
      myChart.dispatchAction({
        type: 'timelinePlayChange',
        playState: event
      })
    },
    // 选择切换市县
    chooseArea (val, index) {
      if (this.parentInfo.length === index + 1) {
        return
      }
      this.parentInfo.splice(index + 1)
      this.clearMemories()
      this.getGeoJson(this.parentInfo[this.parentInfo.length - 1].code)
    }
  }
}
</script>
<style lang="scss">
.map {
  width: calc(100vw - 16px);
  height: calc(100vh - 16px);
}

.mapChoose {
  position: absolute;
  left: 20px;
  top: 55px;
  color: #eee;

  .title {
    padding: 5px;
    border-top: 1px solid rgba(147, 235, 248, 0.8);
    border-bottom: 1px solid rgba(147, 235, 248, 0.8);
    cursor: pointer;
  }

  .icon {
    font-family: "simsun";
    font-size: 25px;
    margin: 0 11px;
  }
}
</style>
