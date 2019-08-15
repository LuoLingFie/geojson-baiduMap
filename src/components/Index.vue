<template>
  <div class="hello">
    <!-- 工具快捷兰 -->
    <div class="hortcutBar">
      <!-- 导入JSON文件 -->
      <div class="importJSON">
        <el-button
          type="success"
          @click="importJSON"
          round
        >导入JSON文件</el-button>
        <input
          v-show="false"
          id="file"
          type="file"
          accept=".json"
        />
      </div>
      <!-- 添加规划名 -->
      <div class="lineName">
        <span>规划名:</span>
        <el-input
          v-model="lineName"
          placeholder="请输入规划名"
        ></el-input>
        <span>创建规划:</span>
        <el-button
          @click="addLine"
          type="success"
          icon="el-icon-plus"
          circle
        ></el-button>
      </div>
      <!-- 开启关闭地图绘制功能 -->
      <div class="drawLine">
        <span>规划列表：</span>
        <el-switch
          v-model="drawLine"
          active-color="#13ce66"
          inactive-color="#999"
        >
        </el-switch>
      </div>
      <!-- 开启关闭地图绘制功能 -->
      <div class="drawLine">
        <span>鼠标喵点：</span>
        <el-switch
          v-model="lineMiao"
          @change='lineMiaoFn'
          active-color="#13ce66"
          inactive-color="#999"
        >
        </el-switch>
      </div>
      <!-- 显示隐藏 规划列表 -->
      <div class="theCurrent">
        <p>当前正在编辑：</p><strong>{{theCurrent.name}}</strong>
        <p>ID: </p><span> {{theCurrent.index!==null?theCurrent.index+1:''}}</span>
      </div>
      <!-- 导出JSON文件 -->
      <el-button
        type="success"
        class="exportJSON"
        @click="exportJSON"
        round
      >导出JSON文件</el-button>
    </div>
    <!-- 规划列表 -->
    <div
      :class="{'showright':!drawLine}"
      class="plalistBox"
    >
      <el-tabs v-model="activeName">
        <el-tab-pane
          label="编辑"
          name="first"
        >
          <ul class="plalistBox-list">
            <li
              v-for="(item,index) in pointArr"
              :key="index"
            >
              <p>{{item.name}}</p>
              <el-tooltip
                class="item"
                effect="dark"
                content="编辑折点线"
                placement="top-start"
              >
                <el-button
                  @click="theEditor(index,1)"
                  :plain="true"
                  type="primary"
                  icon="el-icon-edit-outline"
                ></el-button>
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                content="取消编辑折点线"
                placement="top-start"
              >
                <el-button
                  @click="theEditor(index,0)"
                  :plain="true"
                  type="success"
                  icon="el-icon-document-checked"
                ></el-button>
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                content="查看当前规划坐标"
                placement="top-start"
              >
                <el-button
                  :plain="true"
                  type="info"
                  icon="el-icon-folder-opened"
                  @click="showlineJson"
                ></el-button>
              </el-tooltip>
            </li>
          </ul>
        </el-tab-pane>
        <el-tab-pane
          label="无"
          name="second"
        >无</el-tab-pane>
      </el-tabs>
    </div>
    <el-dialog
      title="当前规划坐标"
      :visible.sync="activeNameShow"
      width="30%"
      center
    >
      <span>{{content}}</span>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button
          type="primary"
          @click="activeNameShow=false"
        >确 定</el-button>
      </span>
    </el-dialog>
    <!-- 地图 -->
    <div id="allmap"></div>
  </div>
</template>

