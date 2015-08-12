**HCMGIS công bố dịch vụ bản đồ nền và không ảnh**
===================
<p align="right" style="font-style:italic">Trương Thanh Tùng <br> Trung tâm Ứng dụng GIS TP.HCM<br><br></p>

HCMGIS vừa công bố dịch vụ **bản đồ nền và không ảnh** (độ phân giải 25cm) khu vực Tp. HCM. Bài viết trình bày cách khai thác, sử dụng dịch vụ bản đồ này với các thư viện **Leaflet, Google Maps, ArcGIS Javascript và Openlayers**.



Leaflet
-------------
Leaflet ra đời không lâu nhưng được cộng đồng khá ưa chuộn bởi sự mượt mà, nguồn thư viện phong phú, đẹp và dễ sử dụng.

Để tạo một trang bản đồ sử dụng Leaflet, đầu tiên phải khai báo thư viện Leaflet vào phần head trong trang html

```html
<!-- Load Leaflet from CDN-->
<link rel="stylesheet" href="http://cdn.jsdelivr.net/leaflet/0.7.3/leaflet.css" />
<script src="http://cdn.jsdelivr.net/leaflet/0.7.3/leaflet.js"></script>
```
Sau đó, thêm gắn thêm các bộ thư viện hỗ trợ cần thiết như jquery, jqueryui, esri-leaflet

```html
<!-- Load Esri Leaflet from CDN -->
<script src="http://cdn.jsdelivr.net/leaflet.esri/1.0.0/esri-leaflet.js"></script>
<link href="http://ajax.googleapis.com/ajax/libs/jqueryui/1.10.4/themes/smoothness/jquery-ui.min.css" rel="stylesheet" type="text/css"/>
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.0/jquery.min.js"></script>
<script src="http://ajax.googleapis.com/ajax/libs/jqueryui/1.10.4/jquery-ui.min.js"></script>
```
Tủy chỉnh css để bản đồ đẹp hơn
Tạo thẻ div với id là map và các đối tượng con

```html
<div id="map">
<div id="basemapslidercontainer">
<div id="basemapslider">
</div>
</div>
```

Khai báo ranh giới, mức zoom tối thiểu, tọa độ khởi tạo cho biến map sau id map
```javascript
var map = L.map('map',{maxBounds: bound, minZoom: 5}).setView([10.7664156071, 106.675753848], 10);
```
Khởi tạo biến bản đồ và tùy chỉnh kèm theo với

 - ***L.esri.dynamicMapLayer*** để nhúng các lớp vector, hàm này sẽ cache tự động bản đồ tùy theo mức zoom mà người dùng hiển thị, phù hợp khi hiển thị bản đổ ở tỉ lệ lớn như 1:200, 1:500
 - ***L.esri.imageMapLayer***  để nhúng lớp ảnh viễn thám như Lidar, Landsat, Kompsat,…
 - ***L.tileLayer*** để nhúng lớp ảnh đã được cắt nhỏ theo định đạng chuẩn

```javascript
  var hcm_basemap = L.esri.dynamicMapLayer({
    url: 'http://hcmgisportal.vn/gisc/rest/services/BaseMap/BaseMap_HCM/MapServer',
    opacity: 1,
    useCors: false
  });

  var lidar = L.esri.imageMapLayer({
    url: 'http://hcmgisportal.vn/gisc/rest/services/TRUE_ORTHOR/ImageServer',
    position: 'back'
  });

  var qtiles = L.tileLayer('http://hcmgisportal.vn/gisc/Mapnik16/{z}/{x}/{y}.jpg', {
    tms: false,
    attribution: 'Generated by QTiles'
  });
```

Thêm thanh trượt điều chỉnh độ trong suốt cho lớp **hcm_basemap**

```javascript
$(document).ready(function () {
    $("#basemapslider").slider({
          animate: true,
          value: 1,
          orientation: "vertical",
          min: 0,
          max: 1,
          step: 0.1,
          slide: function (event, ui) {
              hcm_basemap.setOpacity(ui.value);
          }
     });

      $('#basemapslider').mousedown(function(){
        map.dragging.disable();
      })

      $('#basemapslider').mouseup(function(){
        map.dragging.enable();
    });
  });
```

Khởi tạo bảng điều khiển bật tắt các lớp

