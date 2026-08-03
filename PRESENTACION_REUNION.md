# 🚀 Presentación Ejecutiva: Automatización de Carga de Facturas
## ALMAR Rosario — Proyecto de Integración API Kipintoch & ISO 9001
**Fecha:** 4 de agosto de 2026 | **Presentador:** Fran Bondino | **Versión:** 1.0

---

## 📌 ESTRUCTURA DE LA PRESENTACIÓN DE APERTURA

```
 1. Objetivo y Alcance    ➔ 2. Flujo AS-IS / Dolores ➔ 3. Arquitectura TO-BE ➔ 4. Integración API GET/POST
 5. Manejo de Mails (A/B/C) ➔ 6. Casos de Borde     ➔ 7. Accesos a Pedir    ➔ 8. Handoff a ISO 9001
```

---

## SLIDE 1: Objetivo y Alcance del Proyecto

### 🎯 ¿Qué Venimos a Resolver Hoy?

* **El Problema:** La carga de facturas de costos de proveedores (armadores, despachantes, fletes locales) se realiza mediante un proceso manual, lento y desestructurado a través de cadenas de emails informales.
* **La Solución:** Un bot inteligente que capta las facturas entrantes, lee los PDFs mediante IA/OCR, valida los datos contra la API de Kipintoch y los presenta en un **Dashboard de 1-Clic** para aprobación rápida del operador.
* **El Principio de Diseño:** **"Humano en el Loop" (Human-in-the-Loop)**. El bot realiza el 90% del trabajo pesado (lectura, clasificación y matcheo), pero la aprobación final la realiza el personal autorizado de ALMAR.

> [!IMPORTANT]
> **Foco Clave:** Todo el valor del desarrollo está en optimizar el tramo previo a Kipintoch. Una vez aprobado en el Dashboard, la API inyecta el costo de compra y el PDF de respaldo en la carpeta de forma automática e inmutable (cumpliendo ISO 9001:2015).

---

## SLIDE 2: Diagnóstico del Flujo Actual (AS-IS) & Puntos de Dolor

### 🔄 2.1 Flujo Operativo Actual (Sin Automatización)

```mermaid
flowchart TD
    subgraph PROVEEDORES["🏢 PROVEEDORES / ARMADORES / AGENTES"]
        P1["Línea Marítima (Maersk, MSC)"]
        P2["Agente Internacional en Origen"]
        P3["Despachante de Aduana"]
        P4["Transportista Local (Camión)"]
        P5["Terminal Portuaria"]
    end

    subgraph MAIL["📧 CADENA DE EMAILS (Sin Trazabilidad)"]
        M1["Emails llegan a casillas individuales de operadores"]
        M2["PDFs adjuntos dispersos (Facturas, Debit Notes)"]
        M3["Reenvíos internos y copias a Administración"]
    end

    subgraph MANUAL["👤 PROCESAMIENTO MANUAL (Cuello de Botella)"]
        H1["Buscar manualmente la CARPETA en Kipintoch"]
        H2["Verificar CUIT y Razón Social del Proveedor"]
        H3["Clasificar tipo de gasto (FIA, FMA, THC)"]
        H4["Validar montos vs estimado en cotización"]
        H5["Convertir moneda (USD -> ARS si aplica)"]
    end

    subgraph KIPIN["💻 KIPINTOCH (Carga Manual)"]
        K1["Ingresar costo de compra manualmente"]
        K2["Adjuntar PDF de respaldo"]
        K3["Cierre: cierreFacturaCosto ✅"]
    end

    P1 & P2 & P3 & P4 & P5 --> M1 --> M2 --> M3 --> H1 --> H2 --> H3 --> H4 --> H5 --> K1 --> K2 --> K3

    style MAIL fill:#FFFBEB,stroke:#F59E0B,color:#000
    style MANUAL fill:#FEF2F2,stroke:#EF4444,color:#000
    style KIPIN fill:#ECFDF5,stroke:#10B981,color:#000
```

### ⚠️ 2.2 Mapa de Dolor e Impacto Operativo

