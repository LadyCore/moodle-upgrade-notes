# Actualizacion de PHP 8.1 a 8.3 en paralelo

Segunda etapa: despues de [MariaDB](actualizacion-mariadb.md) y antes de
[Moodle](procedimiento-actualizacion.md). Instalacion PHP 8.3 SCL de Remi sobre
CentOS Stream 9 con Apache y FPM. Conserva PHP anterior y cambia explicitamente
las invocaciones web y CLI; no sustituye el modulo PHP del sistema.

Las colecciones `php83-php-*` permiten instalacion paralela; no son lo mismo
que cambiar el stream modular de `php-*`. Consultar la
[FAQ de Remi](https://blog.remirepo.net/pages/English-FAQ) y comprobar repositorios
adecuados al sistema. El ensayo utilizo `remi-safe` ya configurado; no acredita
una instalacion desde cero de ese repositorio.

## 1. Inventario y respaldo

Con privilegios administrativos, sustituir marcadores:

```bash
MOODLE='<RUTA_MOODLE_ACTUAL>'
WEBUSER='<USUARIO_SERVICIO_WEB>'
HTTPD_SERVICE='<SERVICIO_APACHE>'
RESPALDO='<DIRECTORIO_RESPALDO_NUEVO_FUERA_WEB>'
PHP_ANTERIOR='<RUTA_PHP_CLI_ACTUAL>'
PHP83='/opt/remi/php83/root/usr/bin/php'
FPM83='/opt/remi/php83/root/usr/sbin/php-fpm'
```

```bash
"$PHP_ANTERIOR" -v
rpm -qa 'php*' | sort
dnf repolist --enabled
dnf module list php
ls /etc/php.d
```

Verificar que Moodle y sus plugins funcionen con el PHP destino antes de
seleccionarlo. Inventariar extensiones, limites y pool actuales. No deducir
uso web a partir de `php -v`. Localizar configuracion efectiva de FPM y Apache.

Preparar respaldo nuevo y restringido, sin sobreescribir uno previo:

```bash
umask 077
mkdir "$RESPALDO"
tar -czf "$RESPALDO/php-apache.tgz" /etc/php* /etc/httpd
chmod 600 "$RESPALDO/php-apache.tgz"
gzip -t "$RESPALDO/php-apache.tgz"
tar -tzf "$RESPALDO/php-apache.tgz" | head -n 15
```

Este ejemplo presupone PHP anterior no SCL en `/etc/php*`. Agregar configuraciones
de otras ubicaciones si corresponden. El archivo no contiene RPM ni datos de la
plataforma; mantener tambien el punto de retorno consistente. Inspeccion gzip
no equivale a una restauracion probada.

## 2. Instalar PHP 8.3 y extensiones

El conjunto del ensayo incluyo LDAP; no eliminar dependencias de autenticacion
al construir la lista nueva. Agregar solo extensiones que requiera la instalacion.

```bash
dnf install --assumeno php83-php-{cli,fpm,opcache,xml,mbstring,sodium} php83-php-{gd,intl,mysqlnd,ldap,soap,pecl-zip}
```

**Control:** transaccion paralela sin retirada del PHP original ni componentes
inesperados. Si faltan paquetes, revisar repositorios y nombres antes de confirmar.

```bash
dnf install php83-php-{cli,fpm,opcache,xml,mbstring,sodium} php83-php-{gd,intl,mysqlnd,ldap,soap,pecl-zip}
"$PHP83" -v
"$PHP83" -m
```

Comprobar paquetes y extensiones requeridos. Esto no cambia por si solo el PHP
de Apache, `php` en PATH ni las tareas programadas. No retirar todavia PHP 8.1.

## 3. Trasladar ajustes seleccionados

Leer directivas activas del origen y verificar limites de carga, memoria,
tiempos, zona horaria, OPcache y opciones del pool. No copiar `php.ini` entero
ni archivos de carga de extensiones antiguas a la coleccion nueva.

Si el origen es `/etc/php.ini`, esta seleccion fue utilizada para conservar sus
ajustes locales en el ensayo:

```bash
grep -E '^(max_|memory_|post_|upload_|pcre\.)' /etc/php.ini
```

Revisar la seleccion. Antes de crear `99-moodle.ini`, comprobar que no exista
y que no haya otros overrides con los mismos parametros. Si ya existe, conservar
una copia y fusionar manualmente, no truncarlo. En un destino nuevo:

```bash
grep -E '^(max_|memory_|post_|upload_|pcre\.)' /etc/php.ini > /etc/opt/remi/php83/php.d/99-moodle.ini
cat /etc/opt/remi/php83/php.d/99-moodle.ini
"$PHP83" --ini
```

El filtro solo recoge directivas sin espacios iniciales y no captura ajustes en
otros INI, pools o configuracion web. Es una ayuda para el origen del ensayo,
no un inventario completo. Revisar las otras fuentes efectivas y no importar
valores de otra maquina como si fueran recomendaciones universales.

En el ensayo se conservaron limites locales y `pcre.jit`; no se hizo tuning de
memoria con mediciones nuevas. Su carga en web debe verificarse mas adelante.

## 4. Revisar pool y poner en servicio FPM

```bash
grep '^pm' /etc/opt/remi/php83/php-fpm.d/www.conf
grep -E '^(user|group|listen)' /etc/opt/remi/php83/php-fpm.d/www.conf
"$FPM83" -t
```

Comparar con el pool anterior: usuario, grupo, socket, permisos/ACL y parametros
de procesos. En FPM con `pm=dynamic`, mantener una configuracion coherente entre
minimos, maximos e hijos. No trasladar cantidades de procesos sin considerar
memoria disponible y otras cargas. Si se edita `www.conf`, respaldarlo y repetir
la prueba de sintaxis antes de aplicar cambios.

```bash
systemctl enable --now php83-php-fpm
systemctl is-active php83-php-fpm
```

**Control:** sintaxis satisfactoria y servicio `active`. Todavia no prueba que
Apache este usando el nuevo FPM. No detener el FPM anterior durante este paso.

## 5. Cambiar sockets en Apache

Reservar una ventana, activar mantenimiento usando el PHP anterior y pausar
tareas automaticas si existen. Esperar procesos activos. El cambio de FPM no
migra el esquema Moodle, pero requiere verificar compatibilidad y acceso.

```bash
sudo -u "$WEBUSER" "$PHP_ANTERIOR" "$MOODLE/admin/cli/maintenance.php" --enable
grep -n SetHandler /etc/httpd/conf.d/*.conf
```

Inspeccionar los bloques completos: un grep no demuestra que las ramas
condicionales esten activas. En el caso de referencia habia reglas locales en
`php-fpm.conf` y `php.conf` hacia `/run/php-fpm/www.sock`, ademas del archivo
de la coleccion. La regla FPM nueva usa:

```apache
SetHandler "proxy:unix:/var/opt/remi/php83/run/php-fpm/www.sock|fcgi://localhost"
```

Si esos mismos dos archivos y ruta estan confirmados, la sustitucion acotada
es la siguiente. No ejecutarla si el inventario difiere o existe la copia
`.antes-php83`; adaptar el archivo efectivo y conservar sus ramas condicionales.

```bash
cd /etc/httpd/conf.d
sed -i.antes-php83 's|/run/php-fpm/|/var/opt/remi/php83/run/php-fpm/|g' php-fpm.conf php.conf
httpd -t
```

No incluir `php83-php.conf` en esa sustitucion: ya contiene la ruta nueva.
No editar VirtualHost ni DocumentRoot en esta etapa. Revisar diferencias,
socket real y copias antes de recargar. Los respaldos no deben terminar en
`.conf` para evitar que Apache los vuelva a cargar.

**Control:** `Syntax OK`. No confundir `httpd` sin `-t` con una prueba de sintaxis.
Un aviso de ServerName global no exige modificarlo como parte de esta migracion.

```bash
systemctl reload "$HTTPD_SERVICE"
systemctl is-active "$HTTPD_SERVICE"
```

Si FPM necesita reiniciarse despues de editar sus ajustes, volver a probar
sintaxis y reiniciar `php83-php-fpm` en la ventana; verificar su estado antes
de reabrir. No desactivar SELinux ni ampliar permisos del socket como atajo.

## 6. Verificar PHP web y CLI

Desactivar mantenimiento con PHP nuevo y comprobar Moodle:

```bash
sudo -u "$WEBUSER" "$PHP83" "$MOODLE/admin/cli/maintenance.php" --disable
```

En Administracion del sitio > Servidor > Informacion de PHP y Entorno, verificar:

- Version destino y API FPM/FastCGI.
- `php.ini` y directorio de INI de `/etc/opt/remi/php83`.
- Carga de `99-moodle.ini`, extensiones y valores efectivos necesarios.
- Conexion a base, acceso a un curso y apertura del editor existente.

No publicar `phpinfo` ni crear una pagina publica permanente para comprobarlo.
Si falla, mantener acceso restringido y consultar logs de Apache y FPM.

## 7. Cron y planificadores

Buscar invocaciones existentes sin suponer ubicacion ni periodicidad:

```bash
grep -R 'cli/cron.php' /etc/cron* /var/spool/cron
systemctl list-timers --all
```

Inspeccionar tambien scripts indirectos, supervisores y automatizacion externa.
Cambiar las tareas identificadas al ejecutable absoluto de PHP nuevo,
conservando usuario, frecuencia y registro. No cambiar globalmente `php` en PATH.
Probar manualmente:

```bash
sudo -u "$WEBUSER" "$PHP83" "$MOODLE/admin/cli/cron.php"
```

**Control:** ejecucion correcta, tareas revisadas y planificadores repuestos
con el binario correcto. No deducir exito solo por volver al prompt. En un
laboratorio manual no es obligatorio crear programacion nueva; en operacion
continua debe quedar resuelta segun la documentacion de Moodle. Aislar envios
e integraciones externas en copias de ensayo.

## 8. Recuperacion y cierre

Si solo se cambio PHP y no se actualizaron Moodle ni datos, puede revertirse
el enrutamiento a las copias Apache previas, comprobar sintaxis y servicio FPM
anterior, reponer las invocaciones CLI y verificar acceso. Si tareas o plugins
modificaron datos, aplicar ademas el plan consistente de recuperacion.
No usar este retorno de PHP como downgrade del Moodle ya actualizado.

Conservar PHP anterior y respaldos hasta aceptar las pruebas. Desinstalarlo es
una operacion posterior, no parte del ensayo documentado.

## Resultado de referencia

PHP 8.3.35 instalado en paralelo, FPM activo y habilitado, version web confirmada
como FPM/FastCGI con `99-moodle.ini` cargado, cron con el ejecutable nuevo correcto
y curso/editor Atto accesibles. PHP 8.1 se conservo. La revision integral de
valores efectivos y restauracion de respaldos no se completo. El pool se ajusto
posteriormente y se reinicio con estado activo; sus cantidades no son un perfil
universal de dimensionamiento. Ver la [matriz](estado-y-hallazgos.md).
