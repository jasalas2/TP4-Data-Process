
//Coleccion de imagenes de Sentinel-1
var s1 = ee.ImageCollection('COPERNICUS/S1_GRD')
        //.filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV','VH'))
        .filter(ee.Filter.eq('instrumentMode', 'IW'))
        .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING')) // puede ajustar a ASCENDING
        .filterBounds(roi)
        
// Filtro de imagenes por fecha
var beforeinc = s1.filterDate('2023-04-01', '2023-04-28')
print(beforeinc,'imagenes disponibles antes del incendio')
/* puede observar que para este rango de fechas tenemos 2 imagenes disponibles
aunque del mismo día
*/
//Imagenes luego del incendio
var afterinc = s1.filterDate('2023-05-10', '2023-06-01')
print(afterinc,'imagenes disponibles despues del incendio')


// pasemos de un ImageCollection a un Image
var beforeinc = beforeinc.mosaic().clip(roi) //puedes cambiar mosaic por mean or median
var afterinc =  afterinc.mosaic().clip(roi)
print(beforeinc, 'imagen antes del incendio')
print(afterinc, 'imagen despues del incendio')
 

Map.addLayer(beforeinc, {bands: ['VV'], min: -15, max: -5, gamma: 1.2}, 'antes del incendio sin speckle',0);

//filtro para reducir el speckle
var SMOOTHING_RADIUS = 40;
var beforeinc = beforeinc.focal_mean(SMOOTHING_RADIUS, 'circle', 'meters'); //ya tiene filtro de speckle
var afterinc = afterinc.focal_mean(SMOOTHING_RADIUS, 'circle', 'meters');

//Parametros de visualizacion
var visualization = {
  bands: ['VH'],  // podemos ajustar la banda a VV
  min: -20,
  max: -5,
};

//DVisualicemos las imagenes
Map.addLayer( beforeinc,visualization, 'antes del incendio',0);
Map.addLayer(afterinc, visualization, 'despues del incendio',0);
//Evalue los resultados


//Elimine el /* de la siguiente linea
 

//Unamos las bandas del antes y despues en un solo image
var coll = beforeinc.addBands(afterinc)
print(coll, 'coleccion junta')

//Hagamos una composicion espacio temporal sencilla
Map.addLayer(coll,imageVisParam, 'Sentinel-1')


//Borre el anterior */ 


//Elimine la siguiente linea cuando se le indique



//expresion sencilla de deteccion de cambio 
var change = coll.expression ('VH / VH_1', {
    'VH': coll.select ('VH'),  // ajuste las bandas como considere
    'VH_1': coll.select ('VH_1')})
    .toDouble().rename('change');

Map.addLayer(change, {min: 0,max:2},'Raster de cambio', 0);
print(change, 'cambio')

var coll2 = coll.addBands(change)
print(coll2, 'coleccion junta con cambio')



//Apply Threshold
var DIFF_UPPER_THRESHOLD = 0.75; // pruebe este valor
var zonas_quemadas = change.lt(DIFF_UPPER_THRESHOLD);
Map.addLayer(zonas_quemadas.updateMask(zonas_quemadas),{palette:"D5421E"},'zonas quemadas',1);


 //elimine el siguiente */


