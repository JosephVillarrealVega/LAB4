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
  Se identifican las zonas quemadas mediante un umbral que nosotros proporcionamos y su visualización
  
```js
var DIFF_UPPER_THRESHOLD = 0.75; 
var zonas_quemadas = change.lt(DIFF_UPPER_THRESHOLD);
Map.addLayer(zonas_quemadas.updateMask(zonas_quemadas),{palette:"D5421E"},'zonas quemadas',1);

```

</details>


hola soy yo

![esto es una imagen](https://github.com/JosephVillarrealVega/LAB4/blob/c54bd30d8e2bae43cd9c77aefddda08287e5e418/ejempl.PNG)
