# Diagnostico posterior a la actualizacion

Aplicar solo el apartado del sintoma. No repetir toda la migracion ni modificar
CSS o permisos generales sin comprobaciones previas.

## MariaDB: fallo PREIN o paquetes de ramas mezcladas

Consultar `rpm -qa 'MariaDB-*' | sort` y la version efectiva mediante SQL si
el servidor puede iniciarse con seguridad. El cliente 11.4 no acredita servidor
11.4. No repetir a ciegas un upgrade masivo ni usar `--nodeps` para omitir el
control RPM. Seguir el [flujo de MariaDB](actualizacion-mariadb.md), con respaldo
en frio, retirada controlada solo del servidor viejo e instalacion del candidato.
Comprobar el resultado de `mariadb-upgrade` antes de reabrir la aplicacion.

## PHP: CLI nuevo pero web antigua, o error de socket

La instalacion SCL no cambia automaticamente Apache ni cron. Revisar reglas
SetHandler completas, socket del pool, servicio `php83-php-fpm`, acceso y logs.
No deducir PHP web desde PATH ni desactivar SELinux para resolver un 503.
Seguir el [flujo de PHP](actualizacion-php.md), comprobar sintaxis antes de recargar
y verificar version/configuracion desde la informacion PHP de Moodle.

## La raiz devuelve 404 pero index.php abre

1. Confirmar DocumentRoot y Directory hacia `public`.
2. Revisar DirectoryIndex efectivo y logs privados del VirtualHost.
3. Si falta, integrar `DirectoryIndex index.php`, ejecutar `apachectl configtest`
   y recargar Apache cuando sea seguro reabrirlo.
4. Probar raiz y router por separado.

Fue la correccion del ensayo, no una causa universal de errores 404.

## Router not configured

Comprobar el fallback hacia `r.php` en la configuracion que atiende `public`.
Para una instalacion en la raiz del host, probar:

```text
https://<HOST_MOODLE>/core/check/controller/test
```

Respuesta esperada de la ruta ensayada: `Test controller response`. Si falla,
revisar rutas y PHP-FPM; no ocultar el problema con una bandera.

Tras verificar la ruta, cuando la configuracion manual requiera declararlo,
agregar antes de cargar `lib/setup.php` en `config.php`:

```php
$CFG->routerconfigured = true;
```

La declaracion no configura Apache ni acredita todas las rutas. Ver la
[guia oficial](https://docs.moodle.org/503/en/Upgrading) para otras topologias.

## Composer installed data not found

Comprobar paquete, `composer.lock` y `vendor`. Seguir el apartado Composer del
[procedimiento](procedimiento-actualizacion.md) con el PHP elegido. No reemplazar
el lock mediante `composer update`. Comprobar Entorno despues de instalar
dependencias; no atribuir todos los fallos visuales a Composer.

## Curso vacio con actividades accesibles

1. Identificar formato y version, sin cambiarlos.
2. Comprobar ubicacion en `public/course/format` y dependencias.
3. Abrir un recurso existente por enlace directo.
4. Comparar con un tema de referencia y actualizar el formato si corresponde.
5. Purgar caches y revisar las mismas secciones.

Actualizar Multitopic 4.5.3 a 5.3.0 recupero contenido y banner en el ensayo.
No fue necesario recargar imagenes; no prescribir restauraciones masivas sin
evidencia de perdida.

## Paneles superpuestos o desalineados

Registrar tema, version, pagina y ancho de ventana. Comprobar controles de abrir
y cerrar, layout y CSS personalizado. Comparar con Boost sin borrar ajustes del
otro tema. No aplicar CSS global para un fallo localizado.

Adaptable 502.2.1 mantuvo superposiciones y advertencia de incompatibilidad con
5.3: un upgrade correcto no resolvio ese problema. Los desajustes de Moove y
Trema quedaron sin causa determinada; configuracion es una hipotesis.

## Descarga o instalacion ZIP fallida

Usar `curl -fL`, verificar URL y tag exacto y ejecutar `unzip -t`. Una respuesta
puede contener HTML en lugar del paquete. Ante 404 revisar tambien propietario
y repositorio, no solo el tag. No desactivar TLS ni ignorar errores de extraccion.

Si el instalador web informa acceso de escritura denegado, usar el flujo CLI
con propiedad controlada; no hacer escribible todo el arbol de codigo.

## Restauracion de curso en espera

Revisar tareas pendientes y planificacion. Una restauracion asincrona puede
necesitar cron. Ejecutarlo con el PHP y usuario del procedimiento, revisar la
salida y volver al curso. No restaurar otra copia solo por no ver aun el resultado.

## Conexion a la base de datos

Comprobar servicio, conectividad y logs privados antes de cambiar credenciales.
No publicar configuracion o consultas sensibles. Si un error desaparece sin
diagnostico, registrar causa indeterminada, no una solucion supuesta.
