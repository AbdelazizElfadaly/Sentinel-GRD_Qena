# Sentinel-GRD_Qena
 var pt = ee.Geometry.Polygon(
 [[[32.13936714172362,25.99622414083554],
 [32.17550186157225,25.99622414083554],
 [32.17550186157225,26.0255361872467],
  [32.13936714172362,26.0255361872467],
 [32.13936714172362,25.99622414083554]]]);
 var imgVV = ee.ImageCollection('COPERNICUS/S1_GRD')
        .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
           .filter(ee.Filter.eq('instrumentMode', 'IW'))
      .filterBounds(geometry)
        .select('VV')
        .map(function(image) {
          var edge = image.lt(-30.0);
          var maskedImage = image.mask().and(edge.not());
          return image.updateMask(maskedImage);
        });

var desc = imgVV.filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'));
var asc = imgVV.filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'));

var spring = ee.Filter.date('2016-5-1', '2016-6-25');
var lateSpring = ee.Filter.date('2016-5-1', '2016-6-25');
var summer = ee.Filter.date('2016-5-1', '2016-6-25');

var descChange = ee.Image.cat(
        desc.filter(spring).mean(),
        desc.filter(lateSpring).mean(),
        desc.filter(summer).mean());

var ascChange = ee.Image.cat(
        asc.filter(spring).mean(),
        asc.filter(lateSpring).mean(),
        asc.filter(summer).mean());
        
Map.setCenter(32.156,26.008, 12);
Map.addLayer(ascChange, {min: -25, max: 5}, 'Multi-T Mean ASC', true);
Map.addLayer(descChange, {min: -25, max: 5}, 'Multi-T Mean DESC', true);

Export.image.toDrive({
 image: descChange,
 description: 'descChange2020-8-1-2020-9-25',
 scale: 10,
 region: geometry,
 maxPixels: 3E10

});
