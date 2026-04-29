# Arquitectura de Kanvas

## Descripción General

**KANVAS** es una herramienta de gestión de casos de respuesta a incidentes (IR) construida en Python con interfaz desktop PySide6 (Qt). Proporciona un espacio de trabajo unificado para investigadores que trabajan con hojas de cálculo SOD (Spreadsheet of Doom) o similares, permitiendo completar flujos de trabajo clave sin cambiar entre múltiples aplicaciones.

---

## Estructura del Proyecto

```
kanvas/
├── kanvas.py                          # Archivo principal de la aplicación (2225 líneas)
├── README.md                          # Documentación principal
├── requirements.txt                   # Dependencias de Python
├── sod.xlsx                           # Plantilla de hoja de cálculo SOD
├── mobile_chain_custody_spec.json     # Especificación de cadena de custodia móvil
├── .gitignore                         # Configuración de Git
├── LICENSE                            # Licencia del proyecto
│
├── images/                            # Recursos de iconos e imágenes
│   └── logo.png                       # Logotipo de la aplicación
│
├── assets/                            # Recursos multimedia (GIFs demostrativos)
│   └── kanvas_demo.gif
│
├── markdown_files/                    # Archivos Markdown para editor integrado
│   └── Investigating Palo Alto Networks Firewall Devices.md
│
└── helper/                            # Módulo principal con toda la lógica auxiliar
    ├── __init__.py
    ├── config.py                      # Constantes centrales (nombres de hojas, columnas)
    ├── styles.py                      # Estilos Qt y constantes de color (25157 líneas)
    ├── database_utils.py              # Utilidades de base de datos SQLite
    ├── windowsui.py                   # Interfaz de usuario principal (Ui_KanvasMainWindow)
    ├── search_bar.py                  # Widget de barra de búsqueda
    │
    ├── api_config.py                  # Gestión de claves API (VT, Shodan, OTX, etc.)
    ├── api.yaml                       # Almacenamiento de claves API
    ├── download_links.yaml            # Enlaces para actualizaciones
    ├── download_updates.py            # Lógica de descarga de actualizaciones (41530 líneas)
    ├── system_types.yaml              # Definición de tipos de sistema por defecto
    │
    ├── system_type.py                 # Gestión de tipos de sistema y evidencia
    ├── bookmarks.py                   # Gestor de marcadores y knowledge base
    ├── stix.py                        # Conversión a STIX 2.1 JSON bundles
    ├── defang.py                      # Utilidades para desactivar indicadores (IPs, URLs)
    ├── markdown_editor.py             # Editor y visor Markdown con preview en vivo
    │
    ├── lookups/                       # Módulo de consultas de inteligencia de amenazas
    │   ├── __init__.py
    │   ├── lookup_ip.py               # Consulta de IPs (TOR, IP-API, Shodan, VirusTotal)
    │   ├── lookup_domain.py           # Consulta de dominios (WHOIS, VirusTotal)
    │   ├── lookup_file.py             # Consulta de hashes de archivos (VirusTotal)
    │   ├── lookup_cve.py              # Consulta de vulnerabilidades CVE
    │   ├── lookup_email.py            # Consulta de correos electrónicos (HIBP)
    │   ├── lookup_entraid.py          # Consulta de AppIDs de Microsoft Entra ID
    │   ├── lookup_ransomware.py       # Verificación de víctimas de ransomware
    │   └── ...
    │
    ├── resources/                     # Bases de conocimiento integradas
    │   ├── __init__.py
    │   ├── lolbas.py                  # Living Off the Land Binaries and Scripts
    │   ├── loldrivers.py              # Controladores maliciosos conocidos
    │   ├── lolesxi.py                 # Técnicas de evasión
    │   ├── hijacklibs.py              # DLL Hijacking techniques
    │   ├── artifacts.py               # Artefactos forenses
    │   └── windows_sid.py             # Referencia de SIDs de Windows
    │
    ├── reporting/                     # Motor de generación de reportes
    │   ├── __init__.py
    │   ├── report_builder.py          # UI del constructor de reportes
    │   ├── report_engine.py           # Motor principal de generación
    │   ├── html_exporter.py           # Exportador a HTML autocontenido
    │   └── visualization_generator.py # Generador de visualizaciones para reportes
    │
    ├── mapping_attack.py              # Mapeo MITRE ATT&CK
    ├── mapping_defend.py              # Mapeo MITRE D3FEND
    ├── mapping_veris.py               # Reporte VERIS (Vocabulary for Event Recording)
    │
    ├── viz_network.py                 # Visualización de red lateral (NetworkX + Matplotlib)
    ├── viz_timeline.py                # Línea de tiempo del incidente
    ├── mitre_attack_flow.py           # Constructor de flujos MITRE ATT&CK
    │
    └── resources_data.py              # Datos de recursos (MS Portals, Event IDs)
```

