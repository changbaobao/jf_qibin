<template>
  <div
    class="grid-content MainContainer kfScreenFull"
    style="padding:0;"
  >
    <div
      class="aside"
      :class="{'collopseW':hideSide}"
    >
      <div class="left-tree">
        <el-scrollbar style="height:75%;">
          <el-tree
            ref="pipeWellTree"
            style="background-color:#fff;color:#333;"
            :props="treeProps"
            node-key="id"
            :load="treeLoadNode"
            :render-content="renderContent"
            :append="add"
            lazy
            @node-click="nodeSelect"
          />
        </el-scrollbar>
        <div class="btn-container">
          <div class="btn-item m_b10">
            <el-button
              type="primary"
              plain
              size="mini"
              @click="transformation()"
            >3D转换</el-button>
            <el-button
              type="danger"
              plain
              size="mini"
              @click="showConduit"
            >移除管道</el-button>
          </div>
          <div class="btn-item m_b10">
            <el-button
              type="success"
              plain
              size="mini"
              @click="showTube"
            >移除管井</el-button>
            <el-button
              type="info"
              plain
              size="mini"
              @click="showRegion"
            >移除区域</el-button>
          </div>
          <div class="btn-item">
            <el-button
              type="warning"
              plain
              size="mini"
              @click="showHeat"
            >移除供暖位置</el-button>
          </div>
        </div>
      </div>
      <span
        class="arrow"
        @click="treeShow"
      >
        <img
          src="../../assets/arrow.png"
          alt=""
        >
      </span>
    </div>
    <div
      id="allmap"
      style="width: 100%;height:100%;overflow:hidden;"
    />
    <div class="heatingDetail">
      <div class="item-detail">
        <h2 class="item-title">供热概况</h2>
        <div class="chart-detail">
          <el-col
            :span="10"
            style="height:100%;"
          >
            <pieChart
              :id="heatObj.id"
              height="100%"
              width="100%"
              :series-data="heatObj.seriesData"
              :legend-title="heatObj.legendTitle"
              :title-text="heatObj.titleText"
              :legend-color="heatObj.textColor"
            />
            </pieChart>
          </el-col>
          <el-col
            :span="14"
            style="padding:0 15px;"
          >
            <p>
              全市形成了"热电联产供热为主,可再生能源供热为辅"的集中供热格局。其中,热电联产集中供热覆盖面积2694.61万平方米,各公司覆盖面积见表。地源热泵供热面积513万平方米,燃气壁挂炉取暖面积132.24万平方米。
            </p>
          </el-col>
        </div>
      </div>
      <div
        class="item-detail"
        style="background:#66d3fc;height:calc(55% -10px);margin-top:10px;"
      >
        <h2
          class="item-title"
          style="background:#86dcfd;"
        >面积概况</h2>
        <div class="chart-detail">
          <el-col
            :span="10"
            style="height:100%;"
          >
            <pieChart
              :id="areaObj.id"
              height="100%"
              width="100%"
              :series-data="areaObj.seriesData"
              :color-list="areaObj.colorList"
              :legend-title="areaObj.legendTitle"
              :title-text="areaObj.titleText"
              :legend-color="areaObj.textColor"
            />
            </pieChart>
          </el-col>
          <el-col
            :span="14"
            style="height:100%;padding:0 10px;display: flex;align-items: center;"
          >
            <el-table
              :data="heatCardData"
              size="mini"
              border
              style="width:100%;background:transparent;color:#fff;max-height:100%;overflow-y:auto;"
              :header-cell-style="{
                'background-color': 'rgba(255,255,255,0.3)',
                'color':'#fff'
              }"
            >
              <el-table-column
                prop="companyName"
                label="单位名称"
              />
              <el-table-column
                prop="area"
                align="center"
                label="覆盖面积(万平方米)"
              />
            </el-table>
          </el-col>
        </div>
      </div>
    </div>
  </div>

</template>

