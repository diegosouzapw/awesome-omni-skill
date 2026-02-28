---
name: airparser-api
description: Guia para integrar con el servicio de parsing de documentos Airparser via API, webhooks y Make.com. Usar cuando se configuren inboxes, esquemas de extraccion, o flujos de automatizacion para procesamiento de recibos.
---

# Integracion API de Airparser

## 1) Vision General

Airparser es un servicio de parsing de documentos impulsado por LLM que extrae datos estructurados de emails, PDFs e imagenes. Esta skill cubre la integracion API, configuracion de esquemas de extraccion y flujos de automatizacion para el sistema de procesamiento de recibos de Transports Pau.

**Nota:** Airparser usa el termino "Inbox" para referirse a un parser/proyecto (equivalente al "Mailbox" de Parsio).

---

## 2) Autenticacion

### Clave API

La clave API se almacena en el archivo `.env` del proyecto:

```bash
# Archivo: .env (ya configurado, incluido en .gitignore)
AIRPARSER_API_KEY=tu_clave_api_aqui
```

La clave API se encuentra en: https://app.airparser.com/account

### Cargar variables de entorno

```bash
# Cargar .env antes de ejecutar comandos
source .env

# Verificar que esta cargada
echo $AIRPARSER_API_KEY
```

### URL Base

```
https://api.airparser.com
```

### Header de autenticacion

Usar `X-API-Key` en el header HTTP:

```bash
source .env && curl -X GET https://api.airparser.com/inboxes/ \
  -H "X-API-Key: $AIRPARSER_API_KEY"
```

---

## 3) Conceptos Clave

| Concepto | Descripcion |
|----------|-------------|
| **Inbox** | Proyecto/parser que recibe documentos. Tiene email propio |
| **Document** | Documento recibido (email, PDF, imagen) |
| **Extraction Schema** | Definicion de campos a extraer usando descripcion en lenguaje natural |
| **Webhook** | Endpoint para recibir datos parseados en tiempo real (con firma de seguridad) |

---

## 4) Referencia API - Inboxes

### Listar Inboxes

```bash
source .env && curl -X GET https://api.airparser.com/inboxes \
  -H "X-API-Key: $AIRPARSER_API_KEY"
```

### Eliminar Inbox

```bash
source .env && curl -X DELETE https://api.airparser.com/inboxes/<inbox_id> \
  -H "X-API-Key: $AIRPARSER_API_KEY"
```

### Obtener Inbox ID

El Inbox ID se encuentra en la URL del navegador cuando abres el inbox:

```
https://app.airparser.com/inboxes/<INBOX_ID>/documents
```

O via API listando los inboxes.

---

## 5) Referencia API - Documentos

### Subir documento para parsear

Formatos soportados: EML, PDF, HTML, TXT, MD, DOCX y mas
Tamano maximo: 20MB

```bash
source .env && curl -X POST https://api.airparser.com/inboxes/<inbox_id>/upload \
  -H "X-API-Key: $AIRPARSER_API_KEY" \
  -F "file=@recibo.pdf"
```

Con metadata opcional (se incluira en el JSON parseado como `__meta__`):

```bash
source .env && curl -X POST https://api.airparser.com/inboxes/<inbox_id>/upload \
  -H "X-API-Key: $AIRPARSER_API_KEY" \
  -F "file=@recibo.pdf" \
  -F 'meta={"conductor_id": "123", "vehiculo": "1234-ABC"}'
```

**Retorna:** Document ID

### Obtener documento con datos parseados

```bash
source .env && curl -X GET https://api.airparser.com/docs/<document_id>/extended \
  -H "X-API-Key: $AIRPARSER_API_KEY"
```

**Nota:** Se recomienda usar webhooks para recibir datos en tiempo real en lugar de polling.

### Listar documentos

```bash
source .env && curl -X GET "https://api.airparser.com/inboxes/<inbox_id>/docs?page=1" \
  -H "X-API-Key: $AIRPARSER_API_KEY"
```

Con filtros:

```bash
source .env && curl -X GET "https://api.airparser.com/inboxes/<inbox_id>/docs?page=1&from=2025-01-01&to=2025-12-31&statuses[]=parsed" \
  -H "X-API-Key: $AIRPARSER_API_KEY"
```

Parametros:
- **page** (number): Numero de pagina
- **from** (string): Fecha desde (YYYY-MM-DD)
- **to** (string): Fecha hasta (YYYY-MM-DD)
- **q** (string): Busqueda
- **statuses** (array): Estados: importing, parsed, fail, new, quota, parsing, exception

---

## 6) Referencia API - Extraction Schema

### Crear/Actualizar esquema de extraccion

```bash
source .env && curl -X POST https://api.airparser.com/inboxes/<inbox_id>/schema \
  -H "X-API-Key: $AIRPARSER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "fields": [
      {
        "type": "scalar",
        "data": {
          "name": "total_amount",
          "description": "El importe total del recibo",
          "type": "decimal",
          "default_value": "0.00"
        }
      }
    ]
  }'
```

### Tipos de campos

**1. Scalar (valores simples)**

```json
{
  "type": "scalar",
  "data": {
    "name": "nombre_campo",
    "description": "Descripcion para el LLM",
    "type": "string|integer|decimal|boolean",
    "default_value": "valor_por_defecto"
  }
}
```

Tipos soportados: `string`, `integer`, `decimal`, `boolean`

**2. List (arrays de objetos)**

```json
{
  "type": "list",
  "data": {
    "name": "lineas",
    "description": "Lista de items",
    "attributes": [
      {
        "name": "producto",
        "description": "Nombre del producto",
        "type": "string",
        "default_value": ""
      },
      {
        "name": "cantidad",
        "description": "Cantidad",
        "type": "integer",
        "default_value": "0"
      }
    ]
  }
}
```

**3. Object (objeto unico)**

```json
{
  "type": "object",
  "data": {
    "name": "direccion",
    "description": "Direccion de envio",
    "attributes": [
      {
        "name": "calle",
        "description": "Calle",
        "type": "string",
        "default_value": ""
      }
    ]
  }
}
```

**4. Enum (valores predefinidos)**

```json
{
  "type": "enum",
  "data": {
    "name": "estado",
    "description": "Estado del pedido",
    "values": ["pendiente", "procesando", "enviado", "entregado"]
  }
}
```

### Clonar esquema entre inboxes

```bash
source .env && curl -X POST https://api.airparser.com/inboxes/<source_inbox_id>/schema-clone \
  -H "X-API-Key: $AIRPARSER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"destination_inbox_id": "<target_inbox_id>"}'
```

### Reglas de validacion

- Nombres de campo: 1-100 caracteres, solo letras minusculas, numeros y guiones bajos
- Nombres unicos dentro del esquema
- Descripciones: max 3000 caracteres
- Valores por defecto: max 400 caracteres
- Valores de enum: unicos, 1-100 caracteres cada uno

---

## 7) Webhooks

### Configurar webhook (via UI)

1. Ir a Integrations → Webhooks en Airparser
2. Seleccionar "Create a webhook"
3. Pegar la URL de destino

Airparser envia datos via POST cada vez que se parsea un documento.

### Verificar firma del webhook (seguridad)

Airparser firma cada payload con un Signing Secret. La firma se envia en el header `airparser-signature`.

**Pseudocodigo:**
```
signature = base64(HMAC_SHA256(payload_binary, secret))
```

**Node.js ejemplo:**

```javascript
const crypto = require('crypto');

app.post('/webhook', express.raw({type: 'application/json'}), (req, res) => {
  const SIGNING_SECRET = "<your_secret_key>";
  const signature1 = req.get('airparser-signature');
  const signature2 = crypto
    .createHmac('sha256', SIGNING_SECRET)
    .update(req.rawBody)
    .digest('base64');

  if (signature1 === signature2) {
    // Firma valida
    res.sendStatus(200);
  } else {
    // Firma invalida - rechazar
    res.sendStatus(403);
  }
});
```

**PHP ejemplo:**

```php
$payload = @file_get_contents('php://input');
// Procesar payload...
http_response_code(200);
```

---

## 8) Esquema para Recibos Transports Pau

Esquema de extraccion configurado en `esquema-airparser.json`:

```json
{
  "fields": [
    {
      "type": "scalar",
      "data": {
        "name": "razon_social",
        "description": "Nombre del comercio/empresa del recibo. Recortar espacios, preservar acentos.",
        "type": "string",
        "default_value": ""
      }
    },
    {
      "type": "scalar",
      "data": {
        "name": "fecha",
        "description": "Fecha de transaccion mas cercana al total. Convertir DD/MM/YYYY o DD-MM-YYYY a YYYY-MM-DD.",
        "type": "string",
        "default_value": ""
      }
    },
    {
      "type": "scalar",
      "data": {
        "name": "matricula",
        "description": "Matricula espanola del vehiculo si aparece. Formato normalizado. Vacio si no encontrada.",
        "type": "string",
        "default_value": ""
      }
    },
    {
      "type": "scalar",
      "data": {
        "name": "importe",
        "description": "Total a pagar en EUR. Buscar TOTAL, TOTAL A PAGAR. Usar el total final si hay varios.",
        "type": "decimal",
        "default_value": "0.00"
      }
    },
    {
      "type": "enum",
      "data": {
        "name": "categoria",
        "description": "Clasificar por palabras clave: gasoil (gasoil/diesel/fuel), peajes (peaje/toll/autopista), limpieza_vehiculos (lavado/limpieza), otros (default).",
        "values": ["gasoil", "peajes", "limpieza_vehiculos", "otros"]
      }
    },
    {
      "type": "enum",
      "data": {
        "name": "pago",
        "description": "empresa = facturado a Transports Pau o tarjeta empresa. transportista = pagado por conductor. Vacio si ambiguo.",
        "values": ["empresa", "transportista"]
      }
    }
  ]
}
```

