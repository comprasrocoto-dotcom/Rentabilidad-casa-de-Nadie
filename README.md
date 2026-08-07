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

Opcional: si se agrega la hoja RENTABILIDAD, el dashboard la detecta sola y la usa
como fuente preferente de margen teorico por articulo.

## Pestanas

Resumen, Ventas, Compras, Costo, Rentabilidad, Participacion, Inventarios,
Mercancia Vendida (juego de inventarios con exportacion a Excel) y Nuevo Modulo
(estructura y navegacion listas, contenido pendiente).

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
- Rentabilidad por articulo: el costo unitario sale UNICAMENTE del precio de compra
  del insumo (promedio ponderado de COMPRAS, solo articulos comprados por unidad).
- Meta de food cost: 35%.

## Despliegue

Sitio estatico sin build. 'vercel.json' desactiva cualquier paso de compilacion.