<script>
import BMap from 'BMap'
import FileSaver from 'file-saver'
export default {
  mounted() {
    // 创建地图 并且 设置定位
    this.newBmap()
  },
  data() {
    return {
      map: null, // 地图实例
      pointArr: [], // 手绘规划出来的所有点
      PolylineArr: [], // 地图上的规划实例
      PointCollectionArr: [], // 手绘时候添加的圆点实例
      drawLine: true, // 绘画列表显示隐藏
      lineMiao: false, // 是否开启鼠标绘制开关
      activeNameShow: false, // 查看当前单独坐标
      content: '', // 单独坐标
      lineName: '', // 规划线的名称
      activeName: 'first', // tab栏当前页
      theCurrent: {
        index: null,
        name: ''
      }
    }
  },
  methods: {
    // 创建地图 并且 设置定位
    newBmap() {
      let _this = this
      // 创建地图 并且 设置定位
      var map = new BMap.Map('allmap')
      new BMap.LocalCity().get(result => {
        map.centerAndZoom(result.center, 15)
        map.enableScrollWheelZoom()
        map.setCenter(result.name)
      })
      // 单击获取点击的经纬度\
      map.addEventListener('click', function(e) {
        // 判断有没有创建过至少一个 规划
        if (_this.pointArr.length) {
          // 判断是否开启了绘制功能
          if (_this.lineMiao) {
            let { index } = _this.theCurrent
            _this.pointArr[index].points.push(e.point)
            _this.PolylineArr[index].setPath(_this.pointArr[index].points)

            //  添加点 添加点
            let pointCollection = new BMap.PointCollection(
              _this.pointArr[index].points,
              {
                size: 'BMAP_POINT_SIZE_HUGE',
                color: '#d340c3'
              }
            ) // 初始化PointCollection
            _this.PointCollectionArr.push(pointCollection)
            pointCollection.addEventListener('click', function(e) {})
            map.addOverlay(pointCollection) // 添加Overlay
            //  添加点 添加点
            // }
          }
        }
      })
      // 创建地图 并且 设置定位
      this.map = map
    },
    // 添加规划
    addLine() {
      this.lineMiaoFn(false) // 取消鼠标喵点
      if (this.lineName.trim() !== '') {
        let obj = {
          name: this.lineName,
          points: []
        }
        this.pointArr.push(obj) // 添加到初试数据
        this.newPolyline([obj]) // 创建规划实例
        this.theCurrent.name = this.lineName
        this.theCurrent.index = this.pointArr.length - 1
        this.lineName = ''
        this.$notify({
          message: '添加成功！',
          type: 'success'
        })
      } else {
        this.$notify({
          message: '必须要填写规划名哦！',
          type: 'warning'
        })
      }
    },
    // 创建规划百度实例
    newPolyline(arr) {
      arr.forEach(item => {
        let pts = []
        item.points.forEach((v, i) => {
          pts.push(new BMap.Point(v.lng, v.lat))
        })
        let polyline = new BMap.Polyline(pts, {
          strokeColor: 'blue',
          strokeWeight: 3,
          strokeOpacity: 1
        }) // 创建折线
        this.map.addOverlay(polyline)
        this.PolylineArr.push(polyline)
      })
    },
    // 开启关闭当规划编辑
    theEditor(index, bur) {
      this.lineMiaoFn(false) // 取消鼠标喵点
      if (bur) {
        this.theCurrent.name = this.pointArr[index].name
        this.theCurrent.index = index
        let num = this.PolylineArr[index].ia.length
        if (num) {
          this.PolylineArr.forEach((item, ind) => {
            if (ind === index) {
              item.enableEditing() // 开启编辑
            } else {
              item.disableEditing() // 关闭编辑
            }
          })
        } else {
          this.$alert(
            '当前规划一个点都没有！需要先画大致点位！',
            '规划提示：',
            {
              confirmButtonText: '确定',
              callback: action => {
                this.$notify({
                  type: 'success',
                  message: '以开启当前规划的鼠标喵点！'
                })
                this.lineMiao = true
                this.lineMiaoFn(true)
              }
            }
          )
        }
      } else {
        this.PolylineArr[index].disableEditing()
      }
    },
    // 查看单独坐标数据
    showlineJson() {
      this.lineMiaoFn(false) // 取消鼠标喵点
      this.activeNameShow = true
      this.content = JSON.stringify(this.PolylineArr[this.theCurrent.index].ia)
    },
    // 导出json文件
    exportJSON() {
      this.lineMiaoFn(false) // 取消鼠标喵点
      if (this.pointArr.length) {
        this.PolylineArr.forEach((item, ind) => {
          this.pointArr[ind].points = item.ia
        })
        // 将json转换成字符串
        let data = JSON.stringify(this.pointArr)
        let blob = new Blob([data], { type: '' })
        FileSaver.saveAs(blob, 'BmapYX.json')
        this.$notify({
          message: '导出成功！',
          type: 'success'
        })
      } else {
        this.$notify({
          message: '您一个点都没有画哦！',
          type: 'warning'
        })
      }
    },
    // 导入json
    importJSON() {
      this.lineMiaoFn(false) // 取消鼠标喵点
      let fn = () => {
        let file = document.getElementById('file')
        let _this = this
        file.onchange = e => {
          _this.pointArr = []
          _this.PolylineArr = []
          let reader = new FileReader()
          reader.readAsText(document.getElementById('file').files[0])
          reader.onload = function() {
            _this.map.clearOverlays() // 清除所有的撒点
            // this.result为读取到的json字符串，需转成json对象
            _this.pointArr = JSON.parse(this.result)
            _this.newPolyline(JSON.parse(this.result))
            _this.$notify({
              message: '导入成功！',
              type: 'success'
            })
          }
        }
        // 模拟input点击事件
        var evt = new MouseEvent('click', {
          bubbles: false,
          cancelable: true,
          view: window
        })
        file.dispatchEvent(evt)
      }
      if (this.pointArr.length) {
        this.$confirm('地图上已经有画过了哦，确定重新载入？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          fn()
        })
      } else {
        fn()
      }
    },
    // 描绘地图 开关 触发事件
    lineMiaoFn(e) {
      if (e) {
        if (!this.pointArr.length) {
          this.$notify({
            type: 'info',
            message: '请先创建一个规划!'
          })
          this.lineMiao = false
        } else {
          let { name } = this.theCurrent
          if (name.trim() === '') {
            this.$notify({
              type: 'info',
              message: '请在列表先开启一个规划编辑！'
            })
            this.lineMiao = false
          } else {
            this.map.setDefaultCursor('crosshair') // 设置地图默认的鼠标指针样式    十字架型
            this.map.setDraggingCursor('crosshair') // 设置地图拖拽时的鼠标指针样式
          }
        }
      } else {
        this.map.setDefaultCursor('pointer') // 设置地图默认的鼠标指针样式    手型
        this.map.setDraggingCursor('pointer') // 设置地图拖拽时的鼠标指针样式
        this.lineMiao = false
        this.PointCollectionArr.forEach(item => {
          this.map.removeOverlay(item)
        })
        this.PointCollectionArr = []
      }
    }
  },
  watch: {}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
