<template>
  <div>
    <div style="position: absolute;z-index: 100000">
      <div>
        <button @click="rotation(10)">向左自转一次</button>
        <button @click="rotation(-10)">向右自转一次</button>
        <button @click="spin()">原地旋转</button>
        <button @click="rotateAround()">绕点旋转</button>
        <button @click="rotateAround2()">绕点旋转2</button>
        <button @click="around(45,distance)">绕点左转一次</button>
        <button @click="moveEntity">移动位置</button>
        <button @click="moveEntity2">移动位置2</button>
      </div>
    </div>
    <div id="cesiumContainer" style="width: 100%;height: 100%"></div>
  </div>
</template>
<script>
import * as Cesium from "cesium";

export default {
  components: {},
  data() {
    return {distance: 300}
  },
  methods: {
    init() {
      var that = this;
      var distance = this.distance;
      let viewer = this.viewer;
      var canvas = viewer.scene.canvas;
      var center = new Cesium.Cartesian3(
          canvas.clientWidth / 2.0,
          canvas.clientHeight / 2.0
      );
      var cartesian = viewer.camera.pickEllipsoid(center);
      console.log(cartesian)
      const cartographic = Cesium.Cartographic.fromCartesian(cartesian);

      const x = Cesium.Math.toDegrees(cartographic.longitude);
      const y = Cesium.Math.toDegrees(cartographic.latitude);
      const z = cartographic.height;
      cartesian = Cesium.Cartesian3.fromDegrees(x, y, 600);
      console.log(cartesian)
      var entity = new Cesium.Entity();
      entity.point = new Cesium.PointGraphics({
        pixelSize: 5,
        color: Cesium.Color.RED,
        outlineColor: Cesium.Color.WHITE,
        outlineWidth: 2,
        // disableDepthTestDistance: Number.POSITIVE_INFINITY
      })
      // entity.position = Cesium.Cartesian3.fromDegrees(113.75, 34.5);
      entity.position = cartesian;
      viewer.entities.add(entity);
      this.currEntity = entity;

      var defaultOptions = {head: 0, pitch: -30, distance: distance};
      var options = {
        offset: new Cesium.HeadingPitchRange(Cesium.Math.toRadians(defaultOptions.head), Cesium.Math.toRadians(Math.ceil(defaultOptions.pitch)), defaultOptions.distance)
      };
      viewer.flyTo(entity, options).then(() => {
        // let pointPrimitive = new Cesium.PointPrimitive();

      });
    },
    spin(){

    },
    flyAround(entity, distance, callback) {
      var that = this;
      //给定飞行一周所需时间，比如10s, 那么每秒转动度数
      var angle = 360 / 20;
      // angle = 2;
      var startTime = Cesium.JulianDate.now();
      var stopTime = Cesium.JulianDate.addSeconds(startTime, 20, new Cesium.JulianDate());
      let viewer = this.viewer;
      viewer.clock.startTime = startTime.clone(); // 开始时间
      viewer.clock.stopTime = stopTime.clone(); // 结速时间
      viewer.clock.currentTime = startTime.clone(); // 当前时间
      viewer.clock.clockRange = Cesium.ClockRange.CLAMPED; // 行为方式
      viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK; //时钟设置为当前系统时间; 忽略所有其他设置。
      // 相机的当前heading
      var initialHeading = viewer.camera.heading;
      var i = 0;
      that.around(10, 500)
      var execution = function TimeExecution() {
        // 当前已经过去的时间，单位s
        var delTime = Cesium.JulianDate.secondsDifference(
            viewer.clock.currentTime,
            viewer.clock.startTime
        );
        that.around(10, 500)
        // console.log("delTime", delTime);
        // var heading = Cesium.Math.toRadians(delTime * angle) + initialHeading;
        // var offset = new Cesium.HeadingPitchRange(heading, viewer.camera.pitch, distance);
        // console.log(offset, Cesium.Math.toDegrees(heading))
        // viewer.zoomTo(entity, offset).then((data) => {
        //   console.log(data)
        // }).catch((data) => {
        //   console.log(data)
        // })
        if (
            Cesium.JulianDate.compare(
                viewer.clock.currentTime,
                viewer.clock.stopTime
            ) >= 0
        ) {
          viewer.clock.onTick.removeEventListener(execution);
          callback && callback();
        }
      };
      viewer.clock.onTick.addEventListener(execution);
    },
    rotation(angle) {
      let viewer = this.viewer;
      viewer.scene.camera.setView({
        orientation: {
          heading: Cesium.Math.toRadians(Cesium.Math.toDegrees(viewer.camera.heading) + angle),
          pitch: viewer.camera.pitch
        }
      });
    },
    around(angle, distance) {
      let viewer = this.viewer;
      let newVar = Cesium.Math.toDegrees(viewer.camera.heading) + angle;
      console.log(newVar)
      let offset = new Cesium.HeadingPitchRange(Cesium.Math.toRadians(newVar), viewer.camera.pitch, distance);
      viewer.zoomTo(this.currEntity, offset).then(data => {
        console.log(data)
      });
    },
    rotateAround() {
      var that = this;
      let viewer = this.viewer;
      let degrees = 0;
      let angle = 0.5;
      let interval = setInterval(() => {
        if (degrees > 360) {
          clearInterval(interval);
          return;
        }

        let viewer = this.viewer;
        let newVar = Cesium.Math.toDegrees(viewer.camera.heading) + angle;
        let offset = new Cesium.HeadingPitchRange(Cesium.Math.toRadians(newVar), viewer.camera.pitch, that.distance);
        viewer.zoomTo(this.currEntity, offset).then((success)=>{
          if(success){
            degrees += angle;
          }
        });
      });
    },
    rotateAround2() {
      const that = this;
      let viewer = this.viewer;
      const startTime = Cesium.JulianDate.now();
      const stopTime = Cesium.JulianDate.addSeconds(startTime, 20, new Cesium.JulianDate());
      viewer.clock.startTime = startTime.clone(); // 开始时间
      viewer.clock.stopTime = stopTime.clone(); // 结速时间
      viewer.clock.currentTime = startTime.clone(); // 当前时间
      viewer.clock.clockRange = Cesium.ClockRange.CLAMPED; // 行为方式
      viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK; //时钟设置为当前系统时间; 忽略所有其他设置。
      let currEntity = that.currEntity;
      let i = 0;
      const listener = () => {
        i += 0.5;
        let offset = new Cesium.HeadingPitchRange(Cesium.Math.toRadians(i), viewer.camera.pitch, 500);
        viewer.zoomTo(currEntity, offset).then(data => {
          console.log(data)
        });
        if (Cesium.JulianDate.compare(viewer.clock.currentTime, viewer.clock.stopTime) >= 0) {
          viewer.clock.onTick.removeEventListener(listener);
        }
      };
      viewer.clock.onTick.addEventListener(listener);

      // let interval = setInterval(() => {
      //   let number = Cesium.Math.toDegrees(viewer.camera.heading) + 0.5;
      //   let offset = new Cesium.HeadingPitchRange(Cesium.Math.toRadians(number), viewer.camera.pitch, 500);
      //   viewer.zoomTo(that.currEntity, offset).then(data => {
      //     console.log(data)
      //   });
      // });
    },
    getCenterEntity() {
      let viewer = this.viewer;
      var canvas = viewer.scene.canvas;
      var center = new Cesium.Cartesian3(
          canvas.clientWidth / 2.0,
          canvas.clientHeight / 2.0,
          this.distance
      );
      var cartesian = viewer.camera.pickEllipsoid(center);
      var entity = new Cesium.Entity({position: cartesian});
      return entity;
    },
    moveEntity() {
      setInterval(() => {
        let cartesian = this.currEntity.position._value;
        const cartographic = Cesium.Cartographic.fromCartesian(cartesian);
        const x = Cesium.Math.toDegrees(cartographic.longitude);
        const y = Cesium.Math.toDegrees(cartographic.latitude);
        const z = cartographic.height;
        this.currEntity.position = Cesium.Cartesian3.fromDegrees(x + 0.00001, y, z);
      });
      this.viewer.trackedEntity = this.currEntity;
    },
    moveEntity2() {
      const that = this;
      let viewer = this.viewer;
      let currEntity = that.currEntity;
      const startTime = Cesium.JulianDate.now();
      const stopTime = Cesium.JulianDate.addSeconds(startTime, 20, new Cesium.JulianDate());
      viewer.clock.startTime = startTime.clone(); // 开始时间
      // viewer.clock.stopTime = stopTime.clone(); // 结速时间
      viewer.clock.currentTime = startTime.clone(); // 当前时间
      viewer.clock.clockRange = Cesium.ClockRange.CLAMPED; // 行为方式
      viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK; //时钟设置为当前系统时间; 忽略所有其他设置。
      let originPosition = currEntity.position._value;
      const listener = () => {
        const cartographic = Cesium.Cartographic.fromCartesian(originPosition);
        const x = Cesium.Math.toDegrees(cartographic.longitude);
        const y = Cesium.Math.toDegrees(cartographic.latitude);
        const z = cartographic.height;
        console.log(x, y, z)
        originPosition = Cesium.Cartesian3.fromDegrees(x + 0.00001, y, z);
        return originPosition;
      };
      currEntity.position = new Cesium.CallbackProperty(listener, false)
      viewer.clock.onTick.addEventListener(listener);
      this.viewer.trackedEntity = this.currEntity;
    }
  },
  mounted() {
    Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJlOGM1MWFiZi0wZjgwLTRiNTItYTMxYi00NzhlNjQzY2JjZjAiLCJpZCI6OTMxODcsImlhdCI6MTY2NjkzOTk2M30.9d8JycHUNLnA1Q_U0GdpNUpx5cWTLP2fU7cdOKOz1GA';
    this.viewer = new Cesium.Viewer("cesiumContainer", {
      baseLayerPicker: false,
      timeline: true,// 必须为true显示时间线组件（如不想显示可以使用样式层叠表修改display：none） 否则viewer.timeline.zoomTo会报undefined错误
      homeButton: false,
      fullscreenButton: false,
      infoBox: false,
      sceneModePicker: false,
      navigationInstructionsInitiallyVisible: false,
      navigationHelpButton: false,
      animation: false,
      shouldAnimate: true, // 必须为true开启动画 否则不会达到飞机模型飞行动画效果
      terrainProvider: Cesium.createWorldTerrain()
    });
    this.viewer.scene.globe.depthTestAgainstTerrain = true;
    window.viewer = this.viewer;
    this.init();
  }
}
</script>
<style scoped>
</style>
