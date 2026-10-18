# Actualizacion de plugins

## Seleccion

Para un componente sobre un core operativo, usar el flujo siguiente. Durante
una migracion del core, preparar candidatos en el arbol nuevo antes del upgrade
general. Consultar catalogo y repositorio del mantenedor y elegir una version
fija o commit, no una rama flotante.

| Metadato | Control |
| --- | --- |
| component | Identidad correcta del plugin |
| version y release | Candidato esperado; no es un downgrade |
| requires | Core satisface el minimo |
| dependencies | Componentes y versiones disponibles |
| supported o catalogo | Incluye la version destino |
| maturity | Estable o condicion experimental identificada |
| Subplugins | Incluidos y compatibles como conjunto |

Un minimo satisfecho no garantiza soporte posterior. Si la version destino
queda fuera del rango declarado, detener la instalacion ordinaria. Cualquier
ensayo fuera de rango debe ser deliberado, aislado y recuperable.

## Descargar e inspeccionar

Sustituir marcadores y emplear destinos nuevos fuera del arbol web:

```bash
URL='<URL_HTTPS_DEL_PAQUETE_FIJO>'
ZIP='<RUTA_ZIP_FUERA_WEB>'
ENSAYO='<DIRECTORIO_NUEVO_FUERA_WEB>'
curl -fL -o "$ZIP" "$URL"
unzip -t "$ZIP"
unzip -l "$ZIP" | head -n 15
mkdir "$ENSAYO"
unzip -q "$ZIP" -d "$ENSAYO"
```

Detenerse ante error, archivo invalido o destino existente. Verificar tambien
sumas o firmas publicadas por el autor; `unzip -t` no acredita autenticidad.
Identificar el directorio raiz extraido y leer metadatos:

```bash
CANDIDATO='<DIRECTORIO_EXTRAIDO_DEL_PLUGIN>'
grep -A 3 'plugin->' "$CANDIDATO/version.php"
```

Leer el archivo completo si no se muestran todas las dependencias. Verificar
el nombre final: por ejemplo, `theme_moove` va en `public/theme/moove`, aunque
el ZIP y el directorio extraido tengan otros nombres.

## Sustitucion

Usar las variables y medidas del [procedimiento principal](procedimiento-actualizacion.md).
Activar mantenimiento, detener trafico y planificadores, esperar tareas y capturar
el punto de recuperacion consistente antes de sustituir codigo.

```bash
DESTINO='<RUTA_ACTUAL_DEL_DIRECTORIO_PLUGIN>'
ANTERIOR='<RUTA_RESPALDO_PLUGIN_FUERA_WEB_NO_EXISTENTE>'
mv -T -i "$DESTINO" "$ANTERIOR"
mv -T -i "$CANDIDATO" "$DESTINO"
chown -R "root:$WEBGROUP" "$DESTINO"
restorecon -RF "$DESTINO"
sudo -u "$WEBUSER" "$PHPCLI" "$MOODLE/admin/cli/upgrade.php"
```

Ejecutar linea por linea y detenerse ante error o solicitud de sobrescritura.
Verificar ubicaciones antes de mover. Si un traslado parcial falla, inspeccionar
origen y destino; no continuar sobre un arbol incompleto. No mezclar versiones.

No hacer escribible todo el directorio de temas para permitir ZIP por navegador.
Un fallo de acceso de escritura no demuestra incompatibilidad del plugin.

**Control:** upgrade correcto del componente y sus subplugins. Purgar caches,
salir de mantenimiento, iniciar Apache, realizar la [prueba funcional](validacion.md)
y reponer planificadores segun el procedimiento principal. Conservar codigo viejo
fuera del arbol web; no basta para revertir cambios de base.

## Certificados

Actualizar `tool_certificate` en `public/admin/tool/certificate` con sus elementos
incluidos; no realizar nueve descargas independientes. En la referencia se
actualizaron border, code, date, digitalsignature, image, program, text, userfield
y userpicture junto al gestor.

Abrir una plantilla existente y su vista previa. Emision real y validacion del
codigo son pruebas separadas. Una imagen de firma no acredita firma criptografica.

## Registro

Anotar referencia fija, versiones, dependencias, respaldo, resultado CLI y pruebas.
La [matriz de referencia](estado-y-hallazgos.md) contiene candidatos historicos;
volver a consultar publicaciones antes de elegir paquetes para otra instalacion.