---

## Componentes Principales

### 1. Núcleo de la Aplicación (`kanvas.py`)

**Clase Principal: `MainApp`**

La aplicación sigue un patrón de diseño orientado a objetos con las siguientes responsabilidades:

#### Inicialización (`__init__`)
- Configura `QApplication` con logging a `kanvas.log`
- Carga el logotipo desde `images/logo.png`
- Muestra pantalla de carga (`QSplashScreen`)
- Inicializa la base de datos SQLite (`kanvas.db`)
- Crea instancias de gestores: `SystemTypeManager`, `EvidenceTypeManager`, `ReportEngine`
- Conecta elementos de la UI mediante `connect_ui_elements()`

#### Gestión de Archivos Excel
- **`acquire_file_lock()`**: Implementa bloqueo de archivos con `filelock` para soporte multi-usuario
- **`release_file_lock()`**: Libera el bloqueo al cerrar
- **`load_data_into_treeview()`**: Carga hojas del workbook en un modelo de árbol
- **Modo solo lectura**: Detecta conflictos y permite apertura en modo lectura

#### Interfaz de Usuario
- **`load_ui()`**: Carga la interfaz desde `helper.windowsui.Ui_KanvasMainWindow`
- **`CustomTreeItemDelegate`**: Delegate personalizado para renderizado de items del árbol
  - Resaltado de filas alternas
  - Detección de palabras clave importantes (critical, high, alert, error)
  - Tooltips para texto truncado

#### Funcionalidades Principales
| Método | Descripción |
|--------|-------------|
| `handle_visualize_network()` | Abre ventana de visualización de red |
| `handle_timeline_window()` | Abre línea de tiempo del incidente |
| `handle_mitre_mapping()` | Muestra mapeo ATT&CK |
| `handle_defend_mapping()` | Muestra mapeo D3FEND |
| `handle_veris_window()` | Abre editor VERIS |
| `handle_report_builder()` | Abre constructor de reportes |
| `handle_stix_export()` | Exporta indicadores a STIX 2.1 |
| `defang()` | Desactiva indicadores en el Excel |

---

### 2. Módulo Helper (`helper/`)

#### 2.1 Configuración Central (`config.py`)

Define constantes globales para mantener consistencia:

```python
# Hojas de Excel
SHEET_TIMELINE = "Timeline"
SHEET_SYSTEMS = "Systems"
SHEET_INDICATORS = "Indicators"
SHEET_ACCOUNTS = "Accounts"
SHEET_EVIDENCE_TRACKER = "Evidence Tracker"
SHEET_VERIS = "VERIS"

# Columnas de Timeline
COL_TIMESTAMP = "Timestamp_UTC_0"
COL_ACTIVITY = "Activity"
COL_MITRE_TACTIC = "MITRE Tactic"
COL_MITRE_TECHNIQUE = "MITRE Techniques"
COL_VISUALIZE = "Visualize"
COL_EVENT_SYSTEM = "Event System"
COL_REMOTE_SYSTEM = "Remote System"

# Columnas de Systems
COL_HOSTNAME = "HostName"
COL_IP_ADDRESS = "IPAddress"
COL_SYSTEM_TYPE = "SystemType"
```

#### 2.2 Estilos (`styles.py`)

Centraliza todo el estilizado Qt:

- **Colores**: `COLOR_HIGHLIGHT_BG`, `COLOR_IMPORTANT_TEXT`, etc.
- **Stylesheets**: `MAIN_WINDOW_STYLE`, `BUTTON_STYLE_BASE`, `TREE_VIEW_STYLE`
- **Constantes de UI**: Tamaños de fuente, márgenes, espaciados

