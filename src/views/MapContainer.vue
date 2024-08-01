<template>
  <div id="container">
    <el-button class="btn_1" @click="drawPolygon()">开始绘制</el-button>
    <el-button class="btn_2" @click="editPolygon()">{{
      editing ? "完成编辑" : "开始编辑"
    }}</el-button>
  </div>
</template>
<script>
import AMapLoader from "@amap/amap-jsapi-loader";
let mouseTool;
let polyEditor;
export default {
  name: "map-view",
  data() {
    return {
      editing: false,
    };
  },
  mounted() {
    this.initAMap();
  },
  unmounted() {
    this.map?.destroy();
  },
  methods: {
    initAMap() {
      window._AMapSecurityConfig = {
        securityJsCode: "c096a10e962c5ae822ef7fc0e518ff1d",
      };
      AMapLoader.load({
        key: "488b13cdbdd3d11f38cad568057a910c", // 申请好的Web端开发者Key，首次调用 load 时必填
        version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        plugins: ["AMap.MouseTool", "AMap.PolygonEditor"], //需要使用的的插件列表，如比例尺'AMap.Scale'，支持添加多个如：['...','...']
      })
        .then((AMap) => {
          this.map = new AMap.Map("container", {
            pitch: 80, //地图俯仰角度，有效范围 0 度- 83 度
            viewMode: "3D", //地图模式
            rotateEnable: true, //是否开启地图旋转交互 鼠标右键 + 鼠标画圈移动 或 键盘Ctrl + 鼠标左键画圈移动
            pitchEnable: true, //是否开启地图倾斜交互 鼠标右键 + 鼠标上下移动或键盘Ctrl + 鼠标左键上下移动
            zoom: 17, //初始化地图层级
            rotation: -15, //初始地图顺时针旋转的角度
            zooms: [2, 20], //地图显示的缩放级别范围
            center: [116.333926, 39.997245], //初始地图中心经纬度
            features: ["building"],
          });
        })
        .catch((e) => {
          console.log(e);
        });
    },
    drawPolygon() {
      mouseTool = new AMap.MouseTool(this.map);
      mouseTool.polygon({
        strokeColor: "#FF33FF",
        strokeWeight: 6,
        strokeOpacity: 0.2,
        fillColor: "#1791fc",
        fillOpacity: 0.4,
        // 线样式还支持 'dashed'
        strokeStyle: "solid",
      });
      let _this = this;

      mouseTool.on("draw", function (event) {
        mouseTool.close(true);
        let path = event.obj.getPath(); // 获取多边形的路径
        polyEditor = new AMap.PolygonEditor(_this.map, event.obj);
        let polyphy = new AMap.Polygon({
          path: path,
          extrusionHeight: 50,
          roofColor: "#fff",
          wallColor: "#DDDEDE",
          //bubble: true,
          //draggable: true,
          zIndex: 50,
          extData: { id: 1 },
        });
        _this.map.remove(event.obj);

        polyphy.on("click", () => {
          polyphy.setOptions({
            //修改多边形属性的方法
            roofColor: "#D4FCEF",
            wallColor: "#D4FCEF",
          });
        });
        _this.map.add(polyphy);
      });
    },
    editPolygon() {
      if (!polyEditor) return;
      this.editing ? polyEditor.close() : polyEditor.open();
      this.editing = !this.editing;
    },
  },
};
</script>
<style scoped>
#container {
  width: 100%;
  height: 100vh;
  position: relative;
}
.btn_1 {
  position: absolute;
  top: 30px;
  right: 30px;
  z-index: 999;
}
.btn_2 {
  position: absolute;
  top: 30px;
  right: 150px;
  z-index: 999;
}
::v-deep .amap-logo {
  display: none !important;
}
::v-deep .amap-copyright {
  display: none !important;
}
</style>