```javascript
L.control.layers({
    'Bản đồ Gray': layer.addTo(map),
    'Bản đồ Streets': L.esri.basemapLayer("Streets"),
    'Bản đồ Imagery':  L.esri.basemapLayer("Imagery")
}, {
  'Ảnh Lidar': lidar,
  'Bản đồ nền TPHCM': hcm_basemap.addTo(map),
  'Bản đồ nền TPHCM (QTiles)': qtiles,
  'Lớp giao thông HCM': gt_hcm
}).addTo(map);

```

***Kết quả***
![enter image description here](https://lh3.googleusercontent.com/nnp27jiYd-JX5kZWdHATqFGHHF517StSr7UIjL_wcQ=s0 "IMG_4001.JPG")
Lớp bản đồ nền
Chồng lớp bản đồ nền với không ảnh
Lớp không ảnh

Nhấn vào [đây](http://hcmgisportal.vn/gisc/hcmgis_leaflet.html) để xem ví dụ


Openlayers
-------------
Khai báo thư viện Openlayers trong thẻ head

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/ol3/3.6.0/ol.css" type="text/css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/ol3/3.6.0/ol.js"></script>
```

Tạo thẻ div với id là map

```html
<div id="map" class="map"></div>
```

Khởi tạo biến layer chứa các lớp bản đồ và biến map khai báo thuộc tính cho bản đồ
```javascript
var layers = [
  new ol.layer.Tile({
    source: new ol.source.OSM()
  }),
  new ol.layer.Tile({
    source: new ol.source.TileArcGISRest({
      url: 'http://hcmgisportal.vn/gisc/rest/services/BaseMap/BaseMap_HCM/MapServer'
    })
  })
];
var map = new ol.Map({
  layers: layers,
  target: 'map',
  view: new ol.View({
    center: [11875821, 1208371],
    zoom: 10
  })
});
```

***Kết quả***

Nhấn vào [đây](http://hcmgisportal.vn/gisc/hcmgis_openlayers.html) để xem ví dụ



Google Maps
-------------
Đăng kí **Key API** cho ứng dụng của Google Maps hoặc sử dụng key sau và nhúng vào, thẻ head trong tệp html

```html
<script type="text/javascript"             src="https://maps.googleapis.com/maps/api/js?key=AIzaSyBDKq9HHFQReMFM7PJi3ibNq30WzH8unkI">
</script>
```

Thêm các bộ thư viện hỗ trợ

```html
<script type="text/javascript" src="http://google-maps-utility-library-v3.googlecode.com/svn/trunk/arcgislink/src/arcgislink.js">
</script>
```
Khởi tạo hàm init, tạo style loại bỏ đường giao thông, tạo biến map và cấu hình map
Thêm lớp GIS nền vào bản đồ

```javascript
var url = 'http://hcmgisportal.vn/gisc/rest/services/BaseMap/BaseMap_gt_HCM/MapServer';
var cpc = new gmaps.ags.CopyrightControl(map);
var dynamap = new gmaps.ags.MapOverlay(url, { opacity: 1 });
dynamap.setMap(map);
```

Hiển thị lớp giao thông GIS nền trên Google Maps khi ẩn lớp giao thông của Gmap

***Kết quả***

Nhấn vào [đây](http://hcmgisportal.vn/gisc/hcmgis_googlemaps.html) để xem ví dụ



ArcGIS Javascript 
----------

Khai báo thư viện ArcGIS Javascript

```html
<linkrel="stylesheet"href="http://js.arcgis.com/3.14/esri/css/esri.css">
<scriptsrc="http://js.arcgis.com/3.14/"></script>
```

Khởi tạo biến map và biến lớp giao thông

```javascript
var map;
require(["esri/map", "dojo/domReady!", "esri/layers/ArcGISDynamicMapServiceLayer"], function(Map) {
    map = new Map("map");
    var layer = new esri.layers.ArcGISDynamicMapServiceLayer("http://hcmgisportal.vn:6080/arcgis/rest/services/BaseMap/BaseMap_HCM/MapServer");
     map.addLayer(layer);
});
```
***Kết quả***

Nhấn vào [đây](http://hcmgisportal.vn/gisc/hcmgis_googlemaps.html) để xem ví dụ
