# Rentabilidad - Casa de Nadie

Dashboard web de rentabilidad y food cost para Casa de Nadie. Un solo archivo
'index.html' con HTML + CSS + JavaScript vanilla: sin build, sin framework, sin npm.

## Como funciona

No hay backend ni base de datos. El dashboard lee un Google Sheet publicado,
exportado como XLSX, y se re-sincroniza cada 60 segundos sin recargar la pagina.

- Fuente: hoja de calculo "Formato de Informes Casa de Nadie" (export?format=xlsx)
- Deteccion de cambios: hash FNV-1a con muestreo cada 521 bytes; solo re-renderiza si el archivo cambio
- Fallback: si se abre como archivo local o falla el fetch, aparece un selector para cargar el Excel a mano

El Sheet debe estar compartido como "Cualquiera con el enlace: Lector".

## Hojas que consume

| Hoja | Uso |
|---|---|
| VENTAS | ventas base y neta, unidades, rankings por articulo y familia, clasificacion Cocina/Bar |
| COMPRAS | compras netas por proveedor, familia, articulo, mes y centro de costos |
| INVENTARIOS | valor inventariado por conteo. Cada fecha de documento es un conteo: el primero del periodo es el Inventario Inicial y el segundo el Inventario Final |

| RENTABILIDAD | ficha tecnica por articulo (Precio, Base, Coste, Margen). Es la unica fuente
de la pestana Rentabilidad y la primera opcion de costo unitario en Analisis Comercial |

El encabezado de cada hoja se detecta solo (no tiene que estar en la fila 1) y las
columnas se buscan por palabras clave, asi que renombrarlas no rompe el tablero.

## Pestanas

Resumen, Ventas, Compras, Costo, Rentabilidad, Participacion, Inventarios,
Mercancia Vendida (juego de inventarios con exportacion a Excel) y Analisis Comercial.

### Analisis Comercial

Tablero ejecutivo para decidir que impulsar en la carta. Consolida ventas, costos y
rentabilidad del mismo periodo filtrado:

- Nueve indicadores globales: venta total, base neta, costo total, utilidad bruta,
  margen bruto, ticket promedio, productos vendidos, numero de facturas y costo
  promedio por plato.
- Rankings: mejor margen, mas vendidos por unidades, mayor utilidad total y bajo margen.
- Productos que los meseros deben ofrecer primero: puntaje automatico que pesa
  margen 40%, rotacion 30% y utilidad 30%.
- Ingenieria del Menu (Kasavana-Smith): Estrella, Caballo de batalla, Rompecabezas y Perro.
  El umbral de popularidad es el 70% de las unidades promedio; la rentabilidad se mide
  con el margen de contribucion unitario frente al promedio ponderado.
- Graficos: top 10 mas rentables, top 10 mas vendidos, top 10 mayor utilidad,
  ventas por familia, costos por familia, participacion por categoria y tendencia mensual.

Costo por producto, en este orden de prioridad:

1. Ficha tecnica: columna Coste de la hoja RENTABILIDAD.
2. Precio de compra: costo unitario promedio del insumo en COMPRAS.
3. Tasa de su centro de costos, calibrada para que la suma de los costos por producto
   cuadre exactamente con la Mercancia Vendida del periodo.

El tercer paso es el que garantiza que el modulo no invente cifras: por construccion
suma lo mismo que el juego de inventarios. Si las fichas y los precios de compra ya
sumaran mas que la Mercancia Vendida del periodo (no queda bolsa residual que calibrar),
los costos del centro se escalan y el tablero avisa de forma explicita en la nota
superior. En los dos casos la suma de los costos por producto es igual al costo total.

## Notas de calculo

- El parser es tolerante a los nombres de columna: busca por palabras clave, no por posicion exacta.
- Los numeros se leen en formato es-CO (punto de mil, coma decimal, simbolo de moneda).
- Fuente unica de verdad: la funcion calc() del index.html. Ningun modulo recalcula
  por su cuenta; todos leen el mismo motor, por eso los numeros cuadran entre pestanas.
- Ventas = suma de la columna Base de VENTAS (Base Neta: sin IVA, sin propinas).
  Se incluyen notas credito y descuentos (valores negativos) para cuadrar con el reporte.
- Todos los indicadores respetan a la vez el filtro de Mes, Sede y Centro de costos.
- Mercancia Vendida = Inventario Inicial + Compras - Inventario Final.
  Excel equivalente: =+[@[Inv Inicial]]+[@Compras]-([@[Inv Final]])
- Costo Mercancia Vendida (%) = Mercancia Vendida / Ventas.
  Excel equivalente: =+[@[Mercancia vendida]]/[@[Ventas]]
- Inventario Inicial = primer conteo del periodo seleccionado.
  Inventario Final = segundo (ultimo) conteo del periodo seleccionado.
- Utilidad Bruta = Ventas Base Neta - Mercancia Vendida. Su % se calcula sobre las ventas.
- Con un solo conteo en el periodo el juego de inventarios no es aplicable: la mercancia
  vendida cae a las compras del periodo y el dashboard lo indica de forma explicita.
- Compras del juego de inventarios: solo se toman las compras cuyo Centro de Costo es
  Bar o Cocina. Todo lo demas (aseo, papeleria, utensilios, etc.) queda fuera del calculo
  y se muestra aparte en la pestana Compras para que el descarte sea auditable.
- Todas las tarjetas del Resumen muestran el valor resumido en millones y, debajo,
  el valor exacto en pesos.
- La pestana Rentabilidad no recalcula nada: lee la hoja RENTABILIDAD tal como esta.
  Filtra por Sede y Centro de Costos, descarta las tarifas de Rappi, personal y empleados,
  y aparta como atipicos los registros con Coste fuera del rango 0-100% del precio
  (se listan en la tabla, pero no entran en promedios ni graficos).
- Margen de la hoja RENTABILIDAD = Coste / Precio, es decir food cost, no utilidad.
- Los graficos de una pestana oculta se redimensionan al mostrarla, para que las barras
  y sus etiquetas queden alineadas.

## Rendimiento

Con el volumen real (unas 14.000 filas de ventas, 5.000 de compras y 2.500 de fichas)
las medidas en el navegador son:

- Cambiar de mes, sede o centro de costos: alrededor de 0,15 s.
- Abrir una pestana por primera vez despues de un cambio de filtro: 0,3 a 0,6 s.
- Volver a una pestana ya dibujada: instantaneo.

Dos decisiones explican esos numeros:

1. Solo se dibuja la pestana visible. Un cambio de filtro marca las otras ocho y cada una
   se dibuja cuando se abre, no antes. Antes se redibujaban las nueve de golpe (unos 2 s).
2. Las tablas largas (la hoja RENTABILIDAD completa y la Ingenieria del Menu producto por
   producto) se pintan en tramos de 250 filas, con un boton para ir mostrando mas.

Lo que queda es el tiempo de la libreria XLSX al leer el archivo, que es proporcional al
tamano del Excel y ocurre una sola vez por sincronizacion.
- Meta de food cost: 35%.

## Despliegue

Sitio estatico sin build. 'vercel.json' desactiva cualquier paso de compilacion.