<script>
import Vue from 'vue'
import Qs from 'qs'
import AMap from 'AMap'
import pieChart from '@/components/Charts/PieChart'
var GDMap
export default {
  name: 'QBHeatingSurvey',
  components: { pieChart },
  data() {
    return {
      tableH: '100%',
      heatCardData: [
        {
          companyName: '鹤壁淇滨热力有限公司',
          area: '1694.79',
        },
        {
          companyName: '鹤壁盾安供热有限公司',
          area: '630',
        },
        {
          companyName: '淇县朝歌热力有限公司',
          area: '202.7',
        },
        {
          companyName: '浚州热力有限公司',
          area: '167.12',
        },
        {
          companyName: '合计',
          area: '2694.61',
        },
      ],
      hideSide: true,
      treeProps: {
        // 配置父子树的属性名
        label: 'name',
        isLeaf: 'isLeaf',
        children: '',
      },
      selectNode: {},
      selectNodee: {},
      GaoDeMap: {
        // 百度地图
        AMap: {},
      },
      addDialogVisible: false, // 添加开关
      formTitle: '添加',
      form: {
        structureId: '',
        code: '',
        name: '',
        type: 0,
        remark: '',
        lonLat: '',
      },
      pipe: {
        id: '',
        structureId: '',
        code: '',
        name: '',
        type: 0,
        remark: '',
        lonLat: '',
      },
      node: {},
      resolve: '',
      isFirst: true,
      color: '',
      isDisabled: true,
      saveData: [],
      HeatArr: [],
      markerArr: [],
      marker_poly: -1,
      polylineArr: [],
      polyline_poly: -1,
      polygonArr: [],
      polygon_poly: -1,
      isTubeShow: 1,
      isShowConduit: 1,
      isShowRegion: 1,
      isShowHeat: 1,
      polylineEditorArr: [],
      polygonEditorArr: [],
      isChange: '2D',
      change_sign: true,
      heatObj: {
        id: 'heatChart',
        seriesData: [],
        legendTitle: [],
        textColor: '#fff',
      },
      areaObj: {
        id: 'areaChart',
        seriesData: [],
        legendTitle: [],
        textColor: '#fff',
        colorList: ['#5085f2', '#f2719a', '#ee4e40', '#ffcc00'],
      },
    }
  },
  mounted() {
    this.initMap()
    this.getCompanyStructure()
    this.initChart()
    // this.pipelineDisplay()
  },
  created: function () {
    // this.initMap()
  },
  methods: {
    initChart() {
      this.heatObj.legendTitle = ['热电联产', '地源热泵', '燃气壁挂炉']
      this.heatObj.seriesData = [
        {
          name: '供热概况',
          type: 'pie',
          radius: ['30%', '60%'],
          avoidLabelOverlap: false,
          center: ['50%', '45%'],
          data: [
            { value: 2694.61, name: '热电联产' },
            { value: 513, name: '地源热泵' },
            { value: 132.24, name: '燃气壁挂炉' },
          ],

          itemStyle: {
            emphasis: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)',
            },
          },
          itemStyle: {
            normal: {
              label: {
                show: false,
              },
            },
          },
        },
      ]
      this.areaObj.legendTitle = [
        '鹤壁淇滨热力有限公司',
        '鹤壁盾安供热有限公司',
        '淇县朝歌热力有限公司',
        '浚州热力有限公司',
      ]
      this.areaObj.seriesData = [
        {
          type: 'pie',
          name: '面积概况',
          radius: ['30%', '60%'],
          avoidLabelOverlap: false,
          center: ['50%', '45%'],
          data: [
            { value: 1694.79, name: '鹤壁淇滨热力有限公司' },
            { value: 630, name: '鹤壁盾安供热有限公司' },
            { value: 202.7, name: '淇县朝歌热力有限公司' },
            { value: 167.12, name: '浚州热力有限公司' },
          ],
          itemStyle: {
            emphasis: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)',
            },
          },
          itemStyle: {
            normal: {
              label: {
                show: false,
              },
            },
          },
        },
      ]
    },
    /**
     * 点击收起展开
     */
    treeShow() {
      this.hideSide = !this.hideSide
    },
    renderContent(h, { node, data, store }) {
      return (
        <span class="custom-tree-node">
          <span>{node.data.name}</span>
        </span>
      )
    },
    add(val) {
      this.$refs.pipeWellTree.append(val, this.selectNodee)
    },
    // 3D-2D转换
    transformation: function () {
      if (this.change_sign) {
        this.isChange = '3D'
        this.GaoDeMap.AMap.setZoom(17)
        this.change_sign = false
      } else {
        this.isChange = '2D'
        this.change_sign = true
      }
      this.initMap()
      this.getCompanyStructure()
      this.pipelineDisplay()
      this.node.childNodes = []
      this.treeLoadNode(this.node, this.resolve)
      this.GaoDeMap.AMap.setLang('zh_cn')
    },
    showTube: function () {
      if (this.isTubeShow === 1) {
        for (var i = 0; i < this.markerArr.length; i++) {
          this.markerArr[i].hide()
        }
        this.isTubeShow = 2
      } else {
        for (var p = 0; p < this.markerArr.length; p++) {
          this.markerArr[p].show()
        }
        this.isTubeShow = 1
      }
    },
    showConduit: function () {
      if (this.isShowConduit === 1) {
        for (var i = 0; i < this.polylineArr.length; i++) {
          this.polylineArr[i].hide()
        }
        this.isShowConduit = 2
      } else {
        for (var m = 0; m < this.polylineArr.length; m++) {
          this.polylineArr[m].show()
        }
        this.isShowConduit = 1
      }
    },
    showRegion: function () {
      if (this.isShowRegion === 1) {
        for (var i = 0; i < this.polygonArr.length; i++) {
          this.polygonArr[i].hide()
        }
        this.isShowRegion = 2
      } else {
        for (var m = 0; m < this.polygonArr.length; m++) {
          this.polygonArr[m].show()
        }
        this.isShowRegion = 1
      }
    },
    showHeat: function () {
      if (this.isShowHeat === 1) {
        for (var i = 0; i < this.HeatArr.length; i++) {
          this.HeatArr[i].hide()
        }
        this.isShowHeat = 2
      } else {
        for (var m = 0; m < this.HeatArr.length; m++) {
          this.HeatArr[m].show()
        }
        this.isShowHeat = 1
      }
    },
    // 项目点展示
    getCompanyStructure() {
      this.$http
        .post('/S-Base/getCompanyStructureByRole', Qs.stringify({ type: 2 }))
        .then((res) => {
          if (res.data.success) {
            const val = res.data.data
            for (let index = 0; index < val.length; index++) {
              const element = val[index]
              if (element.lon == null) {
              } else {
                var zoomStyleMapping2 = {
                  6: 0,
                  7: 0,
                  8: 0,
                  9: 0,
                  10: 0,
                  11: 0,
                  12: 0,
                  13: 0,
                  14: 0,
                  15: 1,
                  16: 1,
                  17: 1,
                  18: 1,
                  19: 1,
                  20: 1,
                }
                var marker = new AMap.ElasticMarker({
                  position: [element.lon, element.lat],
                  zooms: [8, 20],
                  styles: [
                    {
                      icon: {
                        img:
                          'https://demo-demo12.oss-cn-beijing.aliyuncs.com/1.png',
                        size: [18, 20], // 可见区域的大小
                        ancher: [9, 10], // 锚点
                        scaleFactor: 2, // 地图放大一级的缩放比例系数
                        maxScale: 2, // 最大放大比例
                        minScale: 1, // 最小放大比例
                      },
                    },
                    {
                      icon: {
                        img:
                          'https://demo-demo12.oss-cn-beijing.aliyuncs.com/1.png',
                        size: [140, 150],
                        ancher: [70, 75],
                        fitZoom: 20,
                        scaleFactor: 2,
                        maxScale: 0.455,
                        minScale: 0.225,
                      },
                    },
                  ],
                  zoomStyleMapping: zoomStyleMapping2,
                })
                var _that = this
                marker.on('click', () => {
                  var MyComponent = Vue.extend({
                    methods: {
                      reShow: function () {
                        if (element.threeDUrl) {
                          window.open(element.threeDUrl)
                        }
                        // _that.transformation()
                      },
                      Technology: function () {
                        _that.$router.push({
                          name: 'configurationMonitor',
                          params: { structureId: element.id },
                        })
                      },
                      dataSelect: function () {
                        _that.$router.push({
                          name: 'saveHistoryManage',
                          params: { structId: element.id },
                        })
                      },
                      energy: function () {
                        _that.$router.push({ name: 'energyTotal' })
                        // _that.$http.post('/S-Base/getStructureParentIdById', Qs.stringify({ parentId: element.id })).then(res => {
                        //   if (res.data.success) {
                        //     const { parentID1, parentID2, parentID3 } = res.data.data
                        //     _that.$router.push({ name: 'heatEnergyDisplay', params: { structId: parentID1, parent: parentID2, grandId: parentID3 }})
                        //   }
                        // })
                      },
                    },
                    template:
                      '<div className="custom-infowindow input-card">' +
                      '<label style="color:grey">' +
                      element.name +
                      '</label><br/>' +
                      // 为 infowindow 添加自定义事件
                      '<input type="button" class="btn" value="3D" v-on:click ="reShow"/>' +
                      '<input type="button" class="btn" value="工艺" v-on:click="Technology"/>' +
                      '<input type="button" class="btn" value="查询" v-on:click="dataSelect"/>' +
                      '<input type="button" class="btn" value="能耗" v-on:click="energy"/>' +
                      '</div>',
                  })
                  var component = new MyComponent().$mount()
                  // 创建一个自定义内容的 infowindow 实例
                  var infoWindow = new AMap.InfoWindow({
                    position: [element.lon, element.lat],
                    content: component.$el,
                  })

                  infoWindow.open(_that.GaoDeMap.AMap)
                })
                this.GaoDeMap.AMap.add(marker)
              }
            }
          }
        })
    },
    // 显示点 折现 区域
    displayBMap(data) {
      var point = []
      var pGon = []
      var xqo = JSON.parse(data.lonLat)
      if (data.type === 1) {
        for (var k in xqo) {
          var rel = [xqo[k].lon, xqo[k].lat]
          point.push(rel)
        }
        this.color = data.pipeColor
        var polyline = new AMap.Polyline({
          path: point,
          strokeColor: this.color,
          strokeOpacity: 1,
          strokeWeight: 3,
          // 折线样式还支持 'dashed'
          strokeStyle: 'solid',
          // strokeStyle是dashed时有效
          strokeDasharray: [10, 5],
          lineJoin: 'round',
          lineCap: 'round',
          zIndex: 50,
        })
        polyline.kfData = data.code
        this.polylineArr.push(polyline)
        polyline.setMap(this.GaoDeMap.AMap)
        var polyEditor = new AMap.PolyEditor(this.GaoDeMap.AMap, polyline)
        polyEditor.kfData = data.code
        this.polylineEditorArr.push(polyEditor)
        var infoWindow = new AMap.InfoWindow({
          anchor: 'top-left',
          content: data.name,
        })
        var _this = this
        polyline.on('mouseover', function (e) {
          infoWindow.open(_this.GaoDeMap.AMap, [e.lnglat.lng, e.lnglat.lat])
        })
        polyline.on('mouseout', function (e) {
          infoWindow.close()
        })
        // 缩放地图到合适的视野级别
        // this.GaoDeMap.AMap.setFitView([ polyline ])
      } else if (data.type === 2) {
        // const icon = new AMap.Icon({
        //   image: require('../../assets/gj.png'),
        //   imageSize: new AMap.Size(36, 36),
        //   anchor: 'center'
        // })
        // var marker = new AMap.Marker({
        //   position: [xqo[0].lon, xqo[0].lat],
        //   icon: icon,
        //   offset: new AMap.Pixel(-20, -30),
        //   cursor: 'move',
        //   // 设置拖拽效果
        //   raiseOnDrag: true
        // })
        // marker.kfData = data.code
        // this.markerArr.push(marker)
        // marker.setMap(this.GaoDeMap.AMap)
        // marker.setTitle(data.name)// 鼠标滑过标签
        var zoomStyleMapping2 = {
          6: 0,
          7: 0,
          8: 0,
          9: 0,
          10: 0,
          11: 0,
          12: 0,
          13: 0,
          14: 0,
          15: 1,
          16: 1,
          17: 1,
          18: 1,
          19: 1,
          20: 1,
        }
        var marker = new AMap.ElasticMarker({
          position: [xqo[0].lon, xqo[0].lat],
          zooms: [8, 20],
          styles: [
            {
              icon: {
                img: 'https://demo-demo12.oss-cn-beijing.aliyuncs.com/gj.png',
                size: [100, 100], // 可见区域的大小
                ancher: [9, 10], // 锚点
                scaleFactor: 2, // 地图放大一级的缩放比例系数
                maxScale: 2, // 最大放大比例
                minScale: 1, // 最小放大比例
              },
            },
            {
              icon: {
                img: 'https://demo-demo12.oss-cn-beijing.aliyuncs.com/gj.png',
                size: [140, 150],
                ancher: [60, 75],
                fitZoom: 20,
                scaleFactor: 2,
                maxScale: 0.455,
                minScale: 0.225,
              },
            },
          ],
          zoomStyleMapping: zoomStyleMapping2,
        })
        marker.kfData = data.code
        this.markerArr.push(marker)
        marker.setMap(this.GaoDeMap.AMap)
        marker.setTitle(data.name) // 鼠标滑过标签
        // this.GaoDeMap.AMap.add(marker)
      } else {
        for (var j in xqo) {
          var rele = []
          rele.push(xqo[j].lon, xqo[j].lat)
          pGon.push(rele)
        }
        var polygon = new AMap.Polygon({
          path: pGon,
          strokeColor: '#FF33FF',
          strokeWeight: 3,
          strokeOpacity: 0.2,
          fillOpacity: 0.4,
          fillColor: '#1791fc',
          zIndex: 50,
        })
        polygon.kfData = data.code
        this.polygonArr.push(polygon)
        this.GaoDeMap.AMap.add(polygon)
        var GpolyEditor = new AMap.PolyEditor(this.GaoDeMap.AMap, polygon)
        GpolyEditor.kfData = data.code
        this.polygonEditorArr.push(GpolyEditor)
        var polygonInfoWindow = new AMap.InfoWindow({
          anchor: 'top-left',
          content: data.name,
        })
        var _that = this
        polygon.on('mouseover', function (e) {
          polygonInfoWindow.open(_that.GaoDeMap.AMap, [
            e.lnglat.lng,
            e.lnglat.lat,
          ])
        })
        polygon.on('mouseout', function (e) {
          polygonInfoWindow.close()
        })
      }
    },
    // 点 折现 区域 编辑
    editableBMap(obj) {
      if (obj.type === 1 || obj.type === 3) {
        var xqo = JSON.parse(obj.lonLat)
        this.GaoDeMap.AMap.setCenter([xqo[0].lon, xqo[0].lat]) // 设置地图中心点
        for (var o = 0; o < this.markerArr.length; o++) {
          this.markerArr[o].setAnimation(null)
        }
        var infoWindow = new AMap.InfoWindow({
          anchor: 'top-left',
          content: obj.name,
        })
        var _this = this
        infoWindow.open(_this.GaoDeMap.AMap, [xqo[0].lon, xqo[0].lat])
      } else if (obj.type === 2) {
        var xqo2 = JSON.parse(obj.lonLat)
        this.GaoDeMap.AMap.setCenter([xqo2[0].lon, xqo2[0].lat]) // 设置地图中心点
        for (var b = 0; b < this.polylineEditorArr.length; b++) {
          this.polylineEditorArr[b].close()
        }
        for (var m = 0; m < this.polygonEditorArr.length; m++) {
          this.polygonEditorArr[m].close()
        }
        for (var l = 0; l < this.markerArr.length; l++) {
          if (obj.code === this.markerArr[l].kfData) {
            if (this.marker_poly === -1) {
              this.markerArr[l].setAnimation('AMAP_ANIMATION_BOUNCE')
              this.markerArr[l].setDraggable(true)
              this.marker_poly = l
            } else {
              this.markerArr[this.marker_poly].setAnimation(null)
              this.markerArr[this.marker_poly].setDraggable(false)
              this.markerArr[l].setAnimation('AMAP_ANIMATION_BOUNCE')
              this.markerArr[l].setDraggable(true)
              this.marker_poly = l
            }
          }
        }
      }
    },
    getHeatingPositionLonLat(node) {
      this.$http
        .post(
          '/S-Base/getHeatingPositionByStructureId',
          Qs.stringify({ structureId: node.data.id })
        )
        .then((res) => {
          if (res.data.success) {
            for (let i = 0; i < res.data.data.length; i++) {
              const lon = res.data.data[i].lon
              const lat = res.data.data[i].lat
              if (lon != null) {
                var marker = new AMap.Marker({
                  position: [lon, lat],
                  icon: new AMap.Icon({
                    size: new AMap.Size(25, 34),
                    image: require('../../assets/poi-marker-default.png'),
                    imageSize: new AMap.Size(25, 34),
                    anchor: 'center',
                  }),
                  offset: new AMap.Pixel(-13, -30),
                })
                marker.setMap(this.GaoDeMap.AMap)
                this.HeatArr.push(marker)
                marker.setTitle(res.data.data[i].position) // 鼠标滑过标签
              }
            }
          }
        })
    },
    initMap: function () {
      // 默认地图中心点
      this.GaoDeMap.AMap = new AMap.Map('allmap', {
        center: [114.293298, 35.744579],
        resizeEnable: true,
        expandZoomRange: true,
        zooms: [3, 20],
        // zoom: 14,
        pitch: 80,
        viewMode: this.isChange,
        rotateEnable: true,
        buildingAnimation: true, // 楼块出现是否带动画
        pitchEnable: true,
      })
      this.GaoDeMap.AMap.setMapStyle('amap://styles/light')
      GDMap = this.GaoDeMap.AMap
      this.GaoDeMap.AMap.addControl(
        new AMap.ControlBar({
          showZoomBar: false,
          showControlButton: true,
          position: {
            right: '10px',
            top: '10px',
          },
        })
      )
      this.pipelineDisplay()
      setTimeout(() => {
        this.getMapData()
      }, 100)
    },
    nodeSelect(obj, node, nodeArr) {
      this.selectNode = obj
      this.selectNodee = node
      // 百度地图中心点
      if (obj.lat > 0) {
        this.GaoDeMap.AMap.setCenter([obj.lon, obj.lat]) // 设置地图中心点
        if (obj.structureTypeName === '项目') {
          this.GaoDeMap.AMap.setZoom(obj.gdZoomlevel)
        } else if (obj.structureTypeName === '子公司') {
          this.GaoDeMap.AMap.setZoom(obj.gdZoomlevel)
        } else {
          this.GaoDeMap.AMap.setZoom(obj.gdZoomlevel)
        }
      } else {
        this.editableBMap(obj)
      }
      if (obj.lonLat !== undefined) {
        this.isDisabled = false
      } else {
        this.isDisabled = true
      }
    },
    // 所有一次管网显示
    pipelineDisplay() {
      this.$http.post('/S-Base/getPipeWellByRoleStrId').then((res) => {
        // this.$http.post('/S-Base/getPipeWellByStructureId', Qs.stringify({structureId: '7e69e14d835145fda3369c69071e8e4d'})).then(res => {
        // console.log(res.data.data)
        if (res.data.success) {
          var data = res.data.data
          for (let index = 0; index < data.length; index++) {
            const element = data[index]
            this.displayBMap(element)
            // for (var k in xqo) {
            //   var rel = [xqo[k].lon, xqo[k].lat]
            //   point.push(rel)
            // }
            // this.color = element.pipeColor
            // var polyline = new AMap.Polyline({
            //   path: point,
            //   strokeColor: this.color,
            //   strokeOpacity: 1,
            //   strokeWeight: 3,
            //   // 折线样式还支持 'dashed'
            //   strokeStyle: 'solid',
            //   // strokeStyle是dashed时有效
            //   strokeDasharray: [10, 5],
            //   lineJoin: 'round',
            //   lineCap: 'round',
            //   zIndex: 50
            // })
            // // this.GaoDeMap.AMap.setZoom(14)
            // polyline.kfData = element.code
            // this.polylineArr.push(polyline)
            // polyline.setMap(this.GaoDeMap.AMap)
            // var polyEditor = new AMap.PolyEditor(this.GaoDeMap.AMap, polyline)
            // polyEditor.kfData = element.code
            // this.polylineEditorArr.push(polyEditor)
            // var infoWindow = new AMap.InfoWindow({
            //   anchor: 'top-left',
            //   content: element.name
            // })
            // var _this = this
            // polyline.on('mouseover', function(e) {
            //   infoWindow.open(_this.GaoDeMap.AMap, [e.lnglat.lng, e.lnglat.lat])
            // })
            // polyline.on('mouseout', function(e) {
            //   infoWindow.close()
            // })
          }
        }
      })
    },
    treeLoadNode(node, resolve) {
      if (this.isFirst) {
        this.node = node
        this.resolve = resolve
        this.isFirst = false
      }
      // 根节点
      if (node.level === 0) {
        this.getTreeData(0, resolve)
      } else {
        // 查询管道  管井    node.state.isLeaf = true
        if (node.data.structureTypeName === '项目') {
          this.$http
            .post(
              '/S-Base/getPipeWellByStructureId',
              Qs.stringify({ structureId: node.data.id })
            )
            .then((res) => {
              this.getHeatingPositionLonLat(node, resolve)
              if (res.data.success) {
                if (res.data.data.length === 0) {
                  resolve([])
                } else {
                  const arr = []
                  for (let i = 0; i < res.data.data.length; i++) {
                    arr.push(res.data.data[i])
                    const flag = false
                    this.$set(res.data.data[i], 'isLeaf', flag)
                  }
                  for (var i in arr) {
                    this.displayBMap(arr[i])
                  }
                  resolve(arr)
                }
              }
            })
        } else {
          // 其它   非根节点  非管道管井
          this.getTreeData(node.data.id, resolve)
        }
      }
    },
    getTreeData(id, resolve) {
      this.$http
        .post('/S-Base/getStructureByParentID', Qs.stringify({ parentId: id }))
        .then(function (res) {
          if (res.data.success) {
            resolve(res.data.data)
          }
        })
        .catch(function (err) {
          console.log(err)
        })
    },
    getMapData() {
      let val = []
      this.$http
        .post('/S-Base/getStructureByParentID', Qs.stringify({ parentId: 0 }))
        .then(function (res) {
          if (res.data.success) {
            val = res.data.data
            if (val.length !== 0) {
              var zoom = val[0].gdZoomlevel
              if (zoom === null) {
                GDMap.setZoom(11)
              } else {
                GDMap.setZoom(zoom)
              }
              if (val[0].lon === null) {
                GDMap.setCenter([116.397045, 39.909203])
              } else {
                GDMap.setCenter([val[0].lon, val[0].lat])
              }
            }
          }
        })
    },
  },
}
</script>

