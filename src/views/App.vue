<template>
  <div id="cxtjmap"></div>
</template>

<script setup lang="ts">
import circleImg from '@/images/circle.png';
import minusImg from '@/images/minus.png';
import plusImg from '@/images/plus.png';
import thunderImg from '@/images/thunder.png';
import thunderstormImg from '@/images/thunderstorm.png';
import triangleImg from '@/images/triangle.png';
import mapboxgl from 'mapbox-gl';
import { onBeforeUnmount, onMounted } from 'vue';

let map: mapboxgl.Map;
function initMapBox() {
  mapboxgl.accessToken ='';
  const params:any = {
    container: 'cxtjmap',
    style: {
      sprite: 'mapbox://sprites/examples/cjikt35x83t1z2rnxpdmjs7y7',
      glyphs: 'mapbox://fonts/mapbox/{fontstack}/{range}.pbf',
      version: 8,
      sources: {
        tdtVec: {
          type: 'raster',
          tiles: [
            'http://t0.tianditu.gov.cn/vec_w/wmts?service=WMTS&request=GetTile&version=1.0.0&tilematrixset=w&layer=vec&style=default&tilematrix={z}&tilerow={y}&tilecol={x}&format=tiles&tk=9f1439233cfad9ed75a0f55417a79e42',
            'http://t1.tianditu.gov.cn/vec_w/wmts?service=WMTS&request=GetTile&version=1.0.0&tilematrixset=w&layer=vec&style=default&tilematrix={z}&tilerow={y}&tilecol={x}&format=tiles&tk=9f1439233cfad9ed75a0f55417a79e42',
            'http://t2.tianditu.gov.cn/vec_w/wmts?service=WMTS&request=GetTile&version=1.0.0&tilematrixset=w&layer=vec&style=default&tilematrix={z}&tilerow={y}&tilecol={x}&format=tiles&tk=9f1439233cfad9ed75a0f55417a79e42',
            'http://t3.tianditu.gov.cn/vec_w/wmts?service=WMTS&request=GetTile&version=1.0.0&tilematrixset=w&layer=vec&style=default&tilematrix={z}&tilerow={y}&tilecol={x}&format=tiles&tk=9f1439233cfad9ed75a0f55417a79e42',
          ],
          tileSize: 256,
        },
        tdtCva: {
          type: 'raster',
          tiles: [
            'http://t0.tianditu.gov.cn/cva_w/wmts?service=WMTS&request=GetTile&version=1.0.0&tilematrixset=w&layer=cva&style=default&tilematrix={z}&tilerow={y}&tilecol={x}&format=tiles&tk=9f1439233cfad9ed75a0f55417a79e42',
            'http://t1.tianditu.gov.cn/cva_w/wmts?service=WMTS&request=GetTile&version=1.0.0&tilematrixset=w&layer=cva&style=default&tilematrix={z}&tilerow={y}&tilecol={x}&format=tiles&tk=9f1439233cfad9ed75a0f55417a79e42',
            'http://t2.tianditu.gov.cn/cva_w/wmts?service=WMTS&request=GetTile&version=1.0.0&tilematrixset=w&layer=cva&style=default&tilematrix={z}&tilerow={y}&tilecol={x}&format=tiles&tk=9f1439233cfad9ed75a0f55417a79e42',
            'http://t3.tianditu.gov.cn/cva_w/wmts?service=WMTS&request=GetTile&version=1.0.0&tilematrixset=w&layer=cva&style=default&tilematrix={z}&tilerow={y}&tilecol={x}&format=tiles&tk=9f1439233cfad9ed75a0f55417a79e42',
          ],
          tileSize: 256,
        },
      },
      layers: [
        {
          id: 'background',
          type: 'background',
          minzoom: 0,
          maxzoom: 24,
          layout: { visibility: 'visible' },
          paint: { 'background-color': 'rgba(11, 15, 79, 1)' },
        },
        {
          id: 'tdtVec',
          type: 'raster',
          source: 'tdtVec',
        },
        {
          id: 'tdtCva',
          type: 'raster',
          source: 'tdtCva',
        },
      ],
    },
    center: { lng: 170, lat: 66 },
    zoom: 2,
    maxBounds: [
      [50, 2],
      [166, 56],
    ],
    pitch: 0,
    bearing: 0,
    logoPosition: undefined,
    antialias: true,
  };

  map = new mapboxgl.Map(params);
}
function initImage() {
  loadImage(minusImg, 'minus', { sdf: true });
  loadImage(triangleImg, 'triangle', { sdf: true });
  loadImage(plusImg, 'plus', { sdf: true });
  loadImage(circleImg, 'circle', { sdf: true });
  loadImage(thunderImg, 'thunder', { sdf: true });
  loadImage(thunderstormImg, 'thunderstorm', { sdf: true });
}
function loadImage(src: string, imageName: string, opt?: any) {
  map.loadImage(
    src,
    (error?: Error | undefined, image?: HTMLImageElement | ImageBitmap | undefined) => {
      if (error) throw error;
      map.addImage(imageName, image!, opt);
    },
  );
}
function initThunder() {
  map.addSource('intracloud-lightning', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [],
    },
  });
  map.addLayer({
    id: 'intracloud-lightning',
    type: 'circle',
    source: 'intracloud-lightning',
    paint: {
      'circle-color': ['get', 'color'],
      'circle-radius': ['interpolate', ['linear'], ['zoom'], 0, 3, 10, 5],
      'circle-stroke-width': 0,
    },
  });
  map.addSource('ground-lightning', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [],
    },
  });
  map.addLayer({
    id: 'ground-lightning',
    type: 'symbol',
    source: 'ground-lightning',
    layout: {
      'icon-image': ['get', 'type'],
      'icon-size': [
        'match',
        ['get', 'size'],
        'medium',
        0.3,
        'large',
        0.5,
        0.3, // 默认大小
      ],
      'icon-rotation-alignment': 'map',
      'icon-allow-overlap': true,
    },
    paint: {
      'icon-color': 'green',
    },
  });
  initLineLayer(map);
  bindEvent();
}
// 初始化
function initLineLayer(_map: mapboxgl.Map) {
  _map.addSource('location-line', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [],
    },
  });

  _map.addLayer({
    id: 'location-line',
    type: 'line',
    source: 'location-line',
    layout: {
      'line-join': 'round',
      'line-cap': 'round',
    },
    paint: {
      'line-color': 'red',
      'line-width': 1,
    },
  });
  _map.addSource('location-line-inner', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [],
    },
  });
  _map.addLayer({
    id: 'location-line-inner',
    type: 'line',
    source: 'location-line-inner',
    layout: {
      'line-join': 'round',
      'line-cap': 'round',
    },
    paint: {
      'line-color': 'red',
      'line-width': 1,
      'line-dasharray': [10, 5],
    },
  });
  _map.addSource('location-line-text', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [],
    },
  });
  //添加文本图层
  _map.addLayer({
    id: 'location-line-text',
    type: 'symbol',
    source: 'location-line-text',
    layout: {
      'text-field': ['get','title'], // 文本来源
      'text-offset': [0, 0.5], // 文本偏移量
      'text-anchor': 'bottom-left', // 文本锚点
      'text-size': 10, // 文本大小
      'text-allow-overlap': true,
      'text-ignore-placement': true,
    },
    paint: {
      'text-color': 'green', // 文本颜色
    },
  });
}
function bindEvent() {
  // 鼠标样式
  let popupPoint: any = null;
  const popup = new mapboxgl.Popup({
    closeOnClick: false,
    closeButton: false,
    offset: 26,
    maxWidth: '340px',
    className: 'thunder-popup',
  });

  map.on('mouseleave', 'ground-lightning', function () {
    map.getCanvas().style.cursor = 'auto';
    popup.remove();
    timeHover = true;
  });

  map.on('mouseleave', 'intracloud-lightning', function () {
    map.getCanvas().style.cursor = 'auto';
    popup.remove();
    timeHover = true;
  });

  map.on('mousemove', 'ground-lightning', async function (e) {
    map.getCanvas().style.cursor = 'pointer';
    if (popupPoint?.id != e.features![0].properties!.id) {
      popupPoint = e.features![0].properties;
    }
    console.log(getPopupStr(popupPoint));
    popup
      .setLngLat([e.lngLat.lng, e.lngLat.lat])
      .setHTML((await getPopupStr(popupPoint)) as string)
      .addTo(map);
  });
  map.on('mousemove', 'intracloud-lightning', async function (e) {
    map.getCanvas().style.cursor = 'pointer';
    if (popupPoint?.id != e.features![0].properties!.id) {
      popupPoint = e.features![0].properties;
    }
    popup
      .setLngLat([e.lngLat.lng, e.lngLat.lat])
      .setHTML((getPopupStr(popupPoint)) as string)
      .addTo(map);
  });

  map.on('click', 'ground-lightning', async function (e) {
    map.getCanvas().style.cursor = 'pointer';
    if (popupPoint?.id != e.features![0].properties!.id) {
      popupPoint = e.features![0].properties;
    }
    
    const resData = getLocationLineData(popupPoint);

    //@ts-ignore
    map.getSource('location-line-inner').setData({
      type: 'FeatureCollection',
      features: resData.inner,
    });
    //@ts-ignore
    map.getSource('location-line').setData({
      type: 'FeatureCollection',
      features: resData.outside,
    });
    //@ts-ignore
    map.getSource('location-line-text').setData({
      type: 'FeatureCollection',
      features: resData.stationName,
    });
  });
}
function convexHull(points: { lat: number; lon: number }[]): { lat: number; lon: number }[] {
  // Find the point with the lowest latitude (and smallest longitude if latitudes are equal)
  let start = points.reduce(
    (prev, curr) =>
      prev.lat < curr.lat
        ? prev
        : prev.lat === curr.lat
          ? prev.lon < curr.lon
            ? prev
            : curr
          : curr,
    points[0],
  );

  // Sort points by polar angle
  points.sort((a, b) => polarAngle(start, a) - polarAngle(start, b));

  // Remove the starting point from the sorted list
  const stack: { lat: number; lon: number }[] = [start, points[0]];

  for (let i = 1; i < points.length; i++) {
    while (
      stack.length >= 2 &&
      orientation(stack[stack.length - 2], stack[stack.length - 1], points[i]) <= 0
    ) {
      stack.pop();
    }
    stack.push(points[i]);
  }

  return stack;
}
function polarAngle(
  pivot: { lat: number; lon: number },
  point: { lat: number; lon: number },
): number {
  return Math.atan2(point.lat - pivot.lat, point.lon - pivot.lon);
}
function orientation(
  p1: { lat: number; lon: number },
  p2: { lat: number; lon: number },
  p3: { lat: number; lon: number },
): number {
  return (p2.lon - p1.lon) * (p3.lat - p1.lat) - (p2.lat - p1.lat) * (p3.lon - p1.lon);
}
function calculateDistance(lat1: number, lon1: number, lat2: number, lon2: number) {
  const R = 6371; // 地球半径，单位：km
  const dLat = ((lat2 - lat1) * Math.PI) / 180;
  const dLon = ((lon2 - lon1) * Math.PI) / 180;
  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos((lat1 * Math.PI) / 180) *
      Math.cos((lat2 * Math.PI) / 180) *
      Math.sin(dLon / 2) *
      Math.sin(dLon / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  const distance = R * c;
  return distance;
}

let timeHover = true;
let strBox: string | null = null;
function getPopupStr(PointObj: any) {
  if (timeHover) {
    timeHover = false;
    if (PointObj) {
      console.log(PointObj);
      strBox = `<div class="thunder">
      <div class="item">地址：${PointObj.address}</div>
      </div>`;
    } else {
      strBox = `<div class="thunder">
       <div>无数据 </div>
       </div>`;
    }
  }

  return strBox;
}
function getLocationLineData(currentPoint: any) {
  
  let stationList = JSON.parse(currentPoint.stationList);

  let stationName: any[] = [];

  let innerLine: any[] = [];
  for (let j = 0; j < stationList.length; j++) {
    // 以下为 地图上画站点名称
    let locationText = {
      type: 'Feature',
      geometry: {
        type: 'Point',
        coordinates: [stationList[j].lon, stationList[j].lat], // 文本的经纬度
      },
      properties: {
        title: stationList[j].sta_name_new,
      },
    };
    stationName.push(locationText);

    // 以下为定位点和站之间划线
    let data2 = {
      type: 'Feature',
      properties: {},
      geometry: {
        type: 'LineString',
        coordinates: new Array(),
      },
    };
    data2.geometry.coordinates.push([currentPoint.longitude, currentPoint.latitude]);
    data2.geometry.coordinates.push([stationList[j].lon, stationList[j].lat]);
    innerLine.push(data2);

    // 以下为定位定位點和站之间画公里数
    const distance = calculateDistance(
      currentPoint.latitude,
      currentPoint.longitude,
      stationList[j].lat,
      stationList[j].lon,
    );
    let lon: number = (currentPoint.longitude + stationList[j].lon) / 2;
    let lat: number = (currentPoint.latitude + stationList[j].lat) / 2;
    let locationText2 = {
      type: 'Feature',
      geometry: {
        type: 'Point',
        coordinates: [lon, lat], // 文本的经纬度
      },
      properties: {
        title: Math.floor(distance),
      },
    };
    stationName.push(locationText2);
  }

  let data1 = {
    type: 'Feature',
    properties: {},
    geometry: {
      type: 'LineString',
      coordinates: new Array(),
    },
  };

  let locationToLocationLine: any[] = [];

  if (stationList.length > 2) {
    const sortedCoordinates = convexHull(stationList);

    for (let j = 0; j < sortedCoordinates.length; j++) {
      data1.geometry.coordinates.push([sortedCoordinates[j].lon, sortedCoordinates[j].lat]);

      let k = j + 1;
      if (j == sortedCoordinates.length - 1) {
        data1.geometry.coordinates.push([sortedCoordinates[0].lon, sortedCoordinates[0].lat]);
        k = 0;
      }
      const distance = calculateDistance(
        stationList[j].lat,
        stationList[j].lon,
        stationList[k].lat,
        stationList[k].lon,
      );
      let lon: number = (stationList[j].lon + stationList[k].lon) / 2;
      let lat: number = (stationList[j].lat + stationList[k].lat) / 2;
      let distanceText = {
        type: 'Feature',
        geometry: {
          type: 'Point',
          coordinates: [lon, lat], // 文本的经纬度
        },
        properties: {
          title: Math.floor(distance),
        },
      };
      stationName.push(distanceText);
    }
    locationToLocationLine.push(data1);
  }

  return { inner: innerLine, outside: locationToLocationLine, stationName: stationName };
}

function drawMapData() {
  let data = formMapData();
  updateLocationThunder(data.groundLightingDataArr, data.cloudLightingDataArr);
}
function updateLocationThunder(groundLightingDataArr: any, cloudLightingDataArr: any) {
  //@ts-ignore
  map.getSource('ground-lightning').setData({
    type: 'FeatureCollection',
    features: groundLightingDataArr,
  });
  //@ts-ignore
  map.getSource('intracloud-lightning').setData({
    type: 'FeatureCollection',
    features: cloudLightingDataArr,
  });
}
function formMapData() {
  const stationListData = [
    {
        "town": "陈巴尔虎旗",
        "sta_name": "站1",
        "city": "呼伦贝尔市",
        "sta_name_new": "陈巴尔虎旗",
        "lon": 119.4489,
        "province": "内蒙古自治区",
        "lat": 49.3392,
    },
    {
        "town": "新巴尔虎左旗",
        "sta_name": "站2",
        "city": "呼伦贝尔市",
        "sta_name_new": "新巴尔虎左旗",
        "lon": 118.2631,
        "province": "内蒙古自治区",
        "lat": 48.1864,
    },
    {
        "town": "新巴尔虎右旗",
        "sta_name": "站3",
        "city": "呼伦贝尔市",
        "sta_name_new": "新巴尔虎右旗",
        "lon": 116.8103,
        "province": "内蒙古自治区",
        "lat": 48.6781,
    }
  ]
  let mapData = [{logotype:1,longitude:111.654,latitude:40.812,address:'内蒙古自治区呼和浩特市赛罕区大学东街126号',stationList:stationListData}];
  const groundLightingDataArr: any = [];
  const cloudLightingDataArr: any = [];
  mapData.forEach((el: any, i: number) => {
    //最后一条算最新数据
    if (i === mapData.length - 1) {
      groundLightingDataArr.push({
        type: 'Feature',
        properties: {
          id: i,
          color: '#fff',
          name: 'new',
          type: 'thunder',
          size: 'large',
          ...el,
        },
        geometry: {
          type: 'Point',
          coordinates: [el.longitude, el.latitude],
        },
      });
    } else {
      if (el.lightningType == 0) {//云闪
        cloudLightingDataArr.push({
          type: 'Feature',
          properties: {
            color: 'green',
            id: i,
            ...el,
          },
          geometry: {
            type: 'Point',
            coordinates: [el.longitude, el.latitude],
          },
        });
      }
      if (el.lightningType == 1) {//地闪
        if (el.peakCurrentIntensity > 0) {//正地闪
          groundLightingDataArr.push({
            type: 'Feature',
            properties: {
              id: i,
              color: 'red',
              name: '加',
              type: 'plus',
              size: 0.3,
              ...el,
            },
            geometry: {
              type: 'Point',
              coordinates: [el.longitude, el.latitude],
            },
          });
        }
        if (el.peakCurrentIntensity <= 0) {//负地闪
          groundLightingDataArr.push({
            type: 'Feature',
            properties: {
              id: i,
              color: 'green',
              name: '减',
              type: 'minus',
              size: 0.3,
              ...el,
            },
            geometry: {
              type: 'Point',
              coordinates: [el.longitude, el.latitude],
            },
          });
        }
       
      }
    }
  });
  return {'groundLightingDataArr':groundLightingDataArr,'cloudLightingDataArr':cloudLightingDataArr};
}
function loadMap() {
  map.on('load', function () {
    initImage();
    initThunder();
    drawMapData();
  });
}
function destroyMapbox() {
  map && map.remove();
}

onMounted(() => {
  initMapBox();
  loadMap();
});
onBeforeUnmount(() => {
  destroyMapbox();
});
</script>

<style>
#cxtjmap {
  height: 100%;
}
.thunder-popup {
  .thunder {
    display: flex;
    flex-wrap: wrap;
    color: black;
    font-size: 14px;
    margin: 12px;
    div {
      width: 50%;
      margin-bottom: 6px;

      &.w-100 {
        width: 100%;
      }
    }
  }
}
</style>
