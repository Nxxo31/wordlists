# wordlists

## Estado
Activo — wordlists para auditorías de seguridad.

## Objetivo Principal
Crear un sistema de gestión, curación y actualización de diccionarios (wordlists) para ataques de fuerza bruta, con funcionalidades de deduplicación, normalización, filtrado por longitud/charset y generación de reglas para hashcat/john.

## Stack Tecnológico
- **Lenguaje:** Python 3.11
- **Dependencias:** tqdm, tqdm-rich, pandas (opcional para análisis). Verificación: LSP (pyright) + code review
- **Almacenamiento:** Sistema de archivos estructurado por categorías (rockyou, wifi, ssh, formularios, etc.)
- **Procesamiento:** Lectura/escritura streaming para archivos gigantes (>10GB)
- **Normalización:** Conversión a UTF‑8, eliminación de líneas vacías, trim, filtrado por charset ASCII/Unicode
- **Deduplicación:** Sort‑uniq externo o estructuras set en memoria (para archivos medianos) o external sort para gigantes
- **Exportación:** Generación de reglas de transformación (leetscase, toggle case, append numbers) para hashcat
- **Interfaz:** CLI sencilla con subcomandos (download, normalize, dedup, filter, stats, gen-rules)
- **Distribución:** Paquete pip opcional o script auto‑contenido

## Objetivos Secundarios
1. Descarga automática de wordlists de fuentes confiables (SecLists, weakpass, etc.)
2. Normalización a UTF‑8 NFKC, eliminación de BOM, stripping de espacios en blanco.
3. Deduplicación eficiente usando sort -u para archivos grandes o set en memoria para medianos.
4. Filtrado por longitud mínima/máxima y por conjunto de caracteres (ej. solo alfanumérico).
5. Generación de estadísticas (longitud media, distribución de caracteres, frecuencia).
6. Creación de reglas de transformación para hashcat basadas en análisis de frecuencia.
7. Soporte para actualizaciones incrementales (solo procesar archivos nuevos o modificados).
8. Modo de prueba (dry-run) para previsualizar cambios sin escribir archivos.
9. Integración opcional con bases de datos dehasheadas (como haveibeenpwned) para validar calidad.
10. Generación de reportes en Markdown/JSON con resumen de operaciones.

## Arquitectura (Resumen)
```
wordlists/
├── raw/                 # Wordlists descargadas tal cual
├── normalized/          # Después de normalización (UTF-8, sin BOM, trimmed)
├── deduped/             # Después de deduplicación
├── filtered/            # Después de aplicación de filtros (longitud, charset)
├── stats/               # Archivos de estadísticas (JSON/Markdown)
├── rules/               # Reglas generadas para hashcat
├── logs/                # Logs de ejecución
├── config/
│   └── config.yml       # Categorías, fuentes de descarga, rutas, opciones
├── scripts/
│   ├── download.py      # Obtiene wordlists de fuentes configuradas
│   ├── normalize.py     # UTF-8 NFKC, strip BOM, trim
│   ├── dedup.py         # Deduplicación (sort -u o set)
│   ├── filter.py        # Longitud, charset, regex
│   ├── stats.py         # Genera estadísticas
│   └── gen_rules.py     # Crea reglas hashcat a partir de análisis
└── cli.py               # Entrypoint con subcomandos (click o argparse)
```

## Flujo de Trabajo Típico
1. **Descarga**: `wordlists download` → pone las listas crudas en `raw/`
2. **Normalización**: `wordlists normalize` → lee de `raw/`, escribe en `normalized/`
3. **Deduplicación**: `wordlists dedup` → lee de `normalized/`, escribe en `deduped/`
4. **Filtrado**: `wordlists filter --min-len 8 --max-len 63 --charset alnum` → produce `filtered/`
5. **Estadísticas**: `wordstats stats` → genera JSON/Markdown en `stats/`
6. **Reglas**: `wordlists gen-rules` → analiza `filtered/` y produce reglas en `rules/`
7. **Uso**: Las wordlists finales en `filtered/` pueden usarse directamente con hashcat/john, junto con las reglas generadas.

## Backlog inicial (Tareas)
| ID   | Tarea                                                                 | Prioridad | Dependencias |
|------|-----------------------------------------------------------------------|-----------|--------------|
| WL-T1| Definir taxonomía de categorías (ej. rockyo, wifi-defaults, routers, phishing, common-passwords) | Alta      | Ninguna |
| WL-T2| Crear script `download` que obtenga wordlists de fuentes confiables (SecLists, weakpass, etc.) y las coloque en `./raw/` | Alta      | WL-T1 |
| WL-T3| Implementar `normalize` (a UTF‑8 NFKC, strip BOM, eliminar duplicados dentro de cada archivo) | Alta      | WL-T2 |
| WL-T4| Implementar `dedup` usando sort -u externo para archivos >5GB, o set en memoria para <5GB | Alta      | WL-T3 |
| WL-T5| Añadir filtro por longitud mínima/máxima y por charset (ej. solo alfanumérico) | Media     | WL-T4 |
| WL-T6| Generar archivo de reglas para hashcat basado en análisis de frecuencia de la wordlist procesada | Media     | WL-T5 |
| WL-T7| Añadir generación de estadísticas (longitud media, distribución de caracteres, top 10 passwords) | Baja      | WL-T5 |
| WL-T8| Implementar modo dry-run para preview de cambios sin escribir archivos | Baja      | WL-T4 |
| WL-T9| Soportar actualizaciones incrementales (basado en hash o timestamp de archivos) | Baja      | WL-T8 |
| WL-T10| Crear documentación de uso con ejemplos y mejores prácticas | Baja      | WL-T9 |

## Limitaciones Conocidas
- **Memoria**: La deduplicación en memoria para archivos >5GB puede consumir mucha RAM; se recomienda usar sort -u externo.
- **Velocidad**: La normalización y filtrado de archivos muy grandes (>10GB) puede ser lenta; se recomienda ejecutar en SSD y usar pipelines de Unix.
- **Codificación**: Asumimos que las wordlists de entrada son mayormente son ASCII o UTF-8; otras codificaciones pueden requerir pasos adicionales.
- **Licencias**: Algunas wordlists pueden tener restricciones de uso; el script de descarga debe respetar los términos de servicio y licencias de las fuentes.
- **Seguridad**: Las wordlists pueden contener contenido sensible; se debe manejar con cuidado y no compartir públicamente sin autorización.

## Seguridad y Ética
- Este herramienta está destinada únicamente para uso en pruebas de penetración autorizadas y auditorías de seguridad con permiso explícito.
- El usuario es responsable de asegurarse de que tiene derecho a usar las wordlists y de cumplir con las leyes locales y nacionales.
- No se deben utilizar diccionarios o técnicas que infrinjan derechos de terceros o que tengan como fin actividades ilícitas.
- Se recomienda almacenar las wordlists en ubicaciones seguras y con permisos restringidos.

## Referencias
- SecLists: https://github.com/danielmiessler/SecLists
- Weakpass: https://weakpass.com/
- Hashcat rules: https://hashcat.net/wiki/doku.php?id=rule_based_attack
- John the Ripper wordlist rules: https://openwall.com/john/doc/MANUAL.shtml#_wordlist_modes
- NIST SP 800-63B: https://pages.nist.gov/800-63-3/sp800-63b.html (para políticas de contraseñas)
- RFC 7914 (scrypt): https://tools.ietf.org/html/rfc7914 (para entender la necesidad de styrong KDF)