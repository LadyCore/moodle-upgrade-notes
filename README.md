# Moodle 4.5.8 a 5.3: notas de laboratorio

Documentacion en preparacion de un ensayo de actualizacion. No es una receta
universal ni un procedimiento aprobado para produccion.

## Alcance

- Moodle 4.5.8 a Moodle 5.3, Build 20261005.
- Entorno de referencia: CentOS Stream 9 Minimal, Apache, PHP 8.3 FPM de Remi y MariaDB 11.4.
- Conservacion de cursos y archivos existentes.
- Revision de compatibilidad de adicionales antes de la validacion funcional final.
- Temas principales: Boost y Adaptable. Alcance ampliado a Moove y Trema como
  alternativas; sus actualizaciones y pruebas se registran por separado.

La actualizacion de Moodle 5.3 descrita aqui se ejecuto en laboratorio, no en
produccion. Las operaciones previas sobre PHP y MariaDB tienen registros
independientes y no deben confundirse con el despliegue de Moodle 5.3.

## Contenido

- [Estado y hallazgos](docs/estado-y-hallazgos.md).
- [Politica de privacidad y publicacion](docs/privacidad.md).
- Procedimiento reproducible: pendiente de extraccion y revision del historial.
- Matriz completa de compatibilidad: pendiente de consolidacion.

## Criterios

- Separar propuestas, ejecuciones confirmadas y pruebas funcionales.
- Documentar el entorno y las versiones exactas de cada ensayo.
- No confundir una migracion de base terminada con una plataforma validada.
- Marcar candidatas RC y soporte no confirmado sin presentarlos como estables.
- Conservar respaldos y definir recuperacion antes de aplicar cambios.
- Revisar procedimientos antes de reproducirlos en produccion.

## Privacidad

Esta guia describe un entorno tecnico reproducible, no identifica las plataformas
de origen ni a sus responsables. No publicar usuarios reales, instituciones,
dominios, IP, nombres de servidores, cursos o plataformas, ni rutas personales.

No incorporar config.php, credenciales, bases de datos, moodledata, archivos de
cursos, datos de participantes ni capturas originales. No copiar la bitacora
operativa completa: este proyecto contiene una sintesis revisada.

El destino acordado es un repositorio publico. No se requiere ocultar la identidad
de quien lo publica; si proteger la informacion de las plataformas de origen y
evitar cualquier vinculacion institucional en documentos, ejemplos y metadatos.
Revisar tambien nombres y correos de autoria para excluir datos institucionales.

La documentacion se publica en elaboracion. Los procedimientos y hallazgos se
ampliaran a medida que se confirmen ejecuciones y resultados.
