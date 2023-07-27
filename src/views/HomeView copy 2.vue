<template>
  <div class="home">
    <div id="mapGlobal.map" style="height: 980px"></div>
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
      allFeature: [],
      pointData: [],
    }
  },
  mounted() {
    this.init();
    this.Listener()

    let a = this.compareArrays([{ id: 1 }, { id: 3 }], [{ id: 1 }, { id: 3 }], 'id')
    console.log(a);
  },
  methods: {
    getbbox() {
      var bounds = mapGlobal.map.getBounds()
      var ne = bounds.getNorthEast()
      var sw = bounds.getSouthWest()
      var bbox = [sw.lng, sw.lat, ne.lng, ne.lat]
    },
    debounce(func = this.loadBuilding, delay = 1000) {
      let timerId;
      return function (...args) {
        clearTimeout(timerId);

        timerId = setTimeout(() => {
          func.apply(this, args);
        }, delay);
      };
    },

    Listener() {
      const debouncedSaveData = this.debounce();
      mapGlobal.map.on('load', function () {
        // 向 WFS 服务发起请求，获取地理数据
        mapGlobal.map.on('moveend', function () {
          debouncedSaveData()
        })
      })
    },
    loadBuilding() {
      //获取四个角点坐标
      // 获取地图视图窗口的边界框
      var bounds = mapGlobal.map.getBounds()
      // 提取边界框的四个角点坐标
      var sw = bounds.getSouthWest() // 西南角点
      var se = bounds.getSouthEast() // 东南角点
      var ne = bounds.getNorthEast() // 东北角点
      var nw = bounds.getNorthWest() // 西北角点
      const earthRad = 6378137.0 //地球半径

      var param = (sw.lat * Math.PI) / 180
      let xsw =
        (earthRad / 2) *
        Math.log((1.0 + Math.sin(param)) / (1.0 - Math.sin(param)))
      xsw = xsw.toFixed(2)
      let ysw = ((sw.lng * Math.PI) / 180) * earthRad
      ysw = (ysw / 10000000).toFixed(7) + 'E7'

      param = (ne.lat * Math.PI) / 180
      let xne =
        (earthRad / 2) *
        Math.log((1.0 + Math.sin(param)) / (1.0 - Math.sin(param)))
      xne = xne.toFixed(2)
      let yne = ((ne.lng * Math.PI) / 180) * earthRad
      yne = (yne / 10000000).toFixed(7) + 'E7'

      console.log(
        'http://192.168.4.64:8080/geoserver/liangXi/wfs?service=WFS&version=2.0.0&request=GetFeature&typeName=dcfw_test3&outputFormat=application%2Fjson&srsName=EPSG%3A4326&bbox=' +
        ysw +
        ',' +
        xsw +
        ',' +
        yne +
        ',' +
        xne
      )
      fetch(
        'http://192.168.4.64:8080/geoserver/liangXi/wfs?service=WFS&version=2.0.0&request=GetFeature&typeName=zhenjiangBuilding&outputFormat=application%2Fjson&srsName=EPSG%3A4326&bbox=' +
        ysw +
        ',' +
        xsw +
        ',' +
        yne +
        ',' +
        xne
      )
        .then((response) => response.json())
        .then((data) => {
          if (data.features.length > 25000) {
            data.features = data.features.slice(1, 25000)
          }
          // 数据去重
          if (this.allFeature.length == 0) {
            this.allFeature = data.features;
          } else {
            let a = this.compareArrays(this.allFeature, data.features)
            console.log(a);
          }








          // 将获取到的数据转换为 GeoJSON 格式
          var geojsonData = {
            type: 'FeatureCollection',
            features: data.features,
          }
          console.log(geojsonData, 'geojsonData')
          // 将转换后的数据添加为 Mapbox GL JS 的矢量数据源
          if (mapGlobal.map.getLayer(`wfs-layer`)) {
            mapGlobal.map.removeLayer(`wfs-layer`)
          }
          if (mapGlobal.map.getSource(`wfs-source`)) {
            mapGlobal.map.removeSource(`wfs-source`)
          }
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


          const queriedFeatures = mapGlobal.map.querySourceFeatures('wfs-source', {
            sourceLayer: 'wfs-layer', // 替换为实际的源图层名称
          });
          console.log(queriedFeatures);

        })
    },
    init() {
      mapboxgl.accessToken =
        'pk.eyJ1IjoiZmVlbGluZ3NreTEyMyIsImEiOiJjbGs5NW5meGowNzNnM21zM3h3N3NjNzc3In0.Zguwo8Z8F3_9KK7_w32w1Q'
      mapGlobal.map = new mapboxgl.Map({
        container: 'mapGlobal.map', //id
        style: 'mapbox://styles/mapbox/streets-v11', // mapbox官方的底图和样式文件
        preserveDrawingBuffer: true, //允许地图导出为图片
        center: [119.42003731498113, 32.18755608727416],
        zoom: 15,
      })

      mapGlobal.map.on('click', function (e) {
        // 获取点击位置的经纬度坐标
        const clickedCoordinates = e.lngLat.toArray()
        // 打印经纬度坐标
        console.log(clickedCoordinates)
      })


    },

    compareArrays(arr1, arr2, property) {
      // 将 arr1 转换为对象映射的形式，以 property 作为键
      const map = {};
      arr1.forEach(item => {
        const key = item['id'];
        map[key] = item;
      });
      // 检查 arr2 中的元素是否存在于映射中并且相同
      const result = arr2.filter((item) => {
        // console.log(map[item[id]] != undefined,item.id,map[item[id]].id);
        // console.log(map[item['id']] != undefined && map[item['id']].id == item.id);
        if (map[item['id']] != undefined && map[item['id']].id == item.id) {
          return item
        }
      });
      // console.log(result);
      return result;
    },
    queryDeleteFeatures() {
      // 获取要清除的要素的过滤条件
      const filterCondition = ['==', 'property', 'value']; // 替换为实际的属性和属性值过滤条件

      // 查询源数据中符合条件的要素
      const queriedFeatures = map.querySourceFeatures('yourSourceId', {
        sourceLayer: 'yourSourceLayer', // 替换为实际的源图层名称
        filter: filterCondition
      });

      // 遍历查询结果，找到要清除的要素并移除
      if (queriedFeatures.length > 0) {
        const targetFeatureIds = queriedFeatures.map(feature => feature.id);

        // 移除要素
        targetFeatureIds.forEach(featureId => {
          map.removeFeature(featureId, {
            source: 'yourSourceId' // 替换为实际的源数据 ID
          });
        });
      }
    }


  },
}
</script>
  
  
  