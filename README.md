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
| INVENTARIOS | valor inventariado, variaciones, faltantes y sobrantes |

Opcionales: si se agregan las hojas MERCANCIA VENDIDA o RENTABILIDAD, el dashboard
las detecta solo y las usa como fuente preferente de costo y margen.

## Pestanas

Resumen, Ventas, Compras, Costo (food cost), Rentabilidad, Participacion,
Inventarios y Ajustes de inventario (con buscador, paginacion y exportacion a Excel).

## Notas de calculo

- El parser es tolerante a los nombres de columna: busca por palabras clave, no por posicion exacta.
- Los numeros se leen en formato es-CO (punto de mil, coma decimal, simbolo de moneda).
- Sin hoja de mercancia vendida, el costo se calcula como compras netas / ventas base:
  es un food cost de compras, util para tendencia pero no exacto para cierre contable.
  Con dos meses de conteo en INVENTARIOS se habilita el costo con ajuste de inventario.
- Meta de food cost: 35%.

## Despliegue

Sitio estatico sin build. 'vercel.json' desactiva cualquier paso de compilacion.
