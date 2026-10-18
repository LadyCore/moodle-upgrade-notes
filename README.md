# Actualizacion de Moodle 4.5 a 5.3

Procedimiento de administracion para preparar, ejecutar y verificar una
actualizacion de Moodle en CentOS Stream 9 Minimal, con Apache, PHP-FPM de Remi,
MariaDB y SELinux. Incluye cambio de raiz web a `public`, actualizacion de
plugins, comprobaciones funcionales y criterios de recuperacion.

## Ruta de lectura

1. [Procedimiento de actualizacion](docs/procedimiento-actualizacion.md): requisitos,
   parametros, respaldo, sustitucion del codigo y actualizador CLI.
2. [Actualizacion de plugins](docs/actualizacion-plugins.md): seleccion del paquete,
   requisitos, sustitucion por componente y subplugins.
3. [Validacion funcional](docs/validacion.md): pruebas y criterios de aceptacion.
4. [Diagnostico](docs/diagnostico.md): sintomas, comprobaciones y acciones acotadas.
5. [Matriz del ensayo de referencia](docs/estado-y-hallazgos.md): versiones exactas,
   resultados obtenidos y limitaciones conocidas.

## Alcance y uso

La guia presupone experiencia con Linux, Apache y administracion de Moodle.
No instala el sistema operativo ni actualiza PHP o MariaDB: esas dependencias
deben cumplir los requisitos de la version elegida antes de comenzar.

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
