# Bản đồ nền

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

  var qtiles = L.tileLayer(' http://hcmgisportal.vn/gisc/Mapnik16/{z}/{x}/{y}.jpg', {
    tms: false,
    attribution: 'Generated by QTiles'
  });
s);
```