```mermaid
flowchart LR
    subgraph DOLORES["🔴 PUNTOS DE DOLOR ENCONTRADOS"]
        D1["📧 Facturas dispersas en múltiples correos"]
        D2["⏳ Carga manual lenta (10-15 min por factura)"]
        D3["❌ Riesgo de error de tipeo en CUITs / Montos"]
        D4["🔍 Búsqueda manual carpeta por carpeta"]
        D5["📋 Sin trazabilidad previa para auditoría ISO"]
        D6["💱 Conversión manual de divisas"]
    end

    subgraph IMPACTOS["💥 IMPACTO EN LA OPERACIÓN E ISO"]
        I1["Demora en el cierre contable de carpetas"]
        I2["Riesgo de facturar al cliente con márgenes desactualizados"]
        I3["Dificultad para auditar el cumplimiento (ISO 7.5)"]
        I4["Cuello de botella en períodos de alto volumen"]
    end

    D1 & D5 --> I3
    D2 & D4 --> I1
    D3 & D6 --> I2
    D2 & D3 --> I4

    style DOLORES fill:#FEF2F2,stroke:#EF4444,color:#000
    style IMPACTOS fill:#FFFBEB,stroke:#F59E0B,color:#000
```

---

## SLIDE 3: Arquitectura Propuesta (TO-BE) en 5 Módulos

```mermaid
flowchart TD
    subgraph PROVEEDORES["🏢 PROVEEDORES"]
        P["Envío de facturas por email"]
    end

    subgraph INGESTA["📥 MÓDULO 1: INGESTA DE EMAILS"]
        I1["Casillas corporativas de operadores"]
        I2["Listener automático (Graph API / IMAP)"]
        I3["Descarga y registro de adjuntos PDF/XML"]
    end

    subgraph EXTRACCION["🤖 MÓDULO 2: EXTRACCIÓN CON IA / OCR"]
        E1["Lectura de PDF mediante OCR + LLM"]
        E2["Extracción: CUIT, Monto, Moneda, HBL, Booking, Ítems"]
    end

    subgraph MATCHING["🔗 MÓDULO 3: MATCHING Y VALIDACIÓN (API)"]
        M1["GET /forwarding/carpetas -> Busca Carpeta"]
        M2["GET /empresas -> Valida CUIT Proveedor"]
        M3["GET /configuracion/gastos -> Mapea Nomenclador"]
        M4["GET /rentabilidad -> Compara vs Estimado"]
    end

    subgraph DASHBOARD["📊 MÓDULO 4: DASHBOARD DE REVISIÓN (HUMAN-IN-THE-LOOP)"]
        D1["Cola de facturas con score de confianza"]
        D2["Vista comparativa lado a lado: PDF vs Datos Extraídos"]
        D3["Aprobación o corrección por operador con 1 clic"]
        D4["Registro inmutable de auditoría (ISO 9001)"]
    end

    subgraph CARGA["✅ MÓDULO 5: CARGA AUTOMÁTICA EN KIPINTOCH"]
        C1["POST Costo de compra en carpeta (API)"]
        C2["POST Adjunto PDF de respaldo"]
        C3["Actualización de cierreFacturaCosto"]
    end

    P --> I1 --> I2 --> I3 --> E1 --> E2
    E2 --> M1 & M2 & M3 & M4 --> D1 --> D2 --> D3
    D3 --> C1 & C2 & C3 & D4

    style INGESTA fill:#EFF6FF,stroke:#2D65EB,color:#000
    style EXTRACCION fill:#F3E8FF,stroke:#9333EA,color:#000
    style MATCHING fill:#FFFBEB,stroke:#F59E0B,color:#000
    style DASHBOARD fill:#ECFDF5,stroke:#10B981,color:#000
    style CARGA fill:#ECFDF5,stroke:#10B981,color:#000
```

---

## SLIDE 4: Manejo de Casillas Descentralizadas (Móduo 1)

Sabemos que en ALMAR **no existe una casilla centralizada**, sino que cada operador recibe correos en su cuenta personal. Para evitar resistencia del personal, ofrecemos **2 opciones transparentes**:

```mermaid
flowchart TD
    subgraph OPCION_A["Opción A: Reglas de Reenvío M365 (Cero Cambio)"]
        A1["Proveedor manda mail a operador@almar.com"] --> A2["Regla M365 reenvía copia con PDF al Bot"]
        A2 --> A3["El empleado no cambia nada de su rutina"]
    end

    subgraph OPCION_B["Opción B: Listener M365 Multi-Casilla (Transparente)"]
        B1["Bot monitorea casillas del equipo via MS Graph API"] --> B2["Filtra automáticamente mails con facturas de proveedores"]
        B2 --> B3["Sin reenvíos ni casillas adicionales"]
    end
```

