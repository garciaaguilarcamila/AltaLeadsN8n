# 🤖 CRM Automático Multiestado con n8n + Google Sheets

> Mini CRM gratuito para pymes — gestión de leads, estados, histórico y dashboard sin pagar HubSpot ni Pipedrive.

![n8n](https://img.shields.io/badge/n8n-automation-orange?style=flat-square)
![Google Sheets](https://img.shields.io/badge/Google%20Sheets-data-green?style=flat-square)
![Apps Script](https://img.shields.io/badge/Apps%20Script-dashboard-blue?style=flat-square)
![Coste](https://img.shields.io/badge/coste-0%20%E2%82%AC-brightgreen?style=flat-square)

---

## 📋 Tabla de contenidos

- [¿Qué es este proyecto?](#qué-es-este-proyecto)
- [Herramientas utilizadas](#herramientas-utilizadas)
- [Estructura del sistema](#estructura-del-sistema)
- [Google Sheets — estructura y acceso](#google-sheets--estructura-y-acceso)
- [Credenciales necesarias en n8n](#credenciales-necesarias-en-n8n)
- [Workflows](#workflows)
  - [Workflow 1 — Alta de Lead](#workflow-1--alta-de-lead)
  - [Workflow 2 — Alimentador Automático](#workflow-2--alimentador-automático)
  - [Workflow 3 — Actualizar Estado](#workflow-3--actualizar-estado)
- [Dashboard](#dashboard)
- [Estados disponibles](#estados-disponibles)
- [Flujo completo de una oportunidad real](#flujo-completo-de-una-oportunidad-real)

---

## ¿Qué es este proyecto?

Un sistema CRM completamente funcional construido con herramientas gratuitas que permite a cualquier empresa:

- ✅ Registrar leads nuevos de forma manual o automática
- ✅ Detectar y evitar duplicados por email y teléfono
- ✅ Gestionar el ciclo de vida de cada oportunidad con estados personalizables
- ✅ Guardar un histórico completo e inmutable de cada cambio de estado
- ✅ Ver un dashboard con métricas comerciales en tiempo real
- ✅ Automatizar la captación diaria de leads desde una hoja de origen

Es especialmente útil para **inmobiliarias, asesorías, academias, clínicas, agencias y negocios locales** que no quieren o no pueden pagar una herramienta CRM mensual.

---

## Herramientas utilizadas

| Herramienta | Uso | Coste |
|---|---|---|
| **n8n** (self-hosted con Docker) | Motor de automatización y workflows | Gratuito |
| **Google Sheets** | Base de datos y dashboard | Gratuito |
| **Google Apps Script** | Detector de cambios + generación del dashboard | Gratuito |
| **Google Cloud Console** | Credenciales OAuth para conectar n8n con Google | Gratuito |

---

## Estructura del sistema

```
┌─────────────────────────────────────────────────────┐
│              GOOGLE SHEETS — CRM Leads               │
│  ┌──────────────┐  ┌───────────┐  ┌─────────────┐  │
│  │ Fuente_Leads │  │   Leads   │  │  Histórico  │  │
│  │ (origen)     │  │ (CRM)     │  │ (log)       │  │
│  └──────────────┘  └───────────┘  └─────────────┘  │
│                    ┌───────────┐                     │
│                    │ Dashboard │                     │
│                    │ (métricas)│                     │
│                    └───────────┘                     │
└─────────────────────────────────────────────────────┘
           ▲                ▲                ▲
           │                │                │
┌──────────┴──┐   ┌─────────┴──┐   ┌────────┴──────┐
│ Workflow 2  │   │ Workflow 1  │   │  Workflow 3   │
│ Alimentador │──▶│ Alta Lead   │   │ Actualizar    │
│ Automático  │   │ (webhook)   │   │ Estado        │
└─────────────┘   └────────────┘   └───────────────┘
                                          ▲
                                          │
                                   ┌──────┴──────┐
                                   │ Apps Script  │
                                   │ (onEdit)     │
                                   └─────────────┘
```

---

## Google Sheets — estructura y acceso

> ⚠️ El Google Sheets **no está incluido en este repositorio** porque vive en Google Drive, no como archivo local. Los JSON de n8n solo contienen la referencia al documento (ID, nombre de hoja y columnas), pero no los datos.

### 🔗 Acceso al Sheets

El Sheets utilizado por este proyecto tiene la siguiente estructura de pestañas:

---

### 📄 Pestaña `Leads`
Hoja principal del CRM. Cada fila es un lead o cliente.

| Columna | Campo | Descripción |
|---|---|---|
| A | `id` | ID único generado automáticamente por n8n |
| B | `nombre` | Nombre completo del lead |
| C | `email` | Email — usado para detectar duplicados |
| D | `telefono` | Teléfono — también detecta duplicados |
| E | `empresa` | Empresa u organización |
| F | `estado` | Estado actual en el pipeline |
| G | `comercial` | Nombre del comercial responsable |
| H | `fecha_alta` | Fecha y hora de registro |
| I | `fecha_ultima_accion` | Última vez que se actualizó |
| J | `observaciones` | Última nota del comercial |
| K | `origen` | De dónde vino el lead |

---

### 📄 Pestaña `Histórico`
Registro inmutable de todos los cambios de estado. Nunca se borra nada.

| Columna | Campo | Descripción |
|---|---|---|
| A | `id_registro` | ID único del registro histórico |
| B | `id_lead` | ID del lead al que pertenece |
| C | `fecha` | Cuándo ocurrió el cambio |
| D | `estado_anterior` | Estado previo al cambio |
| E | `estado_nuevo` | Nuevo estado tras el cambio |
| F | `comercial` | Quién realizó el cambio |
| G | `notas` | Comentario o nota del cambio |
| H | `email_lead` | Email del lead |

---

### 📄 Pestaña `Fuente_Leads`
Hoja de origen para la automatización diaria. El Workflow 2 lee de aquí.

| Columna | Campo | Descripción |
|---|---|---|
| A | `nombre` | Nombre del lead |
| B | `email` | Email |
| C | `telefono` | Teléfono |
| D | `empresa` | Empresa |
| E | `comercial` | Comercial asignado |
| F | `origen` | Origen del lead |
| G | `observaciones` | Notas iniciales |
| H | `enviado` | `no` = pendiente / `si` = ya procesado |
| I | `fecha_envio` | Fecha en que se procesó |

---

### 📄 Pestaña `Dashboard`
Generada automáticamente por Apps Script. No se edita manualmente.

---

## Credenciales necesarias en n8n

Para que los workflows funcionen correctamente necesitas configurar lo siguiente en n8n:

### 1. Google Sheets OAuth2 API
- Ve a `localhost:5678` → **Credentials** → **Add credential**
- Selecciona **Google Sheets OAuth2 API**
- Necesitas un **Client ID** y **Client Secret** de Google Cloud Console
- Activa las APIs: **Google Sheets API** y **Google Drive API**
- URI de redirección: `http://localhost:5678/rest/oauth2-credential/callback`

### 2. Spreadsheet ID
Cada nodo de Google Sheets en los workflows apunta a un documento concreto. Si importas los JSON en tu propio n8n, deberás actualizar el **Spreadsheet ID** en cada nodo con el ID de tu propio Sheets.

El ID se encuentra en la URL del Sheets:
```
https://docs.google.com/spreadsheets/d/{SPREADSHEET_ID}/edit
```

---

## Workflows

### Workflow 1 — Alta de Lead

**Disparador:** Petición POST al webhook `/alta-lead`

Este es el workflow principal del CRM. Recibe los datos de un lead, comprueba si ya existe y lo registra si es nuevo.

```
[Webhook /alta-lead]
        │
        ▼
[Normalizar Datos]
  · Email en minúsculas y sin espacios
  · Teléfono sin espacios
  · Origen = "manual" si no se especifica
        │
        ▼
[Leer Todos los Leads]
  · Lee toda la pestaña Leads del Sheets
        │
        ▼
[Comprobar Duplicados]
  · Compara email con todos los existentes
  · Compara teléfono con todos los existentes
        │
        ▼
[IF ¿Es Duplicado?]
        │
   ┌────┴────┐
   │         │
  SÍ        NO
   │         │
   ▼         ▼
[Responde  [Generar ID único]
 409]        · LEAD-YYYYMMDD-HHMMSS-XXX
             · Estado inicial = "nuevo"
             │
             ▼
        [Guardar en Leads]
             │
             ▼
        [Guardar en Histórico]
             │
             ▼
        [Responde 201 + ID]
```

**Ejemplo de llamada:**
```bash
curl -X POST http://localhost:5678/webhook/alta-lead \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "María García",
    "email": "maria@empresa.com",
    "telefono": "600123456",
    "empresa": "Inmobiliaria Sol",
    "comercial": "Juan López",
    "origen": "formulario_web",
    "observaciones": "Interesada en piso de 3 habitaciones"
  }'
```

---

### Workflow 2 — Alimentador Automático

**Disparador:** Schedule Trigger automático — por defecto cada día a las 9:00

Lee de la hoja `Fuente_Leads` y envía cada lead al Workflow 1 automáticamente, procesando solo los que tienen `enviado = no`.

```
[Schedule Trigger — 9:00 cada día]
        │
        ▼
[Leer Fuente_Leads]
  · Solo filas donde enviado = "no"
        │
        ▼
[Limitar a 30 leads por ejecución]
        │
        ▼
[Por cada lead → HTTP Request → POST /alta-lead]
  · Si responde 409 (duplicado) continúa sin parar
  · Si responde 201 (ok) el lead quedó registrado
        │
        ▼
[Wait — 3 segundos entre leads]
        │
        ▼
[Marcar como enviado en Fuente_Leads]
  · enviado = "si"
  · fecha_envio = ahora
```

---

### Workflow 3 — Actualizar Estado

**Disparador:**
- Automáticamente vía Apps Script cuando el vendedor edita la columna `estado` u `observaciones` en el Sheets
- Manualmente mediante petición POST al webhook `/actualizar-estado`

Gestiona el ciclo de vida de cada lead. Cada cambio queda registrado permanentemente en el histórico.

```
[Webhook /actualizar-estado]
        │
        ▼
[Validar Estado]
  · Verifica que sea un estado permitido
  · Si no es válido → responde 400
        │
        ▼
[Buscar Lead por ID en Leads]
  · Si no existe → responde 404
        │
        ▼
[Preparar Actualización]
  · Guarda el estado anterior para el histórico
        │
        ▼
[Actualizar Lead en Sheets]
  · estado, fecha_ultima_accion, observaciones, comercial
        │
        ▼
[Registrar en Histórico]
  · Añade fila permanente con estado anterior y nuevo
        │
        ▼
[Responde 200]
```

**Ejemplo de llamada:**
```bash
curl -X POST http://localhost:5678/webhook/actualizar-estado \
  -H "Content-Type: application/json" \
  -d '{
    "id_lead": "LEAD-20260320-143022-042",
    "nuevo_estado": "contactado",
    "comercial": "Juan López",
    "notas": "Llamada realizada, muy interesada. Enviar presupuesto esta semana."
  }'
```

---

## Dashboard

Generado automáticamente con **Google Apps Script** en la pestaña `Dashboard`.

**Cómo ejecutarlo:** Menú **🔧 CRM → 📊 Actualizar Dashboard** en Google Sheets

**Secciones:**

| Sección | Contenido |
|---|---|
| 📌 Resumen general | Total leads, activos, ganados, perdidos, tasa de conversión |
| 📋 Leads por estado | Cantidad y porcentaje en cada fase |
| 👤 Leads por comercial | Total, ganados y tasa de conversión individual |
| 📅 Actividad temporal | Leads esta semana, este mes, total acumulado |
| 🕐 Últimos movimientos | Los 10 últimos cambios de estado |
| 🆕 Leads recientes | Los 5 últimos leads registrados |

El Apps Script también incluye una función `onEdit` que detecta cuando el vendedor modifica directamente una celda de `estado` u `observaciones` en el Sheets y llama al Workflow 3 automáticamente.

---

## Estados disponibles

| Estado | Emoji | Descripción |
|---|---|---|
| `nuevo` | 🔵 | Lead recién registrado |
| `validado` | 🟣 | Datos confirmados |
| `contactado` | 🟡 | Primer contacto realizado |
| `pendiente` | 🟠 | Esperando respuesta |
| `presupuestado` | 🟤 | Oferta enviada |
| `cerrado_ganado` | 🟢 | Trato cerrado — es cliente |
| `cerrado_perdido` | 🔴 | Oportunidad perdida |

---

## Flujo completo de una oportunidad real

```
DÍA 1 — 9:00
Workflow 2 lee Fuente_Leads
→ Encuentra a "Sandra Molero"
→ La envía al Workflow 1
→ Workflow 1 la registra en Leads con estado "nuevo"
→ Histórico: — → nuevo

DÍA 1 — 11:00
Juan (comercial) llama a Sandra
→ Cambia estado en Sheets a "contactado"
→ Escribe en observaciones: "Llamada realizada, muy interesada"
→ Apps Script detecta el cambio → llama al Workflow 3
→ Histórico: nuevo → contactado

DÍA 3 — 09:00
Juan envía presupuesto a Sandra
→ Cambia estado a "presupuestado"
→ Observaciones: "Enviado presupuesto de 3.500€"
→ Workflow 3 registra el cambio
→ Histórico: contactado → presupuestado

DÍA 7 — 11:00
Sandra acepta el presupuesto
→ Juan cambia estado a "cerrado_ganado"
→ Observaciones: "Firmado contrato. Cliente desde hoy."
→ Workflow 3 cierra la oportunidad
→ Histórico: presupuestado → cerrado_ganado ✅
→ Dashboard actualiza: +1 cliente ganado
```

---

## Archivos del repositorio

| Archivo | Descripción |
|---|---|
| `workflow_1_alta_lead.json` | Workflow de registro de leads |
| `workflow_2_alimentador.json` | Workflow de automatización diaria |
| `workflow_3_actualizar_estado.json` | Workflow de actualización de estados |
| `apps_script_completo.gs` | Script de dashboard + detector de cambios |
| `estructura_sheets.md` | Referencia detallada de columnas |
| `fuente_leads_ejemplo.csv` | Datos de ejemplo para Fuente_Leads |

---
