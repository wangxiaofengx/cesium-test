<template>
  <div>
    <div style="position: absolute;z-index: 100000">
    </div>
    <div id="cesiumContainer" style="width: 100%;height: 100%"></div>
  </div>
</template>
<script>
import * as Cesium from "cesium";

export default {
  components: {},
  data() {
    return {distance: 500}
  },
  methods: {
    /**
     * 计算飞行路径
     * @param source 数据坐标
     * @returns {SampledPositionProperty|*}
     */
    createProperty(start, source) {
      // 取样位置 相当于一个集合
      let property = new Cesium.SampledPositionProperty();
      for (let i = 0; i < source.length; i++) {
        let time = Cesium.JulianDate.addSeconds(start, source[i].time, new Cesium.JulianDate);
        let position = Cesium.Cartesian3.fromDegrees(source[i].longitude, source[i].latitude, source[i].height);
        // 添加位置，和时间对应
        property.addSample(time, position);
      }
      return property;
    }
  },
  mounted() {
    var viewer = new Cesium.Viewer('cesiumContainer', {
      baseLayerPicker: false,
      timeline: true,// 必须为true显示时间线组件（如不想显示可以使用样式层叠表修改display：none） 否则viewer.timeline.zoomTo会报undefined错误
      homeButton: false,
      fullscreenButton: false,
      infoBox: false,
      sceneModePicker: false,
      navigationInstructionsInitiallyVisible: false,
      navigationHelpButton: false,
      animation: false,
      shouldAnimate: true // 必须为true开启动画 否则不会达到飞机模型飞行动画效果
    });
    // viewer.animation.container.style.display = 'none';//隐藏动画控件
    // viewer.timeline.container.style.display = 'none';//隐藏时间线控件
    // viewer.geocoder.container.style.display = 'none';//隐藏地名查找控件
    // viewer.cesiumWidget.creditContainer.style.display = 'none';//隐藏ceisum标识
    viewer.scene.globe.enableLighting = true;
    let data = [{longitude: 116.405419, latitude: 39.918034, height: 0, time: 0},
      {longitude: 120.2821, latitude: 33.918145, height: 0, time: 30},
      {longitude: 115.497402, latitude: 39.344641, height: 70000, time: 60},
      {longitude: 107.942392, latitude: 29.559967, height: 70000, time: 110},
      {longitude: 106.549265, latitude: 29.559967, height: 0, time: 120}];
    // 起始时间
    let start = Cesium.JulianDate.fromDate(new Date());
    // 结束时间
    let stop = Cesium.JulianDate.addSeconds(start, 120, new Cesium.JulianDate());
    // 设置始时钟始时间
    viewer.clock.startTime = start.clone();
    // 设置时钟当前时间
    viewer.clock.currentTime = start.clone();
    // 设置始终停止时间
    viewer.clock.stopTime = stop.clone();
    // 时间速率，数字越大时间过的越快
    viewer.clock.multiplier = 1;
    // 时间轴
    viewer.timeline.zoomTo(start, stop);
    // 循环执行,即为2，到达终止时间，重新从起点时间开始
    viewer.clock.clockRange = Cesium.ClockRange.LOOP_STOP;
    // 摄像机聚焦到开始位置
    viewer.camera.flyTo({
      destination: Cesium.Cartesian3.fromDegrees(116.405419, 39.918034, 100000)
    })
    let property = this.createProperty(start, data);
    // 添加entity实体
    let planeModel = viewer.entities.add({
      // 和时间轴关联
      availability: new Cesium.TimeIntervalCollection([new Cesium.TimeInterval({
        start: start,
        stop: stop
      })]),
      id: 'move',
      position: property,
      // 根据所提供的位置计算模型的朝向
      orientation: new Cesium.VelocityOrientationProperty(property),
      // 模型
      model: {
        uri: './Apps/SampleData/models/CesiumAir/Cesium_Air.glb',
        minimumPixelSize: 128
      },
      path: {
        resolution: 1,
        material: new Cesium.PolylineGlowMaterialProperty({
          glowPower: 0.1,
          color: Cesium.Color.YELLOW
        }),
        // leadTime、trailTime 不设置 path全显示
        // leadTime:0,// 设置为0时 模型通过后显示path
        trailTime: 0,// 设置为0时 模型通过后隐藏path
        width: 10
      }
    });

    viewer.clock.onTick.addEventListener((tick) => {
      // let q = viewer.entities.getById('move').orientation.getValue(tick.currentTime)
      // 根据四元素获取方位角heading ,pitch, roll 设置仰角等
      // if (q == undefined) return
      // let hpr = Cesium.HeadingPitchRoll.fromQuaternion(q)
      // let heading = Cesium.Math.toDegrees(hpr.heading);
      // let pitch = Cesium.Math.toDegrees(hpr.pitch);
      // let roll = Cesium.Math.toDegrees(hpr.roll);
      let position = viewer.entities.getById('move').position.getValue(tick.currentTime)
      //世界坐标转换为经纬度
      let ellipsoid = viewer.scene.globe.ellipsoid
      let cartographic = ellipsoid.cartesianToCartographic(position);
      let lat = Cesium.Math.toDegrees(cartographic.latitude);
      let lng = Cesium.Math.toDegrees(cartographic.longitude);
      let alt = cartographic.height;
      viewer.scene.camera.setView({
        destination: Cesium.Cartesian3.fromDegrees(lng, lat, 100000),
        // orientation: {
        //   heading,
        //   pitch,
        //   roll
        // }
      })
    })
    viewer.clock.onStop.addEventListener((tick) => {
      console.log(tick, 'stop')
      // 动画执行到结束时间时调用
      // 逻辑代码 removeEventListener => onTick
    })
  }
}
</script>
<style scoped>
</style>