> [!TIP]
> **Recomendación:** La **Opción A** o **Opción B** son 100% transparentes para los operadores y no requieren cambiar los correos de contacto con proveedores ni exigir acciones manuales.

---

## SLIDE 5: Integración Integrada con la API de Kipintoch (GETs + POSTs)

```mermaid
flowchart TD
    subgraph ENTRADA["📄 1. ENTRADA DE DATOS"]
        PDF["Factura PDF / Email"] --> OCR["🤖 Extracción OCR + IA"]
        OCR --> JSON["JSON estructurado"]
    end

    subgraph LECTURA["🔍 2. FASE DE LECTURA Y VALIDACIONES (GETs)"]
        JSON --> G1["GET /forwarding/carpetas<br/>(Busca HBL / Booking / Ref)"]
        JSON --> G2["GET /empresas<br/>(Valida CUIT de Proveedor)"]
        JSON --> G3["GET /configuracion/gastos<br/>(Clasifica Código Nomenclador)"]
        JSON --> G4["GET /rentabilidad<br/>(Verifica Desvío vs Estimado)"]
        
        G1 & G2 & G3 & G4 --> SCORE["📊 Score de Confianza (Match Alto)"]
    end

    subgraph APROBACION["👤 3. FASE DE AUDITORÍA (DASHBOARD)"]
        SCORE --> DASH["🖥️ Dashboard Human-in-the-Loop<br/>(Revisión Comparativa Lado a Lado)"]
        DASH -->|✅ Operador Aprueba| WRITE["⚡ Orden de Escritura Backend"]
    end

    subgraph ESCRITURA["⚡ 4. FASE DE ESCRITURA E IMPACTO (POSTs)"]
        WRITE --> P1["POST /forwarding/carpetas/guardar<br/>(Imputa Costo de Compra en Carpeta)"]
        WRITE --> P2["POST /empresas/guardar<br/>(Alta de Proveedor Nuevo si aplica)"]
        WRITE --> P3["POST /adjuntos/obtener<br/>(Adjunta PDF de Respaldo a Carpeta)"]
        
        P1 & P2 & P3 --> CIERRE["✅ Impacto Contable Final:<br/>cierreFacturaCosto = TRUE<br/>Log de Auditoría ISO 9001 Registrado"]
    end

    style ENTRADA fill:#F8FAFC,stroke:#64748B,color:#000
    style LECTURA fill:#EFF6FF,stroke:#2D65EB,color:#000
    style APROBACION fill:#FFFBEB,stroke:#F59E0B,color:#000
    style ESCRITURA fill:#ECFDF5,stroke:#10B981,color:#000
```

---

## SLIDE 6: Diagrama de Secuencia Operativa (End-to-End)

```mermaid
sequenceDiagram
    autonumber
    actor P as 🏢 Proveedor
    participant E as 📧 Email (Listener)
    participant IA as 🤖 Motor OCR / IA
    participant BOT as ⚙️ Orquestador Backend
    participant API as 🌐 API Kipintoch
    actor OP as 👤 Operador ALMAR
    participant KIP as 💻 Kipintoch (DB)

    P->>EMAIL: Envía factura PDF por email
    EMAIL->>INGESTA: Listener detecta nuevo email
    INGESTA->>INGESTA: Descarga PDF adjunto
    INGESTA->>IA: Envía PDF para extracción

    IA->>IA: OCR + LLM extrae datos
    IA->>MATCH: JSON con datos extraídos

    MATCH->>API: GET /forwarding/carpetas (busca por HBL)
    API-->>MATCH: Carpeta encontrada (ID: 1435)

    MATCH->>API: GET /empresas (valida CUIT proveedor)
    API-->>MATCH: Proveedor validado (ID: 1247)

    MATCH->>API: GET /configuracion/gastos
    API-->>MATCH: Gasto clasificado (FMA)

    MATCH->>API: GET /rentabilidad (costo estimado)
    API-->>MATCH: Estimado: USD 3.100

    MATCH->>DASH: Factura lista para revisión<br/>(Score: 95%, match alto)

    DASH->>USER: Notificación: nueva factura pendiente
    USER->>DASH: Revisa datos lado a lado
    USER->>DASH: ✅ APRUEBA

    DASH->>KIPIN: POST carga costo de compra
    DASH->>KIPIN: POST adjunta PDF
    DASH->>KIPIN: Actualiza cierre contable

    Note over DASH: 📝 Log ISO: Usuario X aprobó<br/>factura INV-2026-04521<br/>el 04/08/2026 a las 14:32
```

