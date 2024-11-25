# Laboratorio 4 Radar de Apertura Sintetica (SAR)

Lab4SAR

## Sentinel-1
### Laboratorio 4 con sentinel-1

<details>
  <summary>Click</summary>
  Se importa las capas y el ROI de Palo Verde
  
```js
var roi = ee.FeatureCollection('projects/mtb2023-399203/assets/Palo_verde');
Map.addLayer(roi, {color: 'green'}, 'ROI');
Map.centerObject(roi, 12);

```

</details>


<details>
  <summary>Click</summary>
  Se llama la colección de imagenes de S1, escogiendo la polarización y la orita en este caso desendente
  
```js
var s1 = ee.ImageCollection('COPERNICUS/S1_GRD')
        //.filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV','VH'))
        .filter(ee.Filter.eq('instrumentMode', 'IW'))
        .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING')) // puede ajustar a ASCENDING
        .filterBounds(roi)
        Map.addLayer( beforeinc,{bands: ['VV'], min: -15, max: -5, gamma: 1.2}, 'antes del incendio sin speckle', 0);

```

</details>

<details>
  <summary>Click</summary>
  Se reduce el speckle o riudo, utilizando un filtro
  
```js
//filtro para reducir el speckle
var SMOOTHING_RADIUS = 50;
var beforeinc = beforeinc.focal_mean(SMOOTHING_RADIUS, 'circle', 'meters');
var afterinc = afterinc.focal_mean(SMOOTHING_RADIUS, 'circle', 'meters');

```

</details>

<details>
  <summary>Click</summary>
  Se seleccionan las bandas según el estudio realizado para visualizar la zona en este caso se utiliza VV
  
```js
var visualization = {
  bands: ['VH'],  
  min: -20,
  max: -5,
};

```

</details>

<details>
  <summary>Click</summary>
  Se añaden las capas de visualizacion tanto antes como despues del incendio
  
```js
Map.addLayer( beforeinc,visualization, 'antes del incendio',0);
Map.addLayer(afterinc, visualization, 'despues del incendio',0);

```

</details>

<details>
  <summary>Click</summary>
  Se realiza una combinación para unir el antes y el despues en una sola imagen y su visualización
  
```js
var coll = beforeinc.addBands(afterinc)
print(coll, 'coleccion junta')
Map.addLayer(coll,imageVisParam, 'Sentinel-1')

```

</details>

<details>
  <summary>Click</summary>
  Se realiza una expresión combinando dos bandas para medir el indice de cambio de las imagenes y su visualización
  
```js
var change = coll.expression ('VH / VH_1', {
    'VH': coll.select ('VH'),   
    'VH_1': coll.select ('VH_1')})
    .toDouble().rename('change');

Map.addLayer(change, {min: 0,max:2},'Raster de cambio', 0);
print(change, 'cambio')

var coll2 = coll.addBands(change)
print(coll2, 'coleccion junta con cambio')

```

</details>

<details>
  <summary>Click</summary>
  Se identifican las zonas quemadas mediante un umbral que nosotros proporcionamos y su visualización
  
```js
var DIFF_UPPER_THRESHOLD = 0.75; 
var zonas_quemadas = change.lt(DIFF_UPPER_THRESHOLD);
Map.addLayer(zonas_quemadas.updateMask(zonas_quemadas),{palette:"D5421E"},'zonas quemadas',1);

```

</details>


## Sentinel-2
### Laboratorio 4 con sentinel-2

<details>
  <summary>Click</summary>
  Para s2 utilizamos un emascaramiento en las nubes ya que no es un SAR, entonces lo primero es realizar esto
  
```js
function cloudMask(image){
  var scl = image.select('SCL');
  var mask = scl.eq(3).or(scl.gte(7).and(scl.lte(10)));
  return image.updateMask(mask.eq(0));
}

```

</details>


<details>
  <summary>Click</summary>
  Se llama a la colección de imagenes de s2 añadiendo el filtro de nuestro ROI y fechas del estudio
  
```js
var s2 = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED").filterBounds(roi) 
  .filterDate('2023-01-01', '2023-12-31') 
  .filterBounds(roi) 
  .map(cloudMask) 
   print(s2)

```

</details>


<details>
  <summary>Click</summary>
  Se añaden los filtros para las fechas antes del incendio y despues del incendio
  
```js
var antes = s2.filter(ee.Filter.or(
 ee.Filter.date('2023-04-01', '2023-04-28')))
print(antes, 'antes del incendio s2');
var despues = s2.filter(ee.Filter.or(
 ee.Filter.date('2023-05-10', '2023-06-01')))
print( despues, 'despues del incendio s2');

```

</details>


<details>
  <summary>Click</summary>
  Se añaden las dos colecciones (antes y despues) en una sola imagen
  
```js
var antes2 = antes.mosaic().clip(roi) 
var despues2 =  despues.mosaic().clip(roi)
print(antes, 'imagen antes del incendio')
print(despues, 'imagen despues del incendio')

```

</details>


<details>
  <summary>Click</summary>
  Se añaden las dos colecciones (antes y despues) en una sola imagen y su visualización
  
```js
var antes2 = antes.mosaic().clip(roi) 
var despues2 =  despues.mosaic().clip(roi)
print(antes, 'imagen antes del incendio')
print(despues, 'imagen despues del incendio')
Map.addLayer( antes2,{bands: ['B4', 'B3', 'B2'], min: 354.3920564417735, max: 1282.2158558183125, gamma: 1.2}, 'antes del incendio s2', 0);


```

