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
| Grid 405.1.5 a 500.1.2, build 2025040707 | Actualizador CLI completo correctamente en ensayo 5.3 | Soporte declarado solo 5.0; compatibilidad funcional y aprobacion de produccion pendientes |
| Onetopic 4.4.08 a 5.1.0(Minitas), build 2026040500.02 | Actualizador CLI completo correctamente en ensayo 5.3 | Soporte declarado solo 5.1; prueba funcional pendiente |
| Tiles 4.5.0.28 a 5.1.0.2, build 2026012570 | Actualizador CLI completo correctamente en ensayo 5.3 | Soporte declarado solo 5.1; prueba funcional pendiente |
| Adaptable 405.2.8 a 502.2.1, build 2026041207 | Actualizador CLI completo correctamente en ensayo 5.3 | Soporte declarado solo 5.2; revision visual y funcional pendiente |
| Dependencias Composer desde composer.lock, sin desarrollo, scripts ni plugins; autoload optimizado | Extraccion real completada; check-platform-reqs --no-dev muestra success; mantenimiento desactivado y Apache active | No acredita desaparicion de avisos web ni correccion del router |

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

- Adaptable 405.2.8 mostraba paneles superpuestos y controles defectuosos en 5.3.
  Actualizado a 502.2.1; tras reabrir y seleccionarlo, persiste superposicion de
  paneles sobre el contenido en portada, administracion y curso. Pendiente
  inspeccionar ajustes de layout, sin causa confirmada. Campos Custom CSS y H5P
  Custom CSS vacios en la captura revisada. El propio tema muestra advertencia
  explicita de incompatibilidad con Moodle 5.3; no presentarlo como compatible.
- Tras instalar dependencias y reabrir el laboratorio, Composer installed data
  not found ya no aparece en Other checks de la comprobacion web mostrada.
  /core/check/controller/test devuelve Test controller response antes de agregar
  $CFG->routerconfigured = true; en config.php, antes de cargar lib/setup.php.
  Tras esa declaracion desaparece Router not configured de Entorno. No fue
  necesario cambiar handlers ni reglas Apache. Esto verifica la ruta probada y
  la declaracion, no todas las rutas ni el funcionamiento de los adicionales.
- Un episodio transitorio de Database connection failed se recupero segun el
  relato de la administradora. Causa no determinada.
- Gestor de certificados emite aviso de metadatos subplugintypes obsoletos.
- Certificados 5.0.11 se descargo y su ZIP paso la prueba; NO se instalo. El
  soporte declarado revisado llegaba a Moodle 5.2, no 5.3.
- Imagenes de tarjetas/tema siguen pendientes de revision. No asumir perdida
  general de archivos ni prescribir recarga masiva de imagenes.

## Dependencias Composer

La ausencia inicial de vendor se abordo conservando composer.lock. Se descargo
un PHAR con version fijada y se verifico su checksum oficial. Tras una simulacion,
la instalacion real se ejecuto bajo mantenimiento con PHP 8.3, sin dependencias
de desarrollo, scripts ni plugins y con autoload autoritativo. Se ajustaron la
propiedad del nuevo vendor y su contexto SELinux antes de reabrir Apache.

check-platform-reqs --no-dev mostro todos los requisitos listados en success con
PHP 8.3.35. ctype y mbstring figuraban provistos por polyfills; una consulta
posterior de modulos confirmo ambas extensiones nativas en CLI. No extrapolar
esa comprobacion a la configuracion efectiva de PHP-FPM.

Tras reabrir, se observa mejora parcial del control del indice del curso: permite
abrir y cerrar la navegacion. No atribuir causalidad exclusiva a Composer ni
declarar el tema validado; la revision visual conjunta sigue pendiente.

El aviso de version del paquete raiz moodle/moodle no detectada, con fallback a
1.0.0, no equivale a cambiar la version de Moodle instalada. La instalacion de
bibliotecas no demuestra por si sola que el router o la interfaz funcionen.

## Compatibilidad revisada

### Alternativa visual con Boost

Ante la superposicion persistente en Adaptable, se selecciono Boost. El ajuste
theme_boost/unaddableblocks permite quitar navigation y settings de la lista de
bloques excluidos, conservando course_list si se desea. Esto habilita agregarlos;
no crea instancias automaticamente ni determina su alcance.

Tras agregar Navigation, una captura del curso muestra navegacion derecha,
indice izquierdo y contenido central sin superposicion visible. Pendiente
validar enlaces y bloque Administration. La configuracion inicial pertenecia al
curso; para alcance global se configuro desde Site home, siguiendo Page contexts
Display throughout the entire site. La administradora confirma que aparece bien;
no equivale a validar individualmente todas las paginas. No borrar
Adaptable ni su configuracion como requisito de esta alternativa.

### Investigacion adicional de Adaptable para 5.3

Consulta del 2026-10-17: ultima release publica encontrada V502.2.1, publicada el
2026-10-06 para Moodle 5.2. Las ramas publicas consultadas no incluyen MOODLE_503;
main declara soporte 5.2. No se localizo paquete publico declarado compatible 5.3.

El autor Gareth Barnard publico el 2026-10-16 el articulo
[Dark mode](https://www.elearningworld.org/dark-mode/), donde describe trabajo
en modo oscuro de Adaptable en una version de desarrollo, junto con cambios en
controles de paneles, en contexto de Moodle 5.3. Esto acredita trabajo en curso,
no una version estable descargable ni un plazo de publicacion. Preguntar al autor
por disponibilidad de prueba seria el siguiente paso; no se lo ha contactado.

Las versiones siguientes se verificaron el 2026-10-17. Son datos de publicaciones,
no certificaciones de compatibilidad independientes. Actualizar la consulta antes
de planificar otro despliegue.

| Componente | Referencia revisada | Rango declarado | Estado del ensayo |
| --- | --- | --- | --- |
| Multitopic | v5.3.0, estable | Moodle 5.3 en catalogo | Actualizado; renderizado recuperado |
| Collapsed Topics | V503.0.2, RC | 5.3 | Actualizado; prueba funcional pendiente |
| Adaptable | V502.2.1, estable | 5.2 | Upgrade CLI exitoso en ensayo 5.3; persiste superposicion visual de paneles |
| Grid | V500.1.2, estable | 5.0 | Actualizado en ensayo 5.3; upgrade CLI exitoso, prueba funcional pendiente |
| Onetopic | 5.1.0(Minitas), commit bf59f12f670d88892112be2205085fa25377a181 | 5.1 | Actualizado en ensayo 5.3; upgrade CLI exitoso, prueba funcional pendiente |
| Tiles | 5.1.0.2, commit b22538a2fb7bbcbd41d6400358d22ddfc5ad307e | 5.1 | Actualizado en ensayo 5.3; upgrade CLI exitoso, prueba funcional pendiente |
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
