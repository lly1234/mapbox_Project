<template>
  <div class="home">
    <div id='map' style="height: 980px;"></div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'HomeView',
  components: {
    HelloWorld
  },
  data() {
    return {
      isPlay: false,
      map: '',
      pointData: [],
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    getbbox() {
      var bounds = map.getBounds();
      var ne = bounds.getNorthEast();
      var sw = bounds.getSouthWest();
      var bbox = [sw.lng, sw.lat, ne.lng, ne.lat];
    },
    init() {
      mapboxgl.accessToken =
        'pk.eyJ1IjoiZmVlbGluZ3NreTEyMyIsImEiOiJjbGs5NW5meGowNzNnM21zM3h3N3NjNzc3In0.Zguwo8Z8F3_9KK7_w32w1Q'
      const map = new mapboxgl.Map({
        container: 'map', //id
        style: 'mapbox://styles/mapbox/streets-v11', // mapbox官方的底图和样式文件
        preserveDrawingBuffer: true,//允许地图导出为图片
        center: [117.14841032254009, 34.1811175841714],
        zoom: 15,
      });

      map.on('click', function (e) {
        // 获取点击位置的经纬度坐标
        const clickedCoordinates = e.lngLat.toArray();
        // 打印经纬度坐标
        console.log(clickedCoordinates);
      });


      map.on('load', function () {

        // 定义上次加载的WFS数据
        let lastWFSData = [];

        // 监听地图移动事件
        // map.on('moveend', async () => {
        //   // 获取地图可见范围的边界框
        //   const bounds = map.getBounds();

        //   // 请求最新的WFS数据
        //   const newWFSData = await requestWFSData(bounds);

        //   // 查找差异数据（新增或更新的特征）
        //   const diffData = findDiffData(newWFSData, lastWFSData);

        //   // 更新地图图层
        //   updateWFSLayer(diffData);

        //   // 保存新的WFS数据
        //   lastWFSData = newWFSData;
        // });

        // // 请求WFS数据（根据可见范围）
        // const requestWFSData = async (bounds) => {
        //   // 构造WFS请求URL，根据可见范围进行过滤
        //   const url = `http://your-wfs-service-url.com/wfs?bbox=${bounds.toArray().join(',')}`;

        //   // 发起请求获取WFS数据
        //   const response = await fetch(url);
        //   const wfsData = await response.json();

        //   return wfsData.features;
        // };

        // // 查找差异数据（新增或更新的特征）
        // const findDiffData = (newData, oldData) => {
        //   const diffFeatures = [];

        //   // 通过比较唯一标识符或其他属性来找到差异的特征数据（可根据实际需求进行具体逻辑的实现）
        //   newData.forEach(newFeature => {
        //     const existingFeature = oldData.find(oldFeature => newFeature.id === oldFeature.id);
        //     if (!existingFeature) {
        //       // 新增的特征
        //       diffFeatures.push(newFeature);
        //     } else if (!isEqual(newFeature.properties, existingFeature.properties)) {
        //       // 更新的特征
        //       diffFeatures.push(newFeature);
        //     }
        //   });

        //   return diffFeatures;
        // };

        // // 更新WFS图层
        // const updateWFSLayer = (diffData) => {
        //   // 移除前一次的差异特征图层
        //   if (map.getLayer('wfs-diff-layer')) {
        //     map.removeLayer('wfs-diff-layer');
        //     map.removeSource('wfs-diff-source');
        //   }

        //   // 创建新的差异特征图层
        //   map.addSource('wfs-diff-source', {
        //     type: 'geojson',
        //     data: {
        //       type: 'FeatureCollection',
        //       features: diffData
        //     }
        //   });

        //   // 添加差异特征图层样式和交互
        //   map.addLayer({
        //     id: 'wfs-diff-layer',
        //     type: 'fill',
        //     source: 'wfs-diff-source',
        //     paint: {
        //       'fill-color': 'rgba(0, 255, 0, 0.5)',
        //       'fill-opacity': 0.5
        //     }
        //     // 可根据差异特征的属性设置其他样式和交互
        //   });
        // };

        // wms
        // map.addSource('city', {
        //   type: 'raster',
        //   tiles: ['http://192.168.4.64:8080/geoserver/liangXi/wms?transparent=true&service=WMS&version=1.1.1&request=GetMap&layers=liangXi:dcfw_test3&bbox={bbox-epsg-3857}&width=256&height=256&srs=EPSG:3857&styles=&format=image/png'],
        //   tileSize: 256
        // });
        // map.addLayer({
        //   id: 'city',
        //   type: 'raster',
        //   source: 'city',
        //   paint: {
        //     'raster-opacity': 1
        //   }
        //   // 'source-layer': 'wfs-source'
        // });


        // geojson
        // map.addSource('geojson-source', {
        //   type: 'geojson',
        //   data: {
        //     "type": "FeatureCollection",
        //     "features": [
        //       {
        //         "type": "Feature",
        //         "properties": {},
        //         "geometry": {
        //           "type": "MultiPolygon",
        //           "coordinates": [
        //             [
        //               [
        //                 [
        //                   117.2620286765798,
        //                   34.199622238088224
        //                 ],
        //                 [
        //                   117.26133713113313,
        //                   34.199622238088224
        //                 ],
        //                 [
        //                   117.26133713113313,
        //                   34.19730256479683
        //                 ],
        //                 [
        //                   117.2620286765798,
        //                   34.19730256479683
        //                 ],
        //                 [
        //                   117.2620286765798,
        //                   34.199622238088224
        //                 ]
        //               ]
        //             ]
        //           ]
        //         }
        //       }
        //     ]
        //   },
        // });

        // map.addLayer({
        //   id: 'geojson-layer',
        //   type: 'fill',
        //   source: 'geojson-source',
        //   paint: {
        //     'fill-color': 'blue',
        //     'fill-opacity': 0.5,
        //   },
        // });


        // 向 WFS 服务发起请求，获取地理数据
        // fetch('http://192.168.4.64:8080/geoserver/liangXi/wfs?service=WFS&version=2.0.0&request=GetFeature&typeName=dcfw_test3&outputFormat=application%2Fjson&srsName=EPSG%3A4326')
        //   .then((response) => {response.json()})
        //   .then(data => {
        //     // 将获取到的数据转换为 GeoJSON 格式
        //     var geojsonData = {
        //       type: 'FeatureCollection',
        //       features: data.features,
        //     };
        //     console.log(geojsonData, "geojsonData")
        //     // 将转换后的数据添加为 Mapbox GL JS 的矢量数据源
        //     map.addSource('wfs-source', {
        //       type: 'geojson',
        //       data: geojsonData
        //     });

        //     // 创建一个基于该数据源的图层，并设置样式
        //     map.addLayer({
        //       id: 'wfs-layer',
        //       type: 'fill-extrusion',
        //       source: 'wfs-source',
        //       paint: {
        //         'fill-extrusion-height':3,
        //         'fill-extrusion-color': 'red',
        //         'fill-extrusion-opacity': 1
        //       },

        //       layout: {},
        //       filter: ['all'],
        //     });
        //   });



        // /wfs?service=WFS&request=GetFeature&typeName=mars%3Ahfjy&version=1.0.0&outputFormat=application%2Fjson&srsName=EPSG%3A4326&bbox=117.229614%2C31.827393%2C117.235107%2C31.832886&maxFeatures=500
        map.addSource('wfs-source', {
          type: 'vector',
          tiles: ['http://192.168.4.64:8080/geoserver/liangXi/wfs?service=WFS&version=2.0.0&request=GetFeature&typeName=dcfw_test2&outputFormat=application%2Fjson&srsName=EPSG%3A4326&bbox={bbox-epsg-3857}&maxFeatures=500'],
          // transformRequest: (url, resourceType) => {
          //   if (resourceType === 'Source' && /^https?:\/\/[^?]*?\.geojson$/i.test(url)) {
          //     return {
          //       url: url,
          //       headers: { 'Mapbox-geojson-rewind': 'true' },
          //       credentials: 'same-origin'
          //     };
          //   }
          // }
          // maxzoom: 20,
          // minzoom: 1,
          // promoteId: 'id', // 根据 WFS 响应中的 'id' 字段设置要素 ID
        });

        map.addLayer({
          id: 'wfs-layer',
          type: 'fill',
          source: 'wfs-source',
          'source-layer': 'liangXi:dcfw_test2', // 替换为你的 WFS 图层名称
          paint: {
            'fill-color': 'blue',
            'fill-opacity': 1,
          },
        });

      })
    }
    // map.on('load', function () {
    //   map.addLayer({
    //     id: 'wms-layer',
    //     type: 'raster',
    //     source: {
    //       type: 'raster',
    //       tiles: ['http://192.168.4.64:8080/geoserver/liangXi/wms?service=WMS&request=GetMap&layers=liangXi:dcfw_test'],
    //       tileSize: 256,
    //     },
    //     paint: {}
    //   });
    // });

  }

}
</script>
