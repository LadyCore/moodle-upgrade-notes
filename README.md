# Actualizacion de plataforma Moodle: MariaDB, PHP y Moodle

Procedimiento de administracion para preparar, ejecutar y verificar una
actualizacion de la plataforma en CentOS Stream 9 Minimal, con Apache y SELinux.
Incluye MariaDB 10.11 a 11.4, PHP 8.1 a 8.3 mediante Remi en paralelo y Moodle
4.5 a 5.3, cambio de raiz web a `public`, plugins y validacion funcional.

## Ruta de lectura

1. [Actualizacion de MariaDB](docs/actualizacion-mariadb.md): repositorio, respaldo,
   sustitucion controlada del servidor, mariadb-upgrade y verificaciones.
2. [Actualizacion de PHP](docs/actualizacion-php.md): instalacion paralela, ajustes,
   pool FPM, sockets Apache, PHP web y cron.
3. [Actualizacion de Moodle](docs/procedimiento-actualizacion.md): requisitos,
   parametros, respaldo, sustitucion del codigo y actualizador CLI.
4. [Actualizacion de plugins](docs/actualizacion-plugins.md): seleccion del paquete,
   requisitos, sustitucion por componente y subplugins.
5. [Validacion funcional](docs/validacion.md): pruebas y criterios de aceptacion.
6. [Diagnostico](docs/diagnostico.md): sintomas, comprobaciones y acciones acotadas.
7. [Matriz del ensayo de referencia](docs/estado-y-hallazgos.md): versiones exactas,
   resultados obtenidos y limitaciones conocidas.

## Alcance y uso

La guia presupone experiencia con Linux, Apache y administracion de Moodle.
No instala el sistema operativo desde cero. El recorrido completo actualiza
primero MariaDB, despues PHP y finalmente Moodle, comprobando la plataforma
entre etapas. Si MariaDB o PHP ya cumplen los requisitos, verificar su estado
y omitir solo la etapa que no corresponda.

Los comandos contienen marcadores que deben sustituirse. Ejecutar los pasos
por separado y comprobar cada resultado; no pegar el documento como un script.
Los comandos del sistema requieren privilegios administrativos; las utilidades
Moodle se ejecutan como el usuario de servicio web.

El ensayo de referencia partio de Moodle 4.5.8 y utilizo Moodle 5.3 Build
20261005, PHP 8.3.35 y MariaDB 11.4.13. No son una recomendacion de versiones
actuales: revisar requisitos, publicaciones y soporte antes de otra ejecucion.

Las instrucciones generalizadas se distinguen de los resultados del ensayo.
Un upgrade CLI exitoso no garantiza compatibilidad funcional. Los paquetes RC
o fuera de su rango declarado se incluyen como referencia experimental, no
como dependencias recomendadas.

## Recuperacion

Antes de modificar codigo o datos debe existir una recuperacion comprobada de
base de datos, `moodledata`, codigo y configuracion del mismo punto consistente.
Conservar solamente el codigo anterior no permite deshacer una migracion de base.
Una copia independiente permite comparar instalaciones; un snapshot ofrece un
punto de retorno, sujeto a que incluya los discos y servicios necesarios.

## Referencias y privacidad

- [Guia oficial de actualizacion de Moodle 5.3](https://docs.moodle.org/503/en/Upgrading).
- [Politica de privacidad del repositorio](docs/privacidad.md).

El repositorio contiene procedimientos y resultados tecnicos anonimizados.
No debe incluir configuraciones reales, credenciales, respaldos, datos de cursos
o participantes, ni capturas originales de instalaciones.
