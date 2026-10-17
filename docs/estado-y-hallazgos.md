# Estado y hallazgos

Fecha de corte: 2026-10-17. Evidencia de laboratorio; no aprobacion de produccion.

## Resultados confirmados

| Cambio | Resultado observado | Limite |
| --- | --- | --- |
| Core 4.5.8 a 5.3 Build 20261005 | Actualizador CLI completo correctamente | No valida todos los adicionales |
| DocumentRoot y Directory hacia public | Acceso web al nuevo arbol | No valida todas las rutas |
| DirectoryIndex index.php explicito dentro del Directory del sitio | Portada por ruta raiz dejo de devolver 404 | No se registro validacion final del router |
| Multitopic 4.5.3 a 5.3.0, build 2026100502 | Reaparecieron pestanas, contenido y banner con Boost y Adaptable | Pruebas exhaustivas de actividades pendientes |
| Collapsed Topics a 503.0.2, build 2026100701 | Actualizador CLI completo correctamente | MATURITY_RC; prueba funcional pendiente |

## Incidentes

### Entrada por directorio devolvia 404

La raiz devolvia una pagina de error generada por Moodle, mientras index.php
explicito mostraba la portada. Tras agregar DirectoryIndex index.php al bloque
Directory del sitio, validar la sintaxis de Apache y recargarlo, la portada por
raiz funciono. FallbackResource /r.php ya estaba configurado. No generalizar este
ajuste a todos los servidores sin revisar su configuracion efectiva.

### Curso aparentemente vacio

El curso conservaba recursos accesibles por enlaces individuales, pero su vista
principal no mostraba secciones con Boost ni Adaptable. Usaba Multitopic 4.5.3.
Actualizar ese formato a 5.3.0 recupero la vista y el banner sin volver a subir
la imagen. La evidencia no justificaba declarar perdidos los archivos.

### Descargas y errores de escritura

- Una descarga del catalogo guardo HTML en lugar de ZIP. unzip -t detecto el problema.
- Otra descarga devolvio 404 por un nombre de usuario GitHub mal escrito en la
  variable de URL. La atribucion inicial del asistente a una v minuscula era
  incorrecta: el tag estaba bien escrito con V mayuscula.
- La descarga del tag correcto paso la prueba del archivo comprimido.
- No usar curl sin comprobar despues que se obtuvo el tipo de paquete esperado.
- Una prueba de integridad del ZIP no acredita firma, compatibilidad ni funcionamiento.

## Pendientes

- Adaptable 405.2.8 muestra paneles superpuestos y controles defectuosos en 5.3.
- Avisos de entorno: Composer installed data not found y Router not configured.
- vendor no existe en la raiz consultada. composer.json y composer.lock existen;
  require incluye bibliotecas de ejecucion. Composer no fue encontrado en PATH.
- Descarga y verificacion de Composer fueron propuestas; resultado aun no recibido.
- Un episodio transitorio de Database connection failed se recupero segun el
  relato de la administradora. Causa no determinada.
- Gestor de certificados emite aviso de metadatos subplugintypes obsoletos.
- Certificados 5.0.11 se descargo y su ZIP paso la prueba; NO se instalo. El
  soporte declarado revisado llegaba a Moodle 5.2, no 5.3.
- Imagenes de tarjetas/tema siguen pendientes de revision. No asumir perdida
  general de archivos ni prescribir recarga masiva de imagenes.

## Compatibilidad revisada

Las versiones siguientes se verificaron el 2026-10-17. Son datos de publicaciones,
no certificaciones de compatibilidad independientes. Actualizar la consulta antes
de planificar otro despliegue.

| Componente | Referencia revisada | Rango declarado | Estado del ensayo |
| --- | --- | --- | --- |
| Multitopic | v5.3.0, estable | Moodle 5.3 en catalogo | Actualizado; renderizado recuperado |
| Collapsed Topics | V503.0.2, RC | 5.3 | Actualizado; prueba funcional pendiente |
| Adaptable | V502.2.1, estable | 5.2 | No instalado; soporte 5.3 no confirmado |
| Grid | V500.1.2, estable | 5.0 | No instalado; soporte 5.3 no confirmado |
| Onetopic | master 5.1.0(Minitas), no paquete fijo | 5.1 | No instalado; soporte 5.3 no confirmado |
| Tiles | master 5.1.0.2, no paquete fijo | 5.1 | No instalado; soporte 5.3 no confirmado |
| Mail | 2.18, build 2026100500 | Incluye 5.3 | Ya instalado, sin reemplazo |
| eMail Test | 3.1.8, build 2026042001 | Incluye 5.3 | Ya instalado, sin reemplazo |

Fuentes primarias:

- [Moodle: actualizar a 5.3](https://docs.moodle.org/503/en/Upgrading).
- [Multitopic v5.3.0](https://github.com/james-cnz/moodle-format_multitopic/releases/tag/v5.3.0).
- [Collapsed Topics V503.0.2](https://github.com/gjbarnard/moodle-format_topcoll/tree/V503.0.2).
- [Adaptable V502.2.1](https://github.com/gjbarnard/moodle-theme_adaptable/tree/V502.2.1).
- [Grid V500.1.2](https://github.com/gjbarnard/moodle-format_grid/tree/V500.1.2).
- [Onetopic, repositorio del autor](https://github.com/davidherney/moodle-format_onetopic).
- [Tiles, repositorio del autor](https://bitbucket.org/dw8/moodle-format_tiles/).
- [Mail, versiones](https://marketplace.moodle.com/plugins/1048/versions).
- [eMail Test, versiones](https://marketplace.moodle.com/plugins/3723/versions).
- [Gestor de certificados, versiones](https://marketplace.moodle.com/plugins/2233/versions).

## Preparacion de produccion

Antes del despliegue real: completar componentes pertinentes y revision funcional,
resolver o aceptar explicitamente restricciones de soporte, definir recuperacion y
avisar a las administradoras sobre cambios visuales esperados. Capturar referencias
y conservar configuraciones de personalizacion antes del cambio. No anunciar una
reconstruccion obligatoria de portadas a partir de un fallo de renderizado no diagnosticado.
