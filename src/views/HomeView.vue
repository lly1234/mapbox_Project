<template>
  <div class="home">
    <div id="mapGlobal.map" style="height: 980px"></div>
  </div>
</template>

<script>
import * as mapGlobal from './mapGlobal.js'

import mapboxgl from 'mapbox-gl'
import 'mapbox-gl/dist/mapbox-gl.css'
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import _debounce from 'lodash/debounce'
export default {
  name: 'HomeView',
  components: {
    HelloWorld,
  },
  data() {
    return {
      isPlay: false,
      pointData: [],
      ne: mapGlobal.map.getBounds().getNorthEast(),
      sw: mapGlobal.map.getBounds().getSouthWest(),
      qingqiu: null
    }
  },
  mounted() {
    this.init()
    this.Listener()
  },
  methods: {
    getbbox() {
      var bounds = mapGlobal.map.getBounds()
      var ne = bounds.getNorthEast()
      var sw = bounds.getSouthWest()
      var bbox = [sw.lng, sw.lat, ne.lng, ne.lat]
    },
    debounce(func = this.loadBuilding, delay = 200) {
      let timerId
      return function (...args) {
        if (timerId) {
          clearTimeout(timerId)
        }
        timerId = setTimeout(() => {
          func.apply(this, args)
        }, delay)
      }
    },

    Listener() {
      const debouncedSaveData = this.debounce()
      mapGlobal.map.on('load', function () {
        // 向 WFS 服务发起请求，获取地理数据
        mapGlobal.map.on('moveend', function () {
          debouncedSaveData()
        })
      })
    },

    loadBuilding() {

      if (mapGlobal.map.getZoom() < 15.5) {
        return
      }
      console.log("当前级别:" + mapGlobal.map.getZoom(), "当前四至范围:" + this.ne + this.sw);


      //获取四个角点坐标
      // 获取地图视图窗口的边界框
      var bounds = mapGlobal.map.getBounds()
      // 提取边界框的四个角点坐标
      var sw = bounds.getSouthWest() // 西南角点
      var ne = bounds.getNorthEast() // 东北角点

      // 创建一个 AbortController 实例
      const abortController = new AbortController();

      // 将 signal 属性传递给 fetch 请求的 options 对象
      const options = {
        signal: abortController.signal
      };

      // // 发起 fetch 请求
      // fetch('https://example.com/api', options)
      //   .then(response => {
      //     // 处理响应
      //   })
      //   .catch(error => {
      //     // 处理错误
      //   });

      // 在需要中止上一个请求的位置调用 abort 方法
      if (this.qingqiu != null) {
        console.log('取消请求');
        abortController.abort();
      }

      this.qingqiu = fetch(
        'http://192.168.4.11:8080/geoserver/liangXi/wfs?service=WFS&version=1.0.0&request=GetFeature&typeName=zhenjiangBuilding2&maxFeatures=20000&outputFormat=application%2Fjson&srsName=EPSG%3A4326&bbox=' +
        sw.lng +
        ',' +
        sw.lat +
        ',' +
        ne.lng +
        ',' +
        ne.lat, options
      )
        .then((response) => response.json())
        .then((data) => {
          this.qingqiu = null;
          // 将获取到的数据转换为 GeoJSON 格式
          var geojsonData = {
            type: 'FeatureCollection',
            features: data.features,
          }

          // 将转换后的数据添加为 Mapbox GL JS 的矢量数据源
          // if (mapGlobal.map.getLayer(`wfs-layer`)) {
          //   mapGlobal.map.removeLayer(`wfs-layer`)
          // }
          if (mapGlobal.map.getSource(`wfs-source`)) {
            //           //比较新旧数据
            //            // 删除旧数组中不在新数组中的元素
            // for (let i =  this.pointData.features.length - 1; i >= 0; i--) {
            //   if (!geojsonData.features.includes(this.pointData.features[i])) {
            //     this.pointData.features.splice(i, 1);
            //   }
            // }

            // // 添加新数组中不在旧数组中的元素
            // for (const item of geojsonData.features) {
            //   if (!this.pointData.features.includes(item)) {
            //     this.pointData.features.push(item);
            //   }
            // }
            mapGlobal.map.getSource(`wfs-source`).setData(geojsonData)

            var features = mapGlobal.map.queryRenderedFeatures({ layers: ['wfs-layer'] });
            console.log(features.length);

            // mapGlobal.map
            //   .getLayer('wfs-layer')
            //   .setPaintProperty('fill-extrusion-height', ['get', 'height'])
            // mapGlobal.map.setPaintProperty(
            //   'wfs-layer',
            //   'fill-extrusion-color',
            //   'white'
            // )
            // mapGlobal.map.setPaintProperty(
            //   'wfs-layer',
            //   'fill-extrusion-opacity',
            //   1
            // )
            // mapGlobal.map.setMinZoom(15)
          } else {
            mapGlobal.map.addSource('wfs-source', {
              type: 'geojson',
              data: geojsonData,
            })
            // 创建一个基于该数据源的图层，并设置样式
            mapGlobal.map.addLayer({
              id: 'wfs-layer',
              type: 'fill-extrusion',
              source: 'wfs-source',
              paint: {
                'fill-extrusion-height': 3,
                'fill-extrusion-height': ['get', 'height'],
                'fill-extrusion-color': 'white',
                'fill-extrusion-opacity': 0.8,
              },
              layout: {},
              filter: ['all'],
              // maxzoom: 18,
              minzoom: 15,
            })
          }

          this.pointData = geojsonData
        })
        .catch(error => {
          if (error.name === 'AbortError') {
            // 当请求被中止时的处理逻辑
          } else {
            // 其他错误的处理逻辑
          }
        });
      }
    ,
    init() {
      mapboxgl.accessToken =
        'pk.eyJ1IjoiZmVlbGluZ3NreTEyMyIsImEiOiJjbGs5NW5meGowNzNnM21zM3h3N3NjNzc3In0.Zguwo8Z8F3_9KK7_w32w1Q'
      mapGlobal.map = new mapboxgl.Map({
        container: 'mapGlobal.map', //id
        style: 'mapbox://styles/mapbox/streets-v11', // mapbox官方的底图和样式文件
        preserveDrawingBuffer: true, //允许地图导出为图片
        center: [119.451424, 32.202181],
        zoom: 15,
      })

      mapGlobal.map.on('click', function (e) {
        // 获取点击位置的经纬度坐标
        const clickedCoordinates = e.lngLat.toArray()
        // 打印经纬度坐标
        console.log(clickedCoordinates)
      })
    },
  },
}
</script>
