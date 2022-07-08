<template>
  <!--为echarts准备一个具备大小的容器dom-->
  <div id="main">
    <changeDate :years="years" @changeYear="changeYear" @changePlay="changePlay" ref="changeDate"/>
    <div id="myChart" style="width: calc(100vw - 16px);height: calc(100vh - 16px);" />
    <div class="mapChoose">
      <span v-for="(item, index) in parentInfo" :key="item.code">
        <span class="title" @click="chooseArea(item, index)">{{
          item.cityName == "全国" ? "中国" : item.cityName
        }}</span>
        <span class="icon" v-show="index + 1 != parentInfo.length">></span>
      </span>
    </div>
    <div class="mapSelect">
      图表类型
      <el-select
        v-model="selectType"
        placeholder="请选择"
        style="width:120px"
        @change="typeChange"
        size="mini">
        <el-option
          v-for="item in selectTypeList"
          :key="item.value"
          :label="item.name"
          :value="item.value">
        </el-option>
      </el-select>
    </div>
    <div class="mapSelect" style="top:155px">
      产业类型
      <el-select
        v-model="selectIndustry"
        placeholder="请选择"
        style="width:120px"
        @change="typeChange"
        size="mini">
        <el-option
          v-for="item in selectIndustryList"
          :key="item.value"
          :label="item.name"
          :value="item.value">
        </el-option>
      </el-select>
    </div>
    <div class="mapSelect" style="top:205px">
      产业层次
      <el-select
        v-model="selectLevel"
        placeholder="请选择"
        style="width:120px"
        @change="typeChange"
        size="mini">
        <el-option
          v-for="item in selectLevelList"
          :key="item.value"
          :label="item.name"
          :value="item.value">
        </el-option>
      </el-select>
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
          '#ff6666',
          '#f5e8c8',
          '#3cb371',
          '#c6b38e',
          '#a4d8c2',
          '#f3d999',
          '#d3758f',
          '#dcc392',
          '#38b6b6',
          '#d5b158',
          '#ff6347',
          '#a092f1',
          '#0a915d',
          '#eaf889',
          '#ff6666',
          '#f5e8c8',
          '#3cb371',
          '#c6b38e',
          '#a4d8c2',
          '#f3d999',
          '#d3758f',
          '#dcc392',
          '#38b6b6',
          '#d5b158',
          '#ff6347',
          '#a092f1',
          '#0a915d',
          '#eaf889',
          '#ff6666',
          '#f5e8c8',
          '#3cb371',
          '#c6b38e',
          '#a4d8c2',
          '#f3d999',
          '#d3758f',
          '#dcc392',
          '#38b6b6',
          '#d5b158',
          '#ff6347',
          '#a092f1',
          '#0a915d',
          '#eaf889'
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
      pieData: [], // 扇形图的参数
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
      pointData: [], // 点的数据
      // 选择器
      selectType: '流出',
      selectTypeList: [
        { name: '流入图', value: '流入' },
        { name: '流出图', value: '流出' }
      ],
      // 追加选择器
      selectIndustry: '全部',
      selectIndustryList: [
        { name: '全部', value: '全部' },
        { name: '光电产业', value: '光电' },
        { name: '生物医药', value: '生物医药' },
        { name: '大数据', value: '大数据' },
        { name: '高端装备', value: '高端装备' },
        { name: '电子信息', value: '电子信息' },
        { name: '新能源汽车', value: '新能源汽车' },
        { name: '5G/移动通信', value: '5G/移动通信' }
      ],
      selectLevel: '全部',
      selectLevelList: [
        { name: '全部', value: '全部' },
        { name: '上游', value: '上游' },
        { name: '中游', value: '中游' },
        { name: '下游', value: '下游' }
      ],
      // 企业名单
      entList: {
        佛山市国星半导体技术有限公司: 8200,
        浙江亚威朗科技有限公司: 9225,
        江苏佛照合同能源管理发展有限公司: 500,
        佛山皓徕特光电有限公司: 840.74,
        佛山市国星通用照明有限公司: 1200,
        佛山市国星电子制造有限公司: 1000,
        南阳宝里钒业股份有限公司: 6000,
        广东省广晟财务有限公司: 329,
        南海市光盛电子有限公司: 500,
        '北京光荣联盟半导体照明产业投资中心（有限合伙）': 700,
        广东省新立电子信息进出口有限公司: 500,
        'TCL莱德电光科技（惠州）有限公司': 1500,
        惠州市TCL鸿创科技有限公司: 5000,
        西安TCL工业研究院有限公司: 3960.52,
        天津七一二通信广播股份有限公司: 2000,
        'TCL微芯科技（广东）有限公司': 800,
        惠州泰科立集团股份有限公司: 5000,
        武汉TCL集团工业研究院有限公司: 25295.341,
        广东易家通数字家庭技术发展有限公司: 207.75,
        'TCL半导体科技（广东）有限公司': 750,
        TCL通讯设备股份有限公司: 8333,
        '乐金电子（惠州）有限公司': 250,
        TCL华星光电技术有限公司: 200,
        惠州TCL住商电子有限公司: 4000,
        惠州市TCL信息技术有限公司: 6400,
        惠州市赛洛特通讯有限责任公司: 2000,
        惠州TCL人力资源服务有限公司: 500,
        深圳聚龙光电有限公司: 28000,
        'TCL通讯设备（惠州）有限公司': 630,
        'TCL互联网科技（深圳）有限公司': 5500,
        惠州TCL集团进出口物流有限公司: 2000,
        北京豪客云信息科技有限公司: 3500,
        翰林汇信息产业股份有限公司: 2000,
        '联合信源数字音视频技术（北京）有限公司': 1000,
        天津硅石材料科技有限公司: 1000,
        深圳聚采供应链科技有限公司: 24500,
        'TCL比艾奇精密电路（惠州）有限公司': 600,
        'TCL文化传媒（深圳）有限公司': 18754,
        北京智趣家科技有限公司: 3000,
        深圳前海启航国际供应链管理有限公司: 1800,
        深圳市TCL高新技术开发有限公司: 5000,
        深圳市永聚源电子有限公司: 4000,
        深圳TCL科技有限公司: 2000,
        '创智半导体科技（惠州）有限公司': 5000,
        中新融创资本管理有限公司: 100,
        惠州市冠邦置业投资有限公司: 1650,
        新疆TCL股权投资有限公司: 12300,
        宁波TCL股权投资有限公司: 7500,
        惠州TCL房地产有限公司: 8000,
        惠州市仲恺TCL智融科技小额贷款股份有限公司: 10000,
        '北京上云创展投资中心（有限合伙）': 14700,
        湖北消费金融股份有限公司: 200,
        深圳前海启航供应链管理有限公司: 1170,
        广州智捷基金销售有限公司: 8000,
        '昆山普诺开源股权投资中心（有限合伙）': 300,
        惠州弘晟科技发展有限公司: 3000,
        'TCL家用电器（景德镇）有限公司': 6300,
        TCL科技集团财务有限公司: 174,
        '广东融创岭岳智能制造与信息技术产业股权投资基金合伙企业（有限合伙）': 18754,
        TCL环保科技股份有限公司: 12534.6,
        天津中环半导体股份有限公司: 150,
        厦门TCL科技产业投资有限公司: 35000,
        '重庆中新融鑫投资中心（有限合伙）': 13000,
        深圳倜享企业管理科技有限公司: 8800,
        'TCL南洋电器（广州）有限公司': 16280,
        '深圳TCL战略股权投资基金合伙企业（有限合伙）': 12000,
        惠州市福盛投资发展有限公司: 3400,
        '广东粤财新兴产业股权投资基金合伙企业（有限合伙）': 10000,
        惠州TCL房地产开发有限公司: 1000,
        无锡TCL创动投资有限公司: 2800,
        '国开思远（北京）投资基金有限公司': 16000,
        上海银行股份有限公司: 10300,
        'TCL医疗放射技术（北京）有限公司': 15000,
        惠州市广雄投资发展有限公司: 100,
        'TCL科技集团（天津）有限公司': 300,
        深圳虹天玻璃有限公司: 78000,
        芜湖天马汽车电子有限公司: 5000,
        厦门天马微电子有限公司: 50000,
        武汉天马微电子有限公司: 78000,
        成都天马微电子有限公司: 600,
        广东聚华印刷显示技术有限公司: 200,
        上海天马有机发光显示技术有限公司: 100,
        深圳中航显示技术有限公司: 213.9,
        深圳中航光电子有限公司: 720,
        深圳天工液晶设备有限公司: 1000,
        上海中航光电子有限公司: 13600,
        上海天马微电子有限公司: 220,
        深圳天马微电子公司世界电子来料加工厂: 116.7,
        深圳蛇口天惠净化有限公司: 3300,
        湖北长江新型显示产业创新中心有限公司: 600,
        深圳天马液晶研究开发中心: 650.7,
        深圳市凯虹实业股份有限公司: 40,
        深圳天鹏镀膜有限公司: 400,
        深圳新金达贸易有限公司: 404.5,
        '深圳天马微（香港世界）天鹏电子加工厂': 120,
        华进半导体封装先导技术研发中心有限公司: 12000,
        无锡聚芯微测科技有限公司: 9887,
        南通深南电路有限公司: 1000,
        天芯互联科技有限公司: 18754
      },
      barData: [], // 柱状图的数据
      barLegend: [] // 柱状图的名称
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
              value: Math.random() > 0.7 ? this.randomNum(200, 300) : this.randomNum(30, 100)
            })
          }
        }
      }
      // console.log('this.mapData', this.mapData)
      for (let i = 0; i < this.mapData.length; i++) {
        this.pieData.push([])
        // this.barLegend.push([])
        for (var j = 0; j < this.mapData[i].length; j++) {
          this.pieData[i].push(this.mapData[i][j])
          // this.barLegend[i].push(this.mapData[i][j].name)
        }
      }
      // 排序pieData
      for (let i = 0; i < this.pieData.length; i++) {
        this.pieData[i].sort((a, b) => {
          return a.value - b.value
        })
        // 如果pieData[i]的长度大于5 把超出部分合并为“其他“
        if (this.pieData[i].length > 5) {
          this.pieData[i] = this.pieData[i].slice(0, 5)
          this.pieData[i].unshift({
            name: '其他',
            value: this.pieData[i].reduce((a, b) => {
              return a + b.value
            }, 0) - this.pieData[i][4].value
          })
        }
      }
      // console.log(this.pieData)
      // （临时）生成一些企业数据
      const randomProperty = obj => {
        const keys = Object.keys(obj)
        const key = keys[keys.length * Math.random() << 0]
        return { name: key, value: obj[key] }
      }
      for (let i = 0; i < this.mapData.length; i++) {
        this.barData.push([])
        const length = this.randomNum(5, 10)
        for (let j = 0; j < length; j++) {
          this.barData[i].push(randomProperty(this.entList))
        }
      }
      // 排序barData
      for (let i = 0; i < this.barData.length; i++) {
        this.barLegend.push([])
        this.barData[i].sort((a, b) => {
          return a.value - b.value
        })
        for (let j = 0; j < this.barData[i].length; j++) {
          this.barLegend[i].push(this.barData[i][j].name)
        }
      }
      // console.log(this.barLegend)
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
              right: '2%',
              top: '50%',
              bottom: '10%',
              width: '20%'
            },
            tooltip: {
              trigger: 'item',
              extraCssText: 'text-align: left;',
              formatter: params => {
                const showList = ['map']
                if (!showList.includes(params.componentSubType)) {
                  return undefined
                }
                console.log(params)
                const dot = `<span style="display:inline-block;margin-right:5px;border-radius:10px;width:10px;height:10px;background-color:${this.colors[this.colorIndex][params.dataIndex]}"></span>`
                // 逐行显示this.pieData的数据
                const pieData = this.pieData
                let returnStr = `${dot}${params.name}${this.selectIndustry === '全部' ? '' : `${this.selectIndustry}产业`}${this.selectType === '流入' ? '外来' : '对外'}投资企业：`
                for (let i = pieData[params.dataIndex].length - 1; i >= 0; i--) {
                  returnStr += `<br />${pieData[params.dataIndex][i].name}：${pieData[params.dataIndex][i].value}家`
                }
                return returnStr
              }
            },
            geo: {
              show: true,
              map: 'map',
              // roam: true,
              roam: false,
              zoom: 1.1,
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
            title: [
              {
                id: 'statistic',
                text: `${this.year}年${this.province[n]}${this.selectIndustry === '全部' ? '' : `${this.selectIndustry}产业`}${this.selectType === '流入' ? '外来' : '对外'}投资情况`,
                left: 'center',
                top: '3%',
                // x: 'center',
                // y: 'top',
                textAlign: 'center',
                textStyle: {
                  color: '#fff',
                  fontSize: 28
                }
              },
              {
                text: '区域投资企业数量分布图',
                left: '79%',
                top: '3%',
                textStyle: {
                  color: '#fff',
                  fontSize: 20
                }
              },
              {
                text: '重点企业投资金额排行榜',
                left: '79%',
                top: '42%',
                textStyle: {
                  color: '#fff',
                  fontSize: 20
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
              name: '(万元)',
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
                // roam: true,
                roam: false,
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
              // 饼图
              {
                zlevel: 1.5,
                type: 'pie',
                symbol: 'none',
                center: ['85%', '22%'],
                radius: '25%',
                color: this.colors[this.colorIndex],
                data: this.pieData[n],
                label: {
                  normal: {
                    show: true,
                    formatter: '{b}({d}%)' // 模板变量有 {a}、{b}、{c}、{d}，分别表示系列名，数据名，数据值，百分比。{d}数据会根据value值计算百分比
                  }
                }
              },
              // 柱状图
              {
                zlevel: 1.5,
                type: 'bar',
                symbol: 'none',
                barGap: '-100%',
                barCategoryGap: '55%',
                itemStyle: {
                  normal: {
                    color: this.colors[this.colorIndex][n]
                  }
                },
                grid: {
                  left: '10%',
                  right: '10%',
                  bottom: '10%',
                  top: '55%'
                },
                label: {
                  normal: {
                    show: true,
                    position: 'right',
                    textStyle: {
                      color: '#FFF',
                      fontSize: 12
                    }
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
          window.onresize = myChart.resize
          // window.addEventListener('resize', () => {
          //   myChart.resize()
          //   console.log(this.$refs.changeDate)
          //   this.$refs.changeDate.$refs.timeline.resize()
          // })
          window.addEventListener('resize', () => { myChart.resize() })
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
      if (!areaList.includes(params.name) || params.componentSubType !== 'map') {
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
      console.log(sum)
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
          if (this.selectType === '流出') {
            res.push([{
              coord: toCoord,
              value: dataItem.value
            }, {
              coord: fromCoord
            }])
          }
          if (this.selectType === '流入') {
            res.push([{
              coord: fromCoord
            }, {
              coord: toCoord,
              value: dataItem.value
            }])
          }
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
      this.pieData = []
      this.barLegend = []
      this.province = []
      this.barData = []
      this.colorIndex = 0
      // console.log('=============clear memories=============')
    },
    async changeYear (year, event) { // 切换年份
      this.mapData = []
      this.geoGpsMap = {}
      this.colorIndex = 0
      this.province = []
      this.pieData = []
      this.barData = []
      this.barLegend = []
      this.year = year
      await this.createProvinceList()
      await this.setData()
      await this.drawMap()
      this.$nextTick(() => {
        this.changePlay(event)
      })
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
    },
    typeChange () {
      this.clearMemories()
      this.getGeoJson(this.parentInfo[this.parentInfo.length - 1].code)
    }
  }
}
</script>
<style lang="scss">

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
.mapSelect {
  position: absolute;
  left: 20px;
  top: 105px;
  color: #eee;
}
</style>