.hello,
#allmap {
  width: 100%;
  height: 100%;
}
.hello {
  // 快捷栏
  .hortcutBar {
    width: 100%;
    height: 50px;
    background-color: rgb(255, 255, 255);
    position: fixed;
    top: 0;
    left: 0;
    z-index: 9;
    box-shadow: 0 8px 5px rgba(0, 0, 0, 0.4);
    display: flex;
    // 导出json文件
    .exportJSON {
      position: absolute;
      right: -15px;
      top: 0;
      height: 100%;
    }
    // 倒入json文件
    .importJSON {
      height: 100%;
      position: relative;
      left: -15px;
      button {
        height: 100%;
      }
    }
    // 地图绘制开关
    .drawLine {
      padding: 13px 10px 0 10px;
      span {
        color: coral;
      }
    }
    // 输入规划名 并且添加
    .lineName {
      padding: 4px 10px 0 10px;
      span {
        color: coral;
      }
      & > div {
        display: inline-block;
        width: 150px;
      }
      & > button {
        // float: right;
      }
    }
    // 当前编辑的样式
    .theCurrent {
      line-height: 50px;
      display: inline-block;
      color: coral;
      display: flex;
      strong {
        width: 100px;
        color: #8634fd;
        font-weight: 700;
        font-size: 20px;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
      }
      span {
        color: #8634fd;
        font-weight: 700;
        font-size: 20px;
      }
    }
  }
  // 规划列表盒子
  .plalistBox {
    box-sizing: border-box;
    padding: 10px;
    position: fixed;
    right: 10px;
    top: 60px;
    z-index: 999;
    height: 600px;
    width: 300px;
    box-shadow: -8px 8px 5px rgba(0, 0, 0, 0.4);
    background-color: #fff;
    transition: right 0.4s;
    &.showright {
      right: -320px;
    }
    .plalistBox-list {
      overflow: auto;
      height: 525px;
      li {
        border-bottom: 1px solid gainsboro;
        box-sizing: border-box;
        padding: 5px 0;
        p {
          color: cadetblue;
          display: inline-block;
          width: 100px;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
        }
        button {
          width: 20px;
          height: 30px;
        }
      }
      &::-webkit-scrollbar {
        width: 5px;
        height: 5px;
      }
      /*定义滚动条轨道 内阴影+圆角*/
      &::-webkit-scrollbar-track {
        background-color: #eee;
      }
      /*定义滑块 内阴影+圆角*/
      &::-webkit-scrollbar-thumb {
        border-radius: 10px;
        background-color: #555;
      }
    }
  }
  // 地图
  #allmap {
    width: 100%;
    height: 100%;
  }
}
</style>
