# Procedimiento de actualizacion

## 1. Alcance y condiciones de entrada

Aplicable a Moodle 4.5 que migra a 5.3 sobre CentOS Stream 9 Minimal, Apache,
PHP-FPM de Remi, MariaDB y SELinux. Adaptar usuarios, servicios y politicas para
otros sistemas. No aplicar los ejemplos a un cluster sin coordinar sus nodos.

Antes de continuar:

- Revisar requisitos en [MoodleDocs](https://docs.moodle.org/503/en/Upgrading) y
  Administracion del sitio > Servidor > Entorno. Resolver errores bloqueantes.
- Disponer de acceso administrativo, espacio para codigo y respaldos y una
  copia de ensayo separada. Esta guia no actualiza PHP o MariaDB.
- Inventariar plugins, subplugins y personalizaciones; fijar versiones o commits
  de los candidatos y comprobar requisitos y soporte.
- Definir y comprobar la recuperacion consistente de base, datos y codigo.

No continuar con respaldos incompletos o dependencias desconocidas. Un paquete
fuera de soporte requiere una decision explicita de ensayo, no una aprobacion
implicita porque el instalador acepte su requisito minimo.

## 2. Parametros

Sustituir todos los marcadores; no son rutas detectadas. Ejecutar bloques por
separado, con privilegios administrativos para tareas del sistema. Las variables
deben volver a definirse al abrir una nueva terminal.

```bash
MOODLE='<RUTA_MOODLE>'
DATOS='<RUTA_DATOS>'
PHPCLI='<RUTA_PHP_CLI>'
WEBUSER='<USUARIO_SERVICIO>'
WEBGROUP='<GRUPO_SERVICIO>'
RESPALDO='<RUTA_RESPALDO_FUERA_WEB>'
PREPARACION='<RUTA_PREPARACION_FUERA_WEB>'
HTTPD_SERVICE='<SERVICIO_APACHE>'
```

En Remi PHP 8.3 el ejecutable suele estar en
`/opt/remi/php83/root/usr/bin/php`; verificarlo, no usar `php` sin comprobarlo.
Identificar tambien el servicio PHP-FPM real, no asumir el nombre `php-fpm`.

```bash
"$PHPCLI" -v
"$PHPCLI" -m
sudo -u "$WEBUSER" "$PHPCLI" -v
```

**Control:** el usuario web ejecuta el PHP elegido. Verificar por separado
PHP-FPM mediante Entorno; la salida CLI no acredita la configuracion web.

## 3. Inventario y recuperacion

Registrar core, Entorno y Administracion del sitio > Plugins > Vista general
de plugins. En 4.5 puede listarse codigo adicional por CLI:

```bash
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/uninstall_plugins.php" --show-contrib
```

Esta opcion lista; no agregar opciones de desinstalacion. Registrar versiones,
dependencias y ubicacion de subplugins.

El respaldo debe cubrir base de datos, `moodledata`, codigo, `config.php` y
configuracion de servicios del mismo punto consistente. Un snapshot solo sirve
si incluye los discos y servicios implicados; coordinar bases externas. Una
copia independiente permite conservar un entorno funcional para comparar.

Aplicar el procedimiento de respaldo y restauracion ya probado para esa
topologia. No se prescribe un volcado SQL universal: su consistencia depende
del motor y almacenamiento. Capturar el respaldo final con escritores detenidos
en el paso 5, antes de modificar codigo o esquema.

## 4. Preparar codigo limpio

Obtener un paquete de version fija de [Moodle Downloads](https://download.moodle.org/).
Registrar URL y comprobar las sumas o firmas publicadas. No desactivar TLS.

```bash
PAQUETE='<ARCHIVO_TGZ_DESCARGADO>'
gzip -t "$PAQUETE"
tar -tzf "$PAQUETE" | head -n 10
mkdir "$PREPARACION"
tar -xzf "$PAQUETE" -C "$PREPARACION"
NUEVO="$PREPARACION/moodle"
```

Si el directorio ya existe, detenerse e inspeccionarlo; no mezclar instalaciones.
Verificar el prefijo del tar antes de definir `NUEVO`. Una prueba gzip correcta
no verifica autenticidad. Comprobar `version.php`, `public` y las utilidades CLI.

Copiar la configuracion a la raiz, no a `public`:

```bash
cp -p "$MOODLE/config.php" "$NUEVO/config.php"
cmp "$MOODLE/config.php" "$NUEVO/config.php"
```

Agregar solo los adicionales inventariados, usando candidatos adecuados. No
sobrescribir el core nuevo con directorios completos de la version anterior.
Consultar la [guia de plugins](actualizacion-plugins.md).

| Tipo | Destino en el arbol 5.3 |
| --- | --- |
| Actividad | `public/mod/<DIRECTORIO_PLUGIN>` |
| Formato | `public/course/format/<DIRECTORIO_PLUGIN>` |
| Tema | `public/theme/<DIRECTORIO_PLUGIN>` |
| Herramienta | `public/admin/tool/<DIRECTORIO_PLUGIN>` |
| Local | `public/local/<DIRECTORIO_PLUGIN>` |

Conservar subplugins del paquete padre y revisar modificaciones locales antes
del reemplazo. Comparar copias y comprobar componentes faltantes. En el ensayo
historico se conservaron inicialmente plugins antiguos para diagnostico; esa
decision no es una recomendacion de compatibilidad.

## 5. Interrumpir escrituras y capturar respaldo final

Pausar los planificadores identificados (cron, timers, trabajadores de tareas y
automatizaciones externas) y registrar su estado para reponerlo. Desde el codigo
todavia activo:

```bash
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/maintenance.php" --enable
systemctl stop "$HTTPD_SERVICE"
pgrep -af '[a]dmin/cli/cron.php'
```

Esperar a que terminen tareas activas y comprobar otros trabajadores conocidos.
`pgrep` es un control complementario, no bloquea nuevas ejecuciones. Mantenimiento
y parada de Apache no detienen por si solos cron. No terminar tareas a ciegas.

Con escritores detenidos, obtener el respaldo final o snapshot consistente
segun el paso 3 y confirmar que puede recuperarse antes de continuar.

## 6. Sustituir codigo

Preparar un directorio de respaldo restringido fuera de toda raiz HTTP. Verificar
que los destinos siguientes no existen y que todas las rutas son las previstas.

```bash
CODIGO_ANTERIOR="$RESPALDO/codigo-anterior"
mv -T -i "$MOODLE" "$CODIGO_ANTERIOR"
mv -T -i "$NUEVO" "$MOODLE"
chown -R "root:$WEBGROUP" "$MOODLE"
restorecon -RF "$MOODLE"
ls -ldZ "$MOODLE" "$MOODLE/public" "$MOODLE/config.php"
```

Ejecutar linea por linea. Detenerse ante error o pregunta de sobrescritura.
El directorio padre de respaldo debe existir. No dejar `config.php` antiguo
publicamente accesible. El usuario web necesita lectura y recorrido, no escritura
general sobre el codigo. No aplicar `chmod 777`.

`restorecon` aplica reglas SELinux existentes: con rutas personalizadas, definir
primero las reglas persistentes necesarias. No supone que toda ruta reciba el
contexto correcto ni sustituye revisar su politica. No aplicar a `moodledata`
los permisos o contextos del codigo.

## 7. Ajustar Apache

Integrar en el VirtualHost real la raiz `<RUTA_MOODLE>/public`, conservando TLS,
logs y manejador PHP-FPM. Para un sitio en la raiz del host:

```apache
DocumentRoot "<RUTA_MOODLE>/public"
<Directory "<RUTA_MOODLE>/public">
    Require all granted
    DirectoryIndex index.php
    FallbackResource /r.php
</Directory>
```

Es un fragmento, no un VirtualHost completo. Si Moodle esta en un subdirectorio,
adaptar la URL del router. No copiar un manejador FPM ajeno ni exponer archivos
sensibles para resolver errores de acceso.

```bash
apachectl configtest
```

**Control:** `Syntax OK`. Mantener Apache detenido hasta terminar el upgrade.
Referencia: [raiz web y enrutamiento](https://docs.moodle.org/503/en/Upgrading).

## 8. Dependencias Composer

Comprobar instrucciones del paquete. Si utiliza Composer o Entorno informa
dependencias ausentes, usar un Composer oficial verificado y el `composer.lock`
del paquete; no ejecutar `composer update` para sustituir ese lock.

Este perfil restringido se utilizo en el ensayo; comprobar su adecuacion al
paquete antes de repetirlo:

```bash
COMPOSER='<RUTA_COMPOSER_PHAR_VERIFICADO>'
cd "$MOODLE"
"$PHPCLI" "$COMPOSER" install --no-dev --no-scripts --no-plugins -a --dry-run
"$PHPCLI" "$COMPOSER" install --no-dev --no-scripts --no-plugins -a
"$PHPCLI" "$COMPOSER" check-platform-reqs --no-dev
chown -R "root:$WEBGROUP" "$MOODLE/vendor"
restorecon -RF "$MOODLE/vendor"
```

**Control:** instalacion y requisitos satisfactorios; lock conservado. Si el
paquete requiere scripts o plugins, detenerse y revisar sus instrucciones,
no retirar restricciones a ciegas. Verificar fuente e integridad del PHAR.

## 9. Actualizar la base

Las utilidades de administracion CLI permanecen en `admin/cli` en la raiz del
proyecto; no agregar `public` a estas rutas.

```bash
cd "$MOODLE"
sudo -u "$WEBUSER" "$PHPCLI" admin/cli/upgrade.php
```

Comprobar la version destino antes de confirmar `y`. Exigir finalizacion correcta
del core y adicionales. Ante un fallo, mantener el sitio cerrado y conservar la
salida en un registro privado antes de diagnosticar o reintentar.

## 10. Reabrir y verificar

Solo tras el upgrade correcto:

```bash
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/purge_caches.php"
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/maintenance.php" --disable
systemctl start "$HTTPD_SERVICE"
systemctl is-active "$HTTPD_SERVICE"
```

**Control:** servicio `active`, portada accesible y Entorno sin errores
bloqueantes. Probar una ruta del router; ver [diagnostico](diagnostico.md).
Aplicar la [validacion funcional](validacion.md).

Para procesar tareas pendientes, usar el mismo PHP y usuario:

```bash
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/cron.php"
```

En ensayo, aislar correos e integraciones externas antes de ejecutar tareas.
Revisar resultados y reponer los planificadores registrados con el PHP correcto.
Una ejecucion manual no configura periodicidad ni garantiza terminar todas las
tareas.

## 11. Recuperacion

Si el resultado no puede aceptarse, mantener la instalacion aislada. Restaurar
base, datos, codigo y configuracion del mismo punto consistente mediante el plan
probado. Verificar servicios y acceso antes de habilitar trafico y planificadores.

No ejecutar codigo viejo contra una base ya migrada. El directorio anterior es
respaldo de codigo, no un mecanismo de downgrade. La restauracion depende de la
topologia y no se reduce a un comando universal.