</details>


<details>
  <summary>Click</summary>
  Se realiza una expresion para los indices de vegetacion en este caso nbr indice normalizado de area quemada y su visualización
  
```js
var preNBR = antes2.normalizedDifference(['B8', 'B12']).rename('nbr');
var postNBR = despues2.normalizedDifference(['B8', 'B12']).rename('nbr');
print(preNBR)

Map.addLayer(preNBR, 
{bands: ['nbr'], min: 0.018902123252200934, max:0.7007203002942072 , gamma: 1.2}, 'nbr antes'); 

Map.addLayer(postNBR, 
{bands: ['nbr'], min: 0.018902123252200934, max:0.7007203002942072 , gamma: 1.2}, 'nbr despues');

var dNBR_unscaled = preNBR.subtract(postNBR);
var dNBR = dNBR_unscaled.multiply(1000);

print("Difference Normalized Burn Ratio: ", dNBR);
var grey = ['white', 'black'];

Map.addLayer(preNBR, {min: -1, max: 1, palette: grey}, 'Prefire Normalized Burn Ratio');
Map.addLayer(postNBR, {min: -1, max: 1, palette: grey}, 'Postfire Normalized Burn Ratio');
Map.addLayer(dNBR, {min: -1000, max: 1000, palette: grey}, 'dNBR greyscale');

```

</details>


<details>
  <summary>Click</summary>
  Por ultimo en s2 para visualizar con colores se añade lo siguiente segun la clasificación deseada para el umbral de zonas quemadas y añadimos los codigos de colores para diferenciarla
  
```js
var sld_intervals =
  '<RasterSymbolizer>' +
    '<ColorMap type="intervals" extended="false" >' +
      '<ColorMapEntry color="#ffffff" quantity="-500" label="-500"/>' +
      '<ColorMapEntry color="#7a8737" quantity="-250" label="-250" />' +
      '<ColorMapEntry color="#acbe4d" quantity="-100" label="-100" />' +
      '<ColorMapEntry color="#0ae042" quantity="100" label="100" />' +
      '<ColorMapEntry color="#fff70b" quantity="270" label="270" />' +
      '<ColorMapEntry color="#ffaf38" quantity="440" label="440" />' +
      '<ColorMapEntry color="#ff641b" quantity="660" label="660" />' +
      '<ColorMapEntry color="#a41fd6" quantity="2000" label="2000" />' +
    '</ColorMap>' +
  '</RasterSymbolizer>';

Map.addLayer(dNBR.sldStyle(sld_intervals), {}, 'dNBR classified');

var thresholds = ee.Image([-1000, -251, -101, 99, 269, 439, 659, 2000]);
var classified = dNBR.lt(thresholds).reduce('sum').toInt();


```

</details>



## Desarrollo
## Codigo SAR de detección de incendios S-1 y S-2
https://code.earthengine.google.com/3a2b31824e529ea88a358a70e225aa92

En las siguientes imagenes se puede observar una comparación de los sistemas Sentinel-1 y Sentinel-2 para la detección de incendios en la área de estudio de Palo Verde realizada para el laboratorio 4




En la siguiente imagen se pueden ver una sección antes del incendio, se nota una escala de grises sin textura indicando un patron siendo este el bosque antes que este se quemara 
![s-1 antes](https://github.com/JosephVillarrealVega/LAB4/blob/e7a7e9a562da094e01d9c40425a8dd42693932f9/S-1%20antes%20del%20incendio.PNG)

En la siguiente imagen ser obvserva una mancha negra (zona quemada), esto despues del incendio, esto pasa ya que al no existir la cobertura de arboles que anteriormente estaban las ondas llegan hasta el suelo, un suelo probablemente plano y descubierto el cual hace que se de retrodispersion por superficie lisa
![s-2 despues](https://github.com/JosephVillarrealVega/LAB4/blob/a9669990d53c2eafe05011170403fa31738f18cd/S-1%20despues%20del%20incendio.PNG)

En esta imagen se aprecia como se veria la imagen antes de las correcciones de speckle, se puede ver una especie de "ruido" donde no se llega a entender por completo la imagen
![antes de speckle](https://github.com/JosephVillarrealVega/LAB4/blob/e7a7e9a562da094e01d9c40425a8dd42693932f9/S-1%20antes%20del%20incendio%20sin%20speckle.PNG)

Ahora con Sentinel-2 se aprecia mucho mejor la imgan esto debido a su resolución espacial ya que es mayor a la de los sistemas radar, la visualizamos en una escala de grises para realizar una mejor comparación con las imagenes optenidas por Sentinel-1 
![nbr antes](https://github.com/JosephVillarrealVega/LAB4/blob/e7a7e9a562da094e01d9c40425a8dd42693932f9/NBR%20antes%20del%20incendio.PNG)

En las imagenes despues del incendio igualmente utilizando los indices NBR y en escala de grises, se ve un gris más claro en la zona quemada esto por que el NBR es sensible a suelos descubiertos por incendios
![nbr despues](https://github.com/JosephVillarrealVega/LAB4/blob/e7a7e9a562da094e01d9c40425a8dd42693932f9/NBR%20despues%20del%20incendio.PNG)

Y en esta ultima imagen podemos ver en sentinel-2 a escala de grises el área de estudio
![escala de grises nbr](https://github.com/JosephVillarrealVega/LAB4/blob/e7a7e9a562da094e01d9c40425a8dd42693932f9/dNBR%20a%20escala%20de%20grises.PNG)
