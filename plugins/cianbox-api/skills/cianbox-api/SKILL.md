---
name: cianbox-api
description: >
  Referencia completa de la API REST de Cianbox, una plataforma argentina de gestion comercial y e-commerce.
  Usa esta skill siempre que el usuario mencione Cianbox, o necesite interactuar con una tienda Cianbox:
  consultar productos, clientes, pedidos, ventas, stock, MercadoLibre/MELI, webhooks, autenticacion,
  o cualquier integracion con Cianbox. Tambien cuando pida generar scripts, servidores MCP, o codigo
  de integracion con Cianbox. Incluso si el usuario no dice "Cianbox" explicitamente pero menciona
  endpoints como cianbox.org o tokens CBX_AT/CBX_RT, activa esta skill.
---

# Cianbox API — Guia de Integracion

Cianbox es una plataforma argentina de gestion comercial (ERP/e-commerce). Su API REST v2 permite gestionar productos, clientes, pedidos, ventas, stock, cotizaciones, webhooks e integracion con MercadoLibre.

## Antes de empezar

Cuando el usuario te pida algo relacionado con Cianbox, lo primero es entender dos cosas:

1. **Nombre de cuenta**: Todas las URLs de la API usan el patron `https://cianbox.org/{cuenta}/api/v2/...`. Pregunta al usuario cual es su nombre de cuenta si no lo mencionó.
2. **Autenticacion**: La API usa tokens de acceso (`access_token`). Si el usuario ya tiene uno, usalo. Si no, vas a necesar generar uno (ver seccion de autenticacion mas abajo).

## Referencia de la API

La documentacion completa de cada endpoint esta en `references/api-reference.md`. Leela cuando necesites los detalles exactos de parametros, respuestas o ejemplos de curl para un endpoint especifico.

La API cubre estos modulos:

| Modulo | Base URL | Operaciones |
|--------|----------|-------------|
| **Autenticacion** | `/api/v2/auth/` | Obtener credenciales, renovar token |
| **Productos** | `/api/v2/productos/` | Listar, filtrar por marca/categoria/codigo, marcas, categorias, listas de precio, temporadas, ajuste de stock |
| **Clientes** | `/api/v2/clientes/` | Listar, crear, editar, categorías de cliente (CRUD), cuenta corriente (`/ctacte`) |
| **Pedidos** | `/api/v2/pedidos/` | Listar, crear, eliminar, editar observaciones, estados de pedido (CRUD) |
| **Ventas** | `/api/v2/ventas/` | Listar, crear (directa/pedido/ML), recursos auxiliares (puntos de venta, tarjetas, entidades, planes, cuentas bancarias, retenciones, percepciones), preview de percepciones. Formas de pago: cuenta corriente, contado mixto, efectivo, tarjeta crédito/débito, financiación propia |
| **Stock** | `/api/v2/stock/` | Remitos internos: listar, crear, recibir (transferencias entre sucursales) |
| **Impuestos** | `/api/v2/impuestos/` | Tipos de percepción, regímenes impositivos |
| **MercadoLibre** | `/api/v2/mercadolibre/` | Listar ventas ML con filtros especificos |
| **Webhooks** | `/api/v2/general/notificaciones/` | Listar, asignar, eliminar webhooks |
| **Cotizaciones** | `/api/v2/general/cotizaciones/` | Listar cotizaciones de monedas |

## Autenticacion

La API usa un esquema de access_token + refresh_token:

- **access_token** (prefijo `CBX_AT-`): Token temporal, vence cada 24 horas. Se envia como query param `?access_token=...` en cada request.
- **refresh_token** (prefijo `CBX_RT-`): Sirve para renovar el access_token sin usuario/contraseña. Vence cada 600 dias.

### Flujo de autenticacion

```
1. POST /auth/credentials  (user + password + app_name + app_code)
   → Devuelve access_token + refresh_token + expires_in

2. Cuando vence el access_token:
   POST /auth/refresh  (refresh_token)
   → Devuelve nuevo access_token + expires_in

3. Cuando vence el refresh_token (cada 600 dias):
   Repetir paso 1
```

