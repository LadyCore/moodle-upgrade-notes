# Politica de privacidad y publicacion

## Objetivo

Publicar conocimiento tecnico que pueda reproducirse en entornos equivalentes,
sin identificar las plataformas de origen, su pertenencia o sus responsables.
Esta regla se aplica desde el primer commit, aunque el repositorio sea privado.

## Excluir

- Usuarios personales o administrativos reales, correos y nombres de personas.
- Instituciones, organizaciones y nombres internos de plataformas o cursos.
- IP publicas o privadas, dominios, URL de instalaciones, hostnames y topologia real.
- Nombres de equipos y maquinas virtuales, rutas personales y marcas institucionales.
- Credenciales, tokens, certificados privados, claves, sesiones y cadenas de conexion.
- Bases de datos, respaldos de cursos, moodledata y configuraciones originales.
- Capturas, nombres de archivos y metadatos que permitan identificar el origen.
- Enlaces a bitacoras privadas, rutas locales del titular o anexos operativos.

Los enlaces a documentacion publica y a los repositorios de autores de plugins
pueden conservarse como atribucion tecnica. No son identificadores del titular
de las plataformas ensayadas.

## Conservar

- Distribucion y version de Linux: CentOS Stream 9, instalacion Minimal en el ensayo.
- Versiones comprobadas de Moodle, PHP, MariaDB, Apache y adicionales.
- Repositorios tecnicos de paquetes y variantes necesarias para reproducir el caso.
- Requisitos, errores depurados, causa comprobada, solucion y limites del ensayo.
- Diferencia entre ensayo y despliegue, sin identificar a quien pertenece cada instancia.
- Estado estable o RC y pruebas pendientes, sin presentar propuestas como ejecuciones.

No generalizar el resultado a otras distribuciones sin explicar las diferencias.
No inventar revisiones de sistema, versiones o caracteristicas no comprobadas.

## Ejemplos y procedimientos

Usar marcadores como <HOST_MOODLE>, <USUARIO_SERVICIO>, <RUTA_MOODLE>,
<RUTA_DATOS> y <RUTA_RESPALDO> en lugar de valores del entorno real.
Explicar que deben sustituirse antes de ejecutar comandos. Los ejemplos deben
ser genericos desde su redaccion; no pegar primero una configuracion sensible
con la intencion de borrarla en otro commit.

## Revision previa a cada envio

1. Revisar el contenido y los nombres de todos los archivos incluidos.
2. Buscar identificadores de origen, direcciones, correos, credenciales y rutas personales.
3. Revisar cambios preparados e historial: borrar un dato de la ultima version no
   lo elimina de commits anteriores.
4. No considerar .gitignore una herramienta de anonimizado ni un control suficiente.
5. Mantener las bitacoras operativas fuera del repositorio.
6. No incorporar capturas originales; preferir ejemplos sinteticos sin metadatos de origen.

## Identidad de publicacion

El destino acordado es un repositorio publico. La identidad personal de quien lo
publica no necesita ocultarse: el alcance de proteccion es la informacion de las
plataformas de origen y su pertenencia institucional.

El propietario del repositorio y los metadatos de autoria de Git son distintos del
contenido de los documentos. Revisar nombres y correos de autoria para no exponer
datos institucionales, y aplicar la misma revision a descripciones, mensajes de
commit, incidencias y adjuntos. No crear cuentas ni organizaciones ni modificar
perfiles o datos de autoria sin autorizacion.

No prometer anonimato absoluto por sustituir nombres o direcciones en documentos.
