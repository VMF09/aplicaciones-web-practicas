# Reflexiona
### 1. ¿Qué ruta relativa usará estaciones.html para cargar primavera.jpg? 

> ```estaciones.html``` está junto a la carpeta de imagenes , así que su ruta relativa será ```imagenes/primavera.jpg```.

### ¿Qué pasaría si movieras las páginas de cada estación a una carpeta paginas/? 

> Desde ```estaciones.html```, todos los enlaces tendrían que dirigir, por ejemplo, a ```paginas/primavera.html```. Desde cada página de estación, la imagen se encontraría con ```../imagenes/primavera.jpg``` y su enlace de vuelta sería ```../estaciones.html```.   También se habría que actualizar la ruta en la hoja CSS a ```../estilos.css```.

### Si las fotos miden 600 × 600 y deben verse a 300 × 300, ¿las recortas o cambias el tamaño con HTML/CSS?

> Se tiene que cambiar el tamaño con HTML o CSS, no se puede recortar. 