#### 2.3 Base de Datos (`database_utils.py`)

Gestiona tablas SQLite:

| Tabla | Propósito |
|-------|-----------|
| `tor_list` | Lista de nodos TOR para lookup de IPs |
| `EvidenceType` | Tipos de evidencia personalizables |
| `bookmarks` | Marcadores organizados por categoría |
| `system_types` | Tipos de sistema con iconos |
| `entra_appid` | AppIDs de Microsoft Entra ID |
| `cisa_ran_exploit` | Exploits conocidos de CISA |
| `ms_portals` | Portales de Microsoft |
| `defend` | Base de datos MITRE D3FEND |

---

### 3. Sistema de Visualización

#### 3.1 Visualización de Red (`viz_network.py`)

**Tecnologías**: NetworkX + Matplotlib + PySide6

**Componentes**:
- **`IconManager`**: Gestiona iconos de sistemas (desde DB o genera círculos de color)
- **`SystemTypeLoader`**: Carga tipos de sistema desde la hoja `Systems`
- **`NetworkGraphBuilder`**: Construye grafo dirigido desde la hoja `Timeline`

**Flujo**:
1. Lee columnas: `Event System`, `Remote System`, `<->`, `Visualize`
2. Filtra entradas con `Visualize = "yes"`
3. Construye grafo usando `networkx.DiGraph()`
4. Asigna iconos según `SystemType`
5. Renderiza con Matplotlib en canvas Qt

#### 3.2 Línea de Tiempo (`viz_timeline.py`)

**Tecnología**: Renderizado custom con QPainter (Qt)

**Características**:
- Orden cronológico basado en `Timestamp_UTC_0`
- Agrupación por tácticas MITRE
- Colores por táctica (configurados en `mapping_attack.py`)
- Exportación a imagen (PNG) y CSV

#### 3.3 MITRE Flow Builder (`mitre_attack_flow.py`)

**Tecnología**: QtWebEngine (Chromium embebido)

**Funcionalidad**:
- Interfaz web-based para construir flujos de ataque
- Integración con MITRE ATT&CK Navigator
- Soporte multi-plataforma con configuraciones específicas:
  - **Windows**: Requiere Visual C++ Redistributable
  - **Linux**: Requiere librerías QtWebEngine
  - **macOS**: Configuración específica de sandbox

---

### 4. Sistema de Mapeo de Frameworks

#### 4.1 MITRE ATT&CK (`mapping_attack.py`)

**Proceso**:
1. Extrae tácticas y técnicas de la hoja `Timeline`
2. Cuenta ocurrencias por táctica
3. Muestra lista coloreada (12 tácticas con colores específicos)

**Colores por Táctica**:
```python
TACTIC_COLORS = {
    "Initial Access": "#e57373",
    "Execution": "#FFB74D",
    "Persistence": "#FFF176",
    "Privilege Escalation": "#AED581",
    "Defense Evasion": "#4FC3F7",
    "Credential Access": "#FF8A65",
    "Discovery": "#9575CD",
    "Lateral Movement": "#4DB6AC",
    "Collection": "#F06292",
    "Command and Control": "#7986CB",
    "Exfiltration": "#A1887F",
    "Impact": "#90A4AE",
}
```

#### 4.2 MITRE D3FEND (`mapping_defend.py`)

**Funcionamiento**:
- Consulta base de datos SQLite `defend`
- Mapea técnicas ATT&CK ofensivas a contramedidas D3FEND
- Muestra tabla con: artefacto ofensivo, táctica D3FEND, técnica D3FEND, artefacto defensivo

#### 4.3 VERIS (`mapping_veris.py`)

**Propósito**: Vocabulario estandarizado para reporte de incidentes

**Categorías**:
- Timeline
- Victim
- Actors
- Action
- Asset
- Attribute
- Plus

**Exportación**: CSV compatible con sistemas gubernamentales y Verizon DBIR

---

### 5. Módulo de Threat Intelligence Lookups (`helper/lookups/`)