<style lang="scss" scoped>
.heatingDetail {
  .item-detail {
    width: 100%;
    height: 50%;
    background: #37bae4;
    box-sizing: border-box;
  }
  .item-title {
    height: 40px;
    line-height: 40px;
    text-indent: 10px;
    font-size: 15px;
    color: #fff;
    background: #61c9ec;
    border-bottom: 1px solid #f1f7f7;
  }
  .chart-detail {
    width: 100%;
    height: calc(100% - 40px);
    display: flex;
    justify-content: center;
    align-items: center;
    p {
      color: #fff;
      line-height: 24px;
      font-size: 15px;
    }
  }
}

.grid-content {
  height: 100%;
  position: relative;
}
.aside {
  position: absolute;
  top: 0;
  left: 0;
  width: 18%;
  height: 100%;
  z-index: 200;
  transition: all 0.5s linear;
  background: #fff;
}
.aside.collopseW {
  width: 0;
}
.aside .left-tree {
  width: calc(100% - 20px);
  padding: 15px 10px;
  height: calc(100% - 30px);
}

.aside .btn-container {
  width: 100%;
  height: calc(25% - 30px);
  background: #f4f4f4;
  padding: 15px 0;
}
.aside .btn-container .btn-item {
  display: flex;
  align-items: center;
  justify-content: center;
}
.aside .tree-node {
  background: none;
  color: #333;
  font-size: 16px;
}
.aside .arrow {
  position: absolute;
  right: 0;
  top: 50%;
  width: 22px;
  height: 44px;
  transform: translate(-50%, -50%);
  z-index: 500;
  transform: rotate(0deg);
  transition: all 0.5s linear;
  cursor: pointer;
}
.aside.collopseW .left-tree {
  display: none;
}
.aside.collopseW .arrow {
  right: -22px;
  transform: rotate(-180deg);
}
.MainContainer .el-scrollbar__wrap {
  overflow-x: hidden !important;
}

.heatingDetail {
  width: 40%;
  height: 100%;
  background: #fff;
  position: absolute;
  top: 0;
  right: 0;
  padding: 10px;
  box-sizing: border-box;
  /* box-shadow: 2px 2px 5px #000; */
}
</style>