### Aplicar esquema via API

```bash
source .env && curl -X POST https://api.airparser.com/inboxes/<inbox_id>/schema \
  -H "X-API-Key: $AIRPARSER_API_KEY" \
  -H "Content-Type: application/json" \
  -d @esquema-airparser.json
```

---

## 9) Integracion con Make.com

### Trigger: Watch Document Parsed

1. Anadir modulo Airparser: "Watch Document Parsed"
2. Conectar cuenta Airparser (email/password)
3. Seleccionar inbox

### Action: Upload a Document for Parsing

Importa un documento externo a Airparser para parsearlo.

### Flujo de procesamiento

```
Trigger Airparser (doc.parsed)
    |
    v
Text Parser (normalizar matricula)
    |
    v
Text Parser (normalizar fecha)
    |
    v
Set Variable (mapeo categoria)
    |
    v
Google Drive (subir original)
    |
    v
Google Sheets (anadir fila)
```

### Si usaste Google Sign Up

Para conectar con Make necesitas credenciales email/password:
1. Cerrar sesion de Airparser
2. Ir a https://app.airparser.com/forgot
3. Crear nueva password via email

---

## 10) Ejemplo: Flujo completo

```bash
# 1. Cargar credenciales
source .env

# 2. Listar inboxes para obtener ID
curl -X GET https://api.airparser.com/inboxes \
  -H "X-API-Key: $AIRPARSER_API_KEY"

# Respuesta: [{"_id": "abc123", "name": "Recibos", ...}]

# 3. Aplicar esquema de extraccion
curl -X POST https://api.airparser.com/inboxes/abc123/schema \
  -H "X-API-Key: $AIRPARSER_API_KEY" \
  -H "Content-Type: application/json" \
  -d @esquema-airparser.json

# 4. Subir documento de prueba
curl -X POST https://api.airparser.com/inboxes/abc123/upload \
  -H "X-API-Key: $AIRPARSER_API_KEY" \
  -F "file=@recibo_gasoil.pdf" \
  -F 'meta={"matricula": "1234-ABC"}'

# Respuesta: "<document_id>"

# 5. Obtener datos parseados (o usar webhook)
curl -X GET https://api.airparser.com/docs/<document_id>/extended \
  -H "X-API-Key: $AIRPARSER_API_KEY"
```

---

## 11) Mapeo de Campos para Transports Pau

### Keywords de Categoria (CA/ES)

| Categoria | Keywords |
|-----------|----------|
| gasoil | gasoil, diesel, combustible, fuel, carburant |
| peajes | peatge, peaje, autopista, toll, via-t, telepeaje |
| limpieza_vehiculos | rentat, lavado, neteja, limpieza, autolavado |
| otros | (fallback por defecto) |

### Mapeo Tipo de Pago

| Tipo | Keywords |
|------|----------|
| empresa | targeta empresa, tarjeta empresa, compte, company card |
| transportista | efectivo, cash, efectiu, targeta personal, tarjeta personal |

### Normalizacion de Matricula

Formato entrada: `1234 ABC`, `1234-ABC`, `1234ABC`
Formato salida: `1234-ABC`

Regex: `^(\d{4})-?([A-Z]{3})$`

---

## 12) Manejo de Errores

| Codigo | Significado | Accion |
|--------|-------------|--------|
| 401 | API key invalida | Verificar AIRPARSER_API_KEY en .env |
| 404 | Inbox/documento no encontrado | Verificar IDs |
| 429 | Limite de peticiones | Esperar y reintentar |
| 500 | Error del servidor | Reintentar |

---

## 13) Recursos

- [Documentacion API Airparser](https://help.airparser.com/public-api/public-api)
- [Configurar Webhooks](https://help.airparser.com/data-export-integrations/webhooks)
- [Integracion Airparser + Make](https://help.airparser.com/data-export-integrations/make-integration)
- [Airparser Knowledge Base](https://help.airparser.com/)