Todos los módulos siguen un patrón común:
- Ventana modal con campo de entrada
- Botones de acción rápida a herramientas externas
- Resultados formateados con resaltado sintáctico

#### 5.1 Lookup de IPs (`lookup_ip.py`)

**Fuentes de Datos**:
1. **TOR Database** (SQLite local): Verifica si es nodo TOR
2. **IP-API**: Geolocalización, ISP, ASN
3. **Shodan**: Puertos abiertos, servicios, vulnerabilidades
4. **VirusTotal**: Puntuación de detección, comentarios

**Enlaces Rápidos**:
- VirusTotal, OTX, DShield, Talos, Spamhaus, Criminal-IP, AbuseIPDB, Pulsedive

#### 5.2 Lookup de Dominios (`lookup_domain.py`)

**Fuentes**:
- **WHOIS** (python-whois): Fecha creación, registrador, nameservers
- **VirusTotal**: Detecciones, categorías, IPs asociadas

**Procesamiento**:
- Extrae dominio desde URL completa
- Calcula días desde creación (resaltado si < 30 días)

#### 5.3 Lookup de Hashes (`lookup_file.py`)

**Soporta**: MD5, SHA-1, SHA-256

**Fuente Principal**: VirusTotal API v3

**Enlaces**: Joe Sandbox, Any.Run, Hybrid Analysis, ThreatFox

#### 5.4 Otros Lookups

| Módulo | Propósito | APIs Utilizadas |
|--------|-----------|-----------------|
| `lookup_cve.py` | Vulnerabilidades CVE | CISA KEV, NVD |
| `lookup_email.py` | Brechas de datos | Have I Been Pwned |
| `lookup_entraid.py` | AppIDs maliciosos | Microsoft Graph |
| `lookup_ransomware.py` | Víctimas públicas | ransomware.live |

---

### 6. Sistema de Reportes (`helper/reporting/`)

#### 6.1 Arquitectura del Report Engine

```
┌─────────────────────────────────────────────────────────┐
│                   Report Builder UI                      │
│              (Selección de secciones)                    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                    Report Engine                         │
│  ┌──────────────────┐  ┌──────────────────────────────┐ │
│  │  MarkdownExporter│  │        HTMLExporter          │ │
│  │  - Tablas MD     │  │  - CSS embebido              │ │
│  │  - Truncamiento  │  │  - Imágenes Base64           │ │
│  └──────────────────┘  │  - Defang automático         │ │
│                        │  - Gráficos insertados       │ │
│                        └──────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│            Visualization Generator                       │
│         (Timeline / Network como PNG Base64)            │
└─────────────────────────────────────────────────────────┘
```

#### 6.2 HTML Exporter (`html_exporter.py`)

**Características Clave**:
- **Autocontenido**: Todo en un solo archivo HTML (CSS, imágenes Base64)
- **Defang automático**: IPs, URLs, dominios se desactivan
- **Secciones incluidas**:
  - Línea de tiempo del incidente
  - Movimiento lateral (grafo)
  - Mapeo MITRE ATT&CK
  - Sistemas comprometidos
  - Cuentas comprometidas
  - IOCs
  - Evidence Tracker
  - VERIS
  - Resumen ejecutivo (Markdown)
  - Recomendaciones (Markdown)

#### 6.3 Report Builder UI (`report_builder.py`)

**Flujo de Usuario**:
1. Selecciona secciones a incluir
2. Configura encabezado (título, autor, colores)
3. Selecciona archivos Markdown para resumen/recomendaciones
4. Opción de filtrar por headings específicos
5. Genera reporte HTML o Markdown

---

### 7. Knowledge Base Integrada

#### 7.1 Recursos (`helper/resources/`)

| Recurso | Descripción | Fuente |
|---------|-------------|--------|
| LOLBAS | Binarios Living-off-the-land | lolbas-project.github.io |
| LOLDrivers | Controladores maliciosos | loldrivers.io |
| LOLESXI | Técnicas de evasión | Investigación interna |
| HijackLibs | DLL Hijacking | hijacklibs.net |
| Artifacts | Artefactos forenses | Comunidad DFIR |
| Windows SID | Security Identifiers | Microsoft Docs |

