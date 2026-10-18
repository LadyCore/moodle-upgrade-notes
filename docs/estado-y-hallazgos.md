# Matriz del ensayo de referencia

Evidencia consolidada hasta el 2026-10-18. Este anexo conserva trazabilidad
tecnica; las instrucciones reutilizables estan en el
[procedimiento](procedimiento-actualizacion.md). Las versiones son referencias
historicas, no una seleccion automatica de paquetes para otra instalacion.

## Entorno

| Elemento | Referencia |
| --- | --- |
| Sistema | CentOS Stream 9 Minimal |
| Web | Apache con PHP-FPM de Remi |
| PHP utilizado | 8.3.35; ejecutable CLI explicitamente seleccionado |
| Base de datos | MariaDB 11.4.13 |
| Origen Moodle | 4.5.8 |
| Destino Moodle | 5.3 Build 20261005, version 2026100500 |

## Componentes

Los nueve elementos de certificados forman parte del paquete del gestor.
`Upgrade correcto` significa que el actualizador termino, no que todas las
funciones del componente hayan sido verificadas.

| Componente | Version ensayada y referencia fija | Soporte declarado revisado | Resultado y limite |
| --- | --- | --- | --- |
| Multitopic | 5.3.0, build 2026100502; tag v5.3.0 | 5.3 | Upgrade correcto; contenido, banner y pestanas recuperados |
| Collapsed Topics | 503.0.2, build 2026100701; tag V503.0.2 | 5.3; RC | Upgrade correcto; prueba en curso representativo pendiente |
| Grid | 500.1.2, build 2025040707; tag V500.1.2 | Solo 5.0 | Upgrade correcto fuera de rango; funcionalidad pendiente |
| Onetopic | 5.1.0(Minitas), build 2026040500.02; commit bf59f12f670d88892112be2205085fa25377a181 | Solo 5.1 | Upgrade correcto fuera de rango; funcionalidad pendiente |
| Tiles | 5.1.0.2, build 2026012570; commit b22538a2fb7bbcbd41d6400358d22ddfc5ad307e | Solo 5.1 | Upgrade correcto fuera de rango; funcionalidad pendiente |
| Adaptable | 502.2.1, build 2026041207; tag V502.2.1 | Solo 5.2 | Upgrade correcto; advertencia de incompatibilidad y paneles superpuestos en 5.3 |
| Moove | 5.3.1, build 2026101200; commit 1e0e60fc3b1fe152a4d6749054ee25dc0a2f32ad | Minimos comprobados; rango no certificado aqui | Portada, curso, foro, libro y espacio personal de mensajes accesibles; alineacion de paneles pendiente |
| Trema | 5.3.0.0, build 2026101200; tag 5.3.0.0 | Minimos comprobados; rango no certificado aqui | Revision basica equivalente a Moove; panel sobre banner y detalles visuales pendientes |
| Gestor de certificados y nueve elementos | 5.0.11, build 2026100600; tag v5.0.11 | 5.0-5.2 | Upgrade correcto fuera de rango; gestion y vista previa PDF satisfactorias |
| Mail | 2.18, build 2026100500 | Incluye 5.3 | Sin reemplazo; entrega no validada |
| eMail Test | 3.1.8, build 2026042001 | Incluye 5.3 | Sin reemplazo; prueba funcional pendiente |
| Zoom | 5.5.1, build 2026082400 | Pendiente de revalidar | Sin reemplazo; revision pendiente |

## Infraestructura

| Comprobacion | Resultado |
| --- | --- |
| Actualizador del core | Finalizo correctamente |
| Raiz web en public | Acceso operativo |
| Entrada por raiz | 404 corregido al establecer DirectoryIndex index.php |
| Composer | Instalacion desde composer.lock; check-platform-reqs --no-dev satisfactorio en CLI |
| Entorno web | Desaparecio Composer installed data not found |
| /core/check/controller/test | Respondio Test controller response antes de declarar routerconfigured |
| Router not configured | Dejo de aparecer tras la declaracion; otras rutas no exhaustivamente verificadas |
| Bloques globales | Navigation y Administration operativos tras habilitacion en Boost y ajuste de alcance; sin duplicados observados |

## Limites funcionales

- Cursos: pestanas operativas, PDF visibles y paginas de foro, libro y
  cuestionario accesibles. No se validaron nuevos intentos, calificaciones,
  finalizacion ni permisos de estudiante. Abrir un recurso URL no comprueba su
  destino externo.
- Mensajeria: apertura y mensajes nuevos en espacio personal con Moove y Trema;
  no demuestra entrega entre usuarios ni notificaciones.
- Certificados: PDF con nombre, fecha, fondo, imagen de firma, logo y QR.
  `coursefullname` aparece literal en la vista previa. Emision real, sustitucion
  del curso, validacion de QR y firma criptografica no comprobadas.
- Formatos: faltan cursos representativos para completar las pruebas.
- Temas: causa no determinada de los desajustes de paneles en Moove y Trema.
  No se aplicaron parches CSS. Adaptable conserva un problema de compatibilidad.

## Observaciones fechadas

La consulta de Adaptable del 2026-10-17 no localizo un paquete publico declarado
compatible con 5.3. V502.2.1 y main correspondian a 5.2. No es el estado actual
garantizado del proyecto. Una mencion de experimentos de modo oscuro no constituye
anuncio de adaptacion, plazo o soporte para 5.3.

El aviso de `tool_certificate/db/subplugins.json` sobre `subplugintypes` obsoletos
estaba presente antes del reemplazo. No aparecio en la salida visible del upgrade
a 5.0.11; no se verificaron todos los contextos de ejecucion.

## Fuentes de paquetes y autoria

- [Multitopic, repositorio del autor](https://github.com/james-cnz/moodle-format_multitopic/releases/tag/v5.3.0).
- [Collapsed Topics, Gareth Barnard](https://github.com/gjbarnard/moodle-format_topcoll/tree/V503.0.2).
- [Grid, Gareth Barnard](https://github.com/gjbarnard/moodle-format_grid/tree/V500.1.2).
- [Adaptable, repositorio de mantenimiento](https://github.com/gjbarnard/moodle-theme_adaptable/tree/V502.2.1).
- [Onetopic, repositorio del autor](https://github.com/davidherney/moodle-format_onetopic).
- [Tiles, repositorio del autor](https://bitbucket.org/dw8/moodle-format_tiles/).
- [Moove, Willian Mano](https://github.com/willianmano/moodle-theme_moove).
- [Trema, Trema Tech](https://github.com/trema-tech/moodle-theme_trema).
- [Gestor de certificados, Moodle Workplace](https://github.com/moodleworkplace/moodle-tool_certificate/tree/v5.0.11).
- [Mail, catalogo de versiones](https://marketplace.moodle.com/plugins/1048/versions).
- [eMail Test, catalogo de versiones](https://marketplace.moodle.com/plugins/3723/versions).