---

## SLIDE 7: Tratamiento de Casos de Borde (Edge Cases)

| Edge Case | Escenario en la Vida Real | Solución Asistida de la Arquitectura |
|:----------|:--------------------------|:-------------------------------------|
| **1. Sin HBL/Booking** | Fletes de camión local que no traen referencia directa de ALMAR | **Búsqueda Heurística Multinivel:** Búsqueda por `CUIT Proveedor` + `N° Contenedor` + `Fechas (±5 días)`. Si hay dudas, presenta 2 candidatas para elección en 1 clic. |
| **2. Proveedor Nuevo** | Factura de una grúa, taller o despachante no registrado en Kipintoch | **Pre-alta Asistida via API:** El OCR extrae datos fiscales del PDF y el Dashboard ofrece el botón **"Dar de Alta Proveedor y Aprobar"** (ejecuta `POST /empresas/guardar`). |
| **3. Factura Multi-Ítem** | Facturas de Maersk/MSC con múltiples renglones de gastos heterogéneos | **Desglose Inteligente (Item Split):** Se separa cada renglón (`FMA`, `THC`, `DOC`). Si aparece un concepto nuevo, el operador confirma la sugerencia y el bot **memoriza la equivalencia**. |

---

## SLIDE 8: Accesos y Requerimientos a Pedir Hoy

Para iniciar el desarrollo del backend y el dashboard de inmediato, se solicita:

1. ✅ **API Token Kipintoch:** *(Ya disponible y validado en la API).*
2. 🔑 **Acceso de Lectura a Emails:** Permisos de lectura en Microsoft 365 (`Mail.Read`) o credenciales para el conector de ingesta.
3. 📄 **Lote de Facturas Reales de Prueba (Dataset):** 10 a 20 PDFs de facturas reales recientes (Maersk, MSC, despachantes, fletes locales, Invoices en USD) para entrenar el motor OCR/IA.
4. ❓ **Respuestas Operativas Clave:**
   - Confirmar el dato prioritario de búsqueda (`HBL`, `Booking`, `Ref Interna`).
   - Definir quién aprueba en el Dashboard (Operador asignado o Administración).
   - Definir el umbral de desvío aceptado en el costo estimado vs factura real.

---

## SLIDE 9: Roadmap de Ejecución & Handoff a ISO 9001

```mermaid
gantt
    title Plan de Desarrollo - Automatización de Carga de Facturas ALMAR
    dateFormat  YYYY-MM-DD
    axisFormat  %W/%M

    section Fase 0: Validación
    Reunión de Validación & Requerimientos  :active, m1, 2026-08-04, 0d
    Cierre de Especificación Técnica         :m2, after m1, 4d

    section Fase 1: Ingesta & OCR
    Conector de Email & Ingesta             :f1_1, 2026-08-11, 7d
    Entrenamiento OCR e IA de Extracción     :f1_2, after f1_1, 10d

    section Fase 2: Integration API
    Conexión API Kipintoch (Carpetas/Empresas):f2_1, after f1_1, 10d
    Motor de Matching & Mapeo de Gastos     :f2_2, after f2_1, 7d

    section Fase 3: UI & Piloto
    Dashboard de Revisión Human-in-the-Loop :f3_1, after f2_2, 10d
    Pruebas Piloto con Facturas Reales      :f3_2, after f3_1, 7d
    Despliegue a Producción & Capacitación  :milestone, f3_3, after f3_2, 0d
```

> 🤝 **Pase de Mando:** Con el marco técnico cerrado y la solicitud de accesos realizada, ceder la palabra al consultor ISO para revisar el Mapa de Procesos y el Manual de Gestión de Calidad.