#### 7.2 Bookmarks (`bookmarks.py`)

**Categorías**:
- Herramientas de seguridad
- Portales de Microsoft (msportals.io)
- Recursos de investigación
- **Personal**: Marcadores personalizados del usuario

**Almacenamiento**: SQLite en `kanvas.db`, tabla `bookmarks`

#### 7.3 Editor Markdown (`markdown_editor.py`)

**Características**:
- Vista dividida: Editor + Preview
- Syntax highlighting (Pygments)
- Soporte para tablas, código fenced
- Gestión de archivos en `markdown_files/`
- Atajos de teclado (Ctrl+S, Ctrl+O, etc.)

---

### 8. Gestión de Tipos de Sistema (`system_type.py`)

**Clases Principales**:

#### `SystemTypeManager`
- Carga tipos desde YAML (`system_types.yaml`) o SQLite
- Cachea iconos para rendimiento
- Proporciona fallback colors si no hay icono

**Tipos por Defecto**:
```yaml
default_system_types:
  - name: "Attacker"
    fallback_color: "#DC3545"
  - name: "Server"
    fallback_color: "#007BFF"
  - name: "Client"
    fallback_color: "#28A745"
  - name: "Gateway"
    fallback_color: "#FFC107"
  - name: "OT"
    fallback_color: "#6F42C1"
```

#### `EvidenceTypeManager`
- CRUD de tipos de evidencia
- Diálogo para añadir nuevos tipos
- Integración con hoja `Evidence Tracker`

#### `IconManager`
- Carga iconos desde `images/`
- Genera iconos circulares de color programáticamente
- Cachea para evitar recarga

---

### 9. Utilidades de Seguridad

#### 9.1 STIX Export (`stix.py`)

**Conversión a STIX 2.1**:
- Detecta tipo de indicador automáticamente
- Genera patrones STIX:
  ```python
  IPAddress → [ipv4-addr:value = '192.168.1.1']
  DomainName → [domain-name:value = 'evil.com']
  SHA256 → [file:hashes.'SHA-256' = 'abc123...']
  URL → [url:value = 'http://evil.com/malware']
  ```
- Crea bundle STIX con metadatos del incidente

#### 9.2 Defang (`defang.py`)

**Transformaciones**:
```python
192.168.1.1     → 192.168.1[.]1
http://evil.com → hxxp://evil.com
evil.com        → evil[.]com
user@evil.com   → user[at]evil.com
```

**Aplicación**:
- Texto plano
- Archivos Excel completos (preservando formato)

---

### 10. Configuración de APIs (`api_config.py`)

**APIs Soportadas**:

| Servicio | Campo YAML | Propósito |
|----------|------------|-----------|
| VirusTotal | `VT_API_KEY` | Reputation de IPs, dominios, hashes |
| Shodan | `SHODEN_API_KEY` | Información de puertos/servicios |
| AlienVault OTX | `OTX_API_KEY` | Threat intelligence colaborativa |
| Have I Been Pwned | `HIBP_API_KEY` | Brechas de emails |
| OpenAI | `openAI_API_KEY` | Futuras integraciones AI |
| Anthropic | `ANTHROPIC_API_KEY` | Futuras integraciones AI |

**Almacenamiento**: `helper/api.yaml`
```yaml
api_keys:
  VT_API_KEY: "tu_api_key_aqui"
  SHODEN_API_KEY: ""
  OTX_API_KEY: ""
```

---

## Flujo de Datos

### Carga de Archivo Excel

```
┌─────────────┐
│   Usuario   │
│  selecciona │
│  archivo    │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────┐
│  MainApp.acquire_file_lock()    │
│  - Verifica lock existente      │
│  - Ofrece modo solo-lectura     │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│  openpyxl.load_workbook()       │
│  - Carga todas las hojas        │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│  load_data_into_treeview()      │
│  - Crea QStandardItemModel      │
│  - Popula con filas del Excel   │
│  - Aplica proxy_model para sort │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│  TreeView muestra datos         │
│  - CustomTreeItemDelegate       │
│  - Resaltado condicional        │
└─────────────────────────────────┘
```

### Generación de Reporte HTML

