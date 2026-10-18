# Validacion funcional

## Preparacion

Usar copias de cursos existentes que ya empleen el formato evaluado. No modificar
el formato del original para fabricar una prueba. Comparar con la referencia
previa cuando exista. Sin curso representativo, registrar `No probado`.

Separar instalacion CLI, soporte declarado y funcionamiento. Registrar tema,
formato y tipo de pagina para distinguir el origen de los fallos.

## Comprobaciones

| Area | Accion | Criterio esperado |
| --- | --- | --- |
| Entorno | Abrir comprobacion del servidor | Sin errores bloqueantes; avisos identificados |
| Acceso | Abrir raiz, iniciar sesion y salir | Sin 404 ni redirecciones incorrectas |
| Router | Probar ruta indicada en diagnostico | Respuesta del controlador, no solo ausencia del aviso |
| Portada | Abrir tarjetas e imagenes | Imagenes y enlaces accesibles |
| Curso | Recorrer secciones, abrir actividad y volver | Contenido conservado; navegacion operativa |
| Recursos | Abrir PDF, libro y enlace externo | Documento legible, indice operativo y destino externo comprobado |
| Foro | Revisar discusion e interaccion autorizada en copia | Contenido y acciones sin errores |
| Cuestionario | En copia, probar intento, guardado y resultado | Persistencia y calificacion coherentes |
| Permisos | Usar cuentas de prueba con roles pertinentes | Accesos y restricciones correctos |
| Finalizacion | Revisar condiciones y progreso en copia | Estados coherentes con las acciones |
| Mensajeria | Enviar entre dos cuentas de prueba | Recepcion real; notificaciones probadas por separado |
| Tareas | Revisar cron y tareas fallidas | Procesamiento esperado con PHP correcto |
| Certificados | Abrir plantilla y vista previa | Texto, fecha e imagenes correctos |
| Emision | Emitir para cuenta de prueba y verificar codigo/QR | Datos sustituidos y validacion coherente |

Son pruebas para reutilizar, no resultados ya obtenidos. Evitar envios a personas
reales o integraciones externas desde un laboratorio.

## Formatos

| Formato | Prueba minima |
| --- | --- |
| Multitopic | Recorrer pestanas y niveles, comprobar banner, abrir recurso y volver |
| Collapsed Topics | Contraer y desplegar secciones, abrir actividad y volver |
| Grid | Comprobar imagenes, abrir seccion desde cuadricula y regresar |
| Onetopic | Recorrer pestanas y comprobar correspondencia con secciones |
| Tiles | Abrir mosaicos, acceder a recurso y regresar |

Verificar titulos, orden, imagenes y enlaces. Una vista principal vacia no
demuestra perdida del recurso: comprobar acceso directo antes de restaurar.

## Temas y bloques

Probar portada, curso, actividad y administracion. Abrir y cerrar paneles,
desplazarse hasta el pie y variar el ancho de ventana. Distinguir superposicion
intencional de contenido inaccesible o controles tapados.

En Boost, si no pueden agregarse Navigation y Administration, revisar
`theme_boost/unaddableblocks`. Quitar `navigation` y `settings` de esa exclusion
permite agregarlos; no crea instancias. Configurar alcance desde la portada y
comprobar paginas pertinentes. No eliminar supuestos duplicados sin observarlos.

## Registro y aceptacion

| Fecha | Versiones core/plugin/tema | Funcion | Accion | Esperado | Obtenido | Estado |
| --- | --- | --- | --- | --- | --- | --- |
| <FECHA> | <VERSIONES> | <FUNCION> | <PASOS> | <CRITERIO> | <OBSERVACION> | Correcto / Fallo / No probado |

Mantener evidencia sensible en registros privados. No publicar participantes,
identificadores de cursos, enlaces privados ni capturas originales.

Aceptar solo con requisitos y funciones criticas comprobados, fallos resueltos
o aceptados explicitamente, y recuperacion disponible. Un pendiente no ejecutado
no es un fallo ni una prueba superada.