Cuando generes codigo de integracion, tene en cuenta que el access_token se pasa siempre como query parameter, no como header Authorization. Esto es particular de Cianbox.

## Patrones comunes

### Paginacion
Todos los endpoints GET de listado soportan `limit` y `page` como query params. Si el usuario necesita obtener todos los registros, implementa paginacion iterativa.

### Filtros por ID multiple
Muchos endpoints aceptan multiples IDs separados por coma: `?id=100,101,102`. Los webhooks pueden notificar hasta 200 IDs por peticion, y los endpoints GET tambien aceptan hasta 200 IDs, lo que facilita la sincronizacion.

### Estructura de respuesta
Todas las respuestas siguen este patron:
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "{cuenta}",
    "module": "...",
    "method": "GET|POST|PUT|DELETE",
    "body": [ ... ] | { ... }
}
```
- Para listados: `body` es un array
- Para operaciones de alta/edicion/baja: `body` es un objeto con `status`, `description` y a veces `id`

### Errores
Verifica siempre que `status` sea `"ok"`. Si el token venció, renova con el refresh_token y reintenta.

## Generando codigo de integracion

Cuando el usuario pida codigo, seguí estas pautas:

1. **Siempre incluí manejo de autenticacion**: el token vence cada 24 horas, asi que el codigo deberia poder renovarlo automaticamente.
2. **Soporta paginacion**: si hay que traer listados completos, iterá con `limit` y `page`.
3. **Lenguaje flexible**: genera en el lenguaje que el usuario prefiera (Python, Node.js, PHP, etc.). Si no lo especifica, usá Python como default por su claridad.
4. **Consulta la referencia**: antes de generar codigo para un endpoint especifico, leé `references/api-reference.md` para tener los parametros exactos y la estructura de la respuesta.

### Ejemplo: script Python para listar productos

```python
import requests

CUENTA = "micuenta"  # Configurar con la cuenta del usuario
BASE_URL = f"https://cianbox.org/{CUENTA}/api/v2"
ACCESS_TOKEN = "CBX_AT-..."  # Configurar con el token

def get_productos(limit=50, page=1, **filtros):
    params = {
        "access_token": ACCESS_TOKEN,
        "limit": limit,
        "page": page,
        **filtros
    }
    resp = requests.get(f"{BASE_URL}/productos", params=params)
    data = resp.json()
    if data.get("status") != "ok":
        raise Exception(f"Error de API: {data}")
    return data["body"]