```
┌─────────────┐
│   Usuario   │
│  inicia     │
│  reporte    │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────┐
│  Report Builder Dialog          │
│  - Selecciona secciones         │
│  - Configura header/footer      │
│  - Selecciona archivos .md      │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│  ReportEngine.generate_report() │
│  - Itera sobre secciones        │
│  - Lee datos de cada hoja       │
└──────┬──────────────────────────┘
       │
       ├─────────────────┬──────────────────┐
       ▼                 ▼                  ▼
┌─────────────┐  ┌──────────────┐  ┌────────────────┐
│HTMLExporter │  │Visualization │  │  Markdown      │
│- Tablas HTML│  │Generator     │  │  Parser        │
│- Defang IOC │  │- Timeline PNG│  │- Summary .md   │
│- CSS inline │  │- Network PNG │  │- Recommend .md │
└──────┬──────┘  └──────┬───────┘  └───────┬────────┘
       │                │                  │
       └────────────────┼──────────────────┘
                        │
                        ▼
              ┌─────────────────┐
              │ Imágenes → Base64│
              │ CSS → <style>    │
              │ MD → HTML        │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  HTML final     │
              │  autocontenido  │
              └─────────────────┘
```

---

## Dependencias Técnicas

### Core Framework
- **PySide6 >= 6.0.0**: Framework Qt para Python
- **shiboken6 >= 6.0.0**: Bindings de C++ para Qt

### Procesamiento de Datos
- **openpyxl >= 3.0.0**: Lectura/escritura de Excel (.xlsx)
- **pandas >= 1.3.0**: Manipulación de DataFrames
- **numpy >= 1.21.0**: Operaciones numéricas

### Visualización
- **networkx >= 2.6.0**: Creación y análisis de grafos
- **matplotlib >= 3.5.0**: Renderizado de gráficos

### Redes y APIs
- **requests >= 2.25.0**: Cliente HTTP para APIs
- **python-whois >= 0.7.0**: Consultas WHOIS
- **shodan >= 1.25.0**: SDK de Shodan

### IA y ML (Futuro)
- **openai >= 1.0.0**: Integración con GPT
- **anthropic >= 0.18.0**: Integración con Claude

### Utilidades
- **Markdown >= 3.3.0**: Parseo de Markdown
- **PyYAML >= 6.0**: Lectura de configuración YAML
- **pygments >= 2.10.0**: Syntax highlighting
- **filelock >= 3.0.0**: Bloqueo de archivos multi-usuario

---

## Consideraciones de Diseño

### 1. Multi-Usuario
- **File Locking**: Usa `filelock` para prevenir escrituras concurrentes
- **Modo Solo-Lectura**: Permite visualizar cuando otro usuario edita
- **Archivos en Red**: Soporta rutas UNC/shared drives

### 2. Cross-Platform
- **Windows**: Soporte completo con QtWebEngine
- **macOS**: Ajustes de fuente específicos (SF Pro, SF Mono)
- **Linux**: Configuración de sandbox para QtWebEngine

### 3. Rendimiento
- **Cache de Iconos**: Evita recarga de imágenes
- **Proxy Models**: Sorting/filtering eficiente en TreeView
- **Lazy Loading**: Ventanas secundarias se crean bajo demanda

### 4. Seguridad
- **Defang Automático**: Previene clicks accidentales en IOCs
- **API Keys Locales**: Almacenadas en `api.yaml` (no en código)
- **Logging**: Todas las acciones se registran en `kanvas.log`

---

## Extensiones Futuras

1. **Integración AI**: Uso de OpenAI/Anthropic para análisis automático
2. **Export SIEM**: Formatos CEF, LEEF para ingestión directa
3. **Colaboración en Tiempo Real**: WebSockets para edición simultánea
4. **Plugins**: Sistema de extensiones para funcionalidad personalizada

---

## Referencias

- **Repositorio**: https://github.com/WithSecureLabs/Kanvas
- **SOD Template**: Incluido como `sod.xlsx`
- **Documentación**: `README.md` en raíz del proyecto
- **Licencia**: Ver archivo `LICENSE`

---

*Documento generado basado en análisis estático del código fuente de Kanvas.*
