# Actualizacion de MariaDB 10.11 a 11.4

Primera etapa del recorrido: MariaDB, [PHP](actualizacion-php.md) y
[Moodle](procedimiento-actualizacion.md). Metodo para servidor unico de MariaDB
Community instalado mediante RPM oficiales sobre CentOS Stream 9. No aplicar
sin adaptacion a Galera, replicacion, contenedores o bases externas.

El ensayo paso de servidor 10.11.15 a 11.4.13. Los comandos siguientes generalizan
el flujo resuelto; no recomiendan fijar hoy esos parches historicos. Revisar la
[ruta oficial 10.11 a 11.4](https://mariadb.com/docs/server/server-management/install-and-upgrade-mariadb/upgrading/mariadb-community-server-upgrade-paths/upgrading-from-mariadb-10-11-to-mariadb-11-4)
y las opciones de configuracion retiradas o modificadas antes del cambio.

## 1. Parametros e inventario

Ejecutar con privilegios administrativos, linea por linea. Sustituir marcadores:

```bash
MOODLE='<RUTA_MOODLE_ACTUAL>'
PHPCLI='<PHP_COMPATIBLE_CON_MOODLE_ACTUAL>'
WEBUSER='<USUARIO_SERVICIO_WEB>'
HTTPD_SERVICE='<SERVICIO_APACHE>'
DBSERVICE='<SERVICIO_MARIADB>'
DB='<NOMBRE_BD_MOODLE>'
RESPALDO='<DIRECTORIO_RESPALDO_NUEVO_FUERA_WEB>'
REPOFILE='<ARCHIVO_REPOSITORIO_MARIADB>'
REPOID='<ID_REPOSITORIO_MARIADB>'
MARIADB_SERVER_RPM='<NOMBRE_VERSION_RELEASE_RPM_SERVIDOR_11_4>'
DBOPTS=(--protocol=socket)
```

El nombre RPM del ensayo fue `MariaDB-server-11.4.13-1.el9`; sustituir por el
candidato elegido y disponible. Confirmar autenticacion administrativa local
para los clientes MariaDB, sin poner contrasenas en la linea de comandos.

```bash
rpm -qa 'MariaDB-*' | sort
dnf repolist --enabled
mariadb "${DBOPTS[@]}" -e 'SELECT @@version, @@datadir;'
mariadb "${DBOPTS[@]}" -e 'SHOW DATABASES;'
my_print_defaults mysqld
```

**Control:** identificar version del servidor real, datadir y paquetes. La
version del cliente no prueba la del servidor. Registrar configuracion, plugins
de almacenamiento, cifrado, ubicaciones externas y repositorios efectivos.
Si el servidor esta detenido, consultar su configuracion y arrancar solo cuando
sea seguro; la ausencia de `datadir` en un filtro no autoriza asumir una ruta.

### Conexion local y TLS

En el ensayo un cliente nuevo exigio SSL a un servidor local sin ese soporte.
La consulta se completo por socket con `--skip-ssl`. Solo si se confirma ese
mismo escenario local, puede definirse `DBOPTS=(--protocol=socket --skip-ssl)`.
No deshabilitar TLS en conexiones remotas ni cambiar la configuracion global
para resolver una consulta. Las opciones de cliente no configuran TLS de Moodle.

## 2. Respaldos y disponibilidad del paquete

Crear un destino nuevo y restringido; detenerse si ya existe:

```bash
umask 077
mkdir "$RESPALDO"
cp -a /etc/my.cnf /etc/my.cnf.d "$RESPALDO/"
cp -a "$REPOFILE" "$RESPALDO/repositorio-mariadb.repo"
mariadb-dump "${DBOPTS[@]}" --single-transaction --routines --triggers --events "$DB" > "$RESPALDO/base-moodle.sql"
ls -lh "$RESPALDO/base-moodle.sql"
tail -n 5 "$RESPALDO/base-moodle.sql"
```

Comprobar estado de salida de cada orden. La existencia y cierre del SQL no
demuestran recuperabilidad: ensayar su restauracion. `--single-transaction`
necesita tablas transaccionales y ausencia de DDL concurrente para esa garantia;
no cubre por si solo usuarios, grants ni otras bases. Respaldar el conjunto del
servidor mediante el plan de recuperacion; completar la copia en frio del paso 3.

Revisar el archivo de repositorio antes de editarlo. Si contiene exactamente
la URL de rama del caso documentado, el cambio acotado es:

```bash
grep -E '^(baseurl|gpgkey|gpgcheck|sslverify|enabled)' "$REPOFILE"
sed -i 's|rpm.mariadb.org/10.11/|rpm.mariadb.org/11.4/|' "$REPOFILE"
grep '^baseurl' "$REPOFILE"
dnf --disablerepo='*' --enablerepo="$REPOID" list --available MariaDB-server
```

Si el archivo tiene otra estructura, adaptar con la documentacion del proveedor;
no repetir una sustitucion que no coincide. Mantener validacion de firmas GPG y
certificados HTTPS; no copiar excepciones TLS del entorno de origen. Exigir que
el RPM elegido este disponible antes de retirar el servidor anterior. Conservar
tambien instaladores y dependencias necesarios para una recuperacion.

## 3. Detener escritores y copiar datos en frio

Activar mantenimiento, pausar planificadores y esperar a que terminen tareas y
otros clientes que escriban en esta base. Guardar su estado para reponerlo.

```bash
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/maintenance.php" --enable
systemctl stop "$HTTPD_SERVICE"
systemctl stop "$DBSERVICE"
systemctl is-active "$HTTPD_SERVICE" "$DBSERVICE"
```

**Control:** ambos `inactive`, sin otros escritores ni instancias de la base.
Definir el datadir que devolvio la consulta, no el que sugiera un ejemplo:

```bash
DATADIR='<DATADIR_EFECTIVO_SIN_BARRA_FINAL>'
df -h "$DATADIR" "$RESPALDO"
cp -a "$DATADIR" "$RESPALDO/datadir"
diff -qr "$DATADIR" "$RESPALDO/datadir"
```

No copiar encima de una copia previa. Verificar espacio y estado de salida.
`diff` sin diferencias verifica contenido, no permisos ni una restauracion.
Incluir tablespaces externos, claves de cifrado y todos los recursos que no
esten dentro del datadir. Mantener a salvo la copia original sin arrancarla
con el motor nuevo. Si estos requisitos no se pueden garantizar, detenerse.

## 4. Sustitucion controlada de RPM

El salto directo `dnf upgrade 'MariaDB-*'` fallo en PREIN en el ensayo y dejo
cliente y bibliotecas en 11.4 con servidor 10.11. No repetirlo como metodo de
cambio de rama. Inspeccionar la transaccion antes de cada confirmacion:

```bash
dnf remove --noautoremove --assumeno MariaDB-server
```

**Control:** se retira unicamente el servidor previsto. Si aparecen eliminaciones
adicionales, detenerse. `--noautoremove` evita limpieza automatica, pero no
garantiza que DNF nunca retire dependientes; leer siempre el resumen.

Con respaldo comprobado, servicio detenido y candidato disponible:

```bash
dnf remove --noautoremove MariaDB-server
dnf install --assumeno "$MARIADB_SERVER_RPM"
dnf install "$MARIADB_SERVER_RPM"
rpm -qa 'MariaDB-*' | sort
```

Confirmar solo transacciones revisadas, sin `--nodeps`, `--allowerasing` ni
automatizacion de respuestas. Verificar origen y coherencia de servidor,
cliente, common, shared y paquetes compat. La simulacion con `--assumeno`
termina abortada deliberadamente; no instala nada.

Si ya existe un estado parcial por un fallo PREIN, empezar por inventariarlo.
El ensayo resolvio ese estado retirando solo el servidor viejo y colocando el
nuevo con este flujo. No suponer que el estado parcial ocurre en toda migracion.
No inicializar otro datadir ni borrar archivos para resolver el reemplazo.

## 5. Arrancar, actualizar tablas y verificar

Revisar opciones retiradas y archivos `.rpmnew` o `.rpmsave` antes del arranque.
Mantener la web y tareas cerradas durante la actualizacion:

```bash
systemctl start "$DBSERVICE"
systemctl is-active "$DBSERVICE"
mariadb-upgrade "${DBOPTS[@]}"
systemctl restart "$DBSERVICE"
systemctl is-active "$DBSERVICE"
mariadb "${DBOPTS[@]}" -e 'SELECT @@version, @@datadir;'
```

**Control:** servicio `active`, actualizador completo sin errores y version del
servidor en la rama elegida. Si falla, conservar salida privada, consultar el
journal del servicio y no reabrir Moodle. No usar inicio sin controles de acceso
como atajo de diagnostico.

## 6. Validar Moodle antes de pasar a PHP

Con la base operativa, desactivar mantenimiento mediante el PHP que funcionaba
antes de esta etapa e iniciar la web:

```bash
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/maintenance.php" --disable
systemctl start "$HTTPD_SERVICE"
systemctl is-active "$HTTPD_SERVICE"
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/cron.php"
```

Abrir Moodle y un curso existente. Comprobar cron y tareas fallidas; reponer
planificadores solo cuando corresponda. No avanzar a PHP con un fallo de base
sin resolver. En copias de ensayo, aislar envios e integraciones externas.

## 7. Recuperacion

Antes de migrar, disponer de snapshot o respaldo consistente de la plataforma,
configuracion y versiones RPM anteriores. Ante fallo irreversible, aislar el
servicio y restaurar ese conjunto mediante el plan probado. No arrancar MariaDB
10.11 sobre datos ya modificados por 11.4 ni considerar una reinstalacion RPM
como downgrade de los datos. El procedimiento de restauracion debe contemplar
propiedad, SELinux, claves y archivos externos.

## Resultado de referencia y alcance

Se completo la copia en frio, retirada controlada del servidor 10.11.15,
instalacion del servidor 11.4.13, `mariadb-upgrade` con cierre `OK`, reinicio,
consulta efectiva de version, acceso a curso y cron correcto. No se ensayo la
restauracion de los respaldos. Ver la [matriz](estado-y-hallazgos.md).