```

## Construyendo un servidor MCP para Cianbox

Si el usuario quiere conectar Claude directamente a su cuenta de Cianbox (por ejemplo, "quiero que Claude pueda consultar mis productos"), la mejor opcion es construir un servidor MCP. Para esto:

1. Leé la skill `mcp-builder` si esta disponible — tiene las mejores practicas para construir servidores MCP.
2. Usa como referencia la documentacion de la API en `references/api-reference.md` para definir las tools del MCP.
3. Las tools mas utiles para un MCP de Cianbox suelen ser:
   - `list_productos` — listar/buscar productos con filtros
   - `get_producto` — obtener un producto por ID
   - `list_clientes` — listar/buscar clientes
   - `get_cuenta_corriente` — cuenta corriente de un cliente
   - `create_pedido` — crear un pedido
   - `list_pedidos` — listar pedidos con filtros
   - `list_ventas` — listar ventas
   - `create_venta` — crear venta (directa, desde pedido o desde ML)
   - `preview_percepciones` — calcular percepciones antes del alta
   - `ajuste_stock` — cargar ajustes de stock
   - `list_remitos_internos` — listar remitos internos (transferencias entre sucursales)
   - `create_remito_interno` — crear remito interno
   - `recibir_remito_interno` — marcar un remito interno como recibido
   - `list_ventas_ml` — ventas de MercadoLibre
4. Maneja la autenticacion internamente en el servidor: guarda el refresh_token y renova el access_token automaticamente cuando venza.

## Alta de Ventas

El endpoint `POST /api/v2/ventas/alta` permite crear ventas con distintos origenes y formas de pago. Es el endpoint mas complejo de la API.

### Flujo recomendado para crear una venta

1. Consultar **recursos auxiliares** (`puntos_venta`, `tarjetas`, `entidades`, `formas_entrega`, etc.) para obtener los IDs necesarios.
2. Si corresponde, obtener **percepciones automaticas** con `POST /api/v2/ventas/percepciones/preview`.
3. Armar el payload y enviarlo a `POST /api/v2/ventas/alta`.

### Origenes soportados
- `directa` — venta normal con `productos[]`
- `pedido` — factura un pedido existente (`id_pedido`)
- `mercadolibre` — factura una venta de ML (`id_venta_ml`)

### Formas de pago soportadas
- `cuenta_corriente` — sin cobro inmediato
- `efectivo` — cobro en efectivo
- `tarjeta_credito` / `tarjeta_debito` — requiere bloque `tarjeta`
- `financiacion_propia` — requiere bloque `financiacion`
- `contado_mixto` — requiere bloque `cobro` (efectivo + tarjetas + depositos + retenciones + valores)

### Reglas clave
- Las percepciones automaticas se resuelven **antes** del alta via preview, no dentro del alta.
- En `contado_mixto`, la suma de medios de pago debe igualar el total de la venta.
- `cobro.transferencias[]` no existe — usar `cobro.depositos[]` para depositos y transferencias.
- `entrega.id_pv_remito = "pendiente"` o `"no_stock"` evita movimiento de stock.

Consulta `references/api-reference.md` para ver los 9 casos de uso completos con payloads listos para usar.

## Stock / Transferencias entre sucursales

El modulo `/api/v2/stock/remito_interno` modela transferencias de stock entre sucursales reutilizando el circuito real del sistema (PEPS/UEPS, numeros de serie, datos de despacho).

### Flujo tipico

1. **Crear** el remito interno (`POST .../alta`) con `id_sucursal_origen`, `id_sucursal_destino`, `id_punto_venta` (talonario interno) y `productos[]`.
2. Si `entregado=false`, el stock sale de la sucursal de origen pero queda pendiente en transito hasta que la sucursal destino lo **reciba** (`POST .../recibir`).
3. Si `entregado=true` al alta, el movimiento se consolida inmediatamente (ingreso en destino).

### Reglas clave

- `id_sucursal_origen` debe ser distinta de `id_sucursal_destino`.
- Si la cuenta no tiene `forzar_entrega_remito` habilitado, el alta valida stock disponible en origen.
- `numeros_serie[]` solo aplica a productos con manejo de serie, y nunca puede superar `cantidad`.
- El numero de comprobante se asigna automaticamente.
- Un remito solo puede recibirse una vez (no se puede reingresar).

### Cuenta corriente de cliente

El endpoint `GET /api/v2/clientes/ctacte` devuelve movimientos y saldo de cuenta corriente del cliente. **La documentacion publica no lo incluye** — usalo con precaucion y confirmá parametros/respuesta contra una cuenta real antes de integrarlo.

## Webhooks

Cianbox puede notificar a tu aplicacion cuando ocurren cambios. Los eventos disponibles son: `clientes`, `pedidos`, `productos`, `categorias`, `marcas`, `listas_precio`, `sucursales`, `cotizaciones`, `estados`.

Cada notificacion envia un JSON con el evento y los IDs afectados (hasta 200 por notificacion). Tu aplicacion debe responder HTTP 200 en menos de 30 segundos.

Esto es ideal para sincronizacion en tiempo real — por ejemplo, mantener un sistema externo actualizado con los cambios de productos o pedidos de Cianbox.

## MercadoLibre / MELI

El modulo de MercadoLibre permite consultar ventas provenientes de la integracion con MercadoLibre. Tiene filtros especificos como `id_venta_ml`, `id_user_ml`, `id_publicacion_ml`, `vigente` y `cancelada`.

Nota: las URLs del modulo ML en la documentacion original usan `http://cianbox.test/` (entorno de prueba). En produccion, usá `https://cianbox.org/`.
