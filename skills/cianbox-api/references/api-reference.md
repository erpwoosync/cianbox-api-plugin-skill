# Documentación API Cianbox

Esta documentación ha sido generada a partir del repositorio [cianbox/api-docs](https://github.com/cianbox/api-docs).

## Tabla de Contenidos

1. [Autenticación](#autenticación)
2. [Webhooks](#webhooks)
3. [General](#general)
4. [Productos](#productos)
5. [Clientes](#clientes)
6. [Pedidos/Órdenes](#pedidosórdenes)
7. [Ventas](#ventas)
8. [Stock / Remitos Internos](#stock--remitos-internos)
9. [Mercado Libre](#mercado-libre)

---

## Autenticación

### Obtener credenciales

**Descripción:**
Obtiene las credenciales de acceso (token de acceso y refresco) usando usuario y contraseña válidos de Cianbox.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/auth/credentials`

**Método:** `POST`

**Parámetros:**

| Parámetro | Requerido | Descripción | Ejemplo |
|---|---|---|---|
| app_name | SI | Nombre de la aplicación | Mi App re Copada |
| app_code | SI | Código de la aplicación | mi-app-re-copada |
| user | SI | Nombre de usuario de Cianbox | |
| password | SI | Contraseña de Cianbox | |

**Ejemplos:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{"app_name":"Mi App","app_code":"mi-app","user":"usuario","password":"secreto1234"}' \
'https://cianbox.org/micuenta/api/v2/auth/credentials'

curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
-d "app_name=Mi App&app_code=mi-app&user=usuario&password=secreto1234" \
'https://cianbox.org/micuenta/api/v2/auth/credentials'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "micuenta",
    "module": "auth",
    "method": "POST",
    "body": {
        "access_token": "CBX_AT-R8xyfKa0Wi9qeIG5-130033-q2cxQIojoHVQnsXxmiXQinXmJlUEWt1p-570984383",
        "refresh_token": "CBX_RT-8hY8A80WGMiHVKCKFOrXjtMP-734794790-SkAeRIGtEJeQRmhG-446258",
        "expires_in": 85497
    }
}
```

**Valor Descripción:**
* `access_token`: Token de acceso temporal, vence cada 24 horas (tiempo restante determinado por expires_in)
* `refresh_token`: Token de refresco, sirve para solicitar un nuevo token de acceso. Vence cada 600 días, y una vez vencido hay que repetir el proceso de obtención de credenciales vía usuario y contraseña
* `expires_in`: Tiempo restante del token de acceso expresado en segundos

### Renovar credenciales

**Descripción:**
Obtiene un nuevo token de acceso usando un token de refresco válido.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/auth/refresh`

**Método:** `POST`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| refresh_token | SI | Token de refresco válido |

**Ejemplos:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{"refresh_token":"CBX_RT-czntKqNbP87PLT6ai2c0mDle-289145332-3ONbLx2q4o9AtkZu-559849"}' \
'https://cianbox.org/micuenta/api/v2/auth/refresh'

curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
-d "refresh_token=CBX_RT-czntKqNbP87PLT6ai2c0mDle-289145332-3ONbLx2q4o9AtkZu-559849" \
'https://cianbox.org/micuenta/api/v2/auth/refresh'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "micuenta",
    "module": "auth",
    "method": "POST",
    "body": {
        "access_token": "CBX_AT-c2pGCQqkR8QefYGw-947800-R6Rq8NwqvrCHuk9zmoddW49bDbhrMGqD-311038496",
        "expires_in": 86107
    }
}
```

**Valor Descripción:**
* `access_token`: Token de acceso temporal, vence cada 24 horas (tiempo restante determinado por expires_in)
* `expires_in`: Tiempo restante del token de acceso expresado en segundos

---

## Webhooks

### Recibe notificaciones

Para que tu aplicación pueda conocer los eventos que se producen del lado de Cianbox® y actuar en consecuencia, podés suscribir una o varias URLs (webhooks) a nuestro sistema de notificaciones.

Disponemos de una variedad de eventos que disparan notificaciones en tiempo real cuando se realizan diferentes acciones dentro de Cianbox®. Por ej. cuando el usuario cambia el precio de un producto, se modifican los datos de un cliente o se da de baja un pedido.

Con los siguientes recursos podés manejar tus suscripciones:

* Listar eventos y webhooks
* Asignar un webhook a eventos
* Dar de baja un webhook

Una vez suscripto vas a recibir el siguiente JSON en tu/s URL/s:

```json
{
    "event":"productos",
    "created":"2019-05-24 10:58:44",
    "id":[
        "10564", "10725", "587", "964"
    ],
    "endpoint":"productos"
}
```

Cada evento puede notificar hasta 200 ids por cada petición. Así mismo los recursos GET tambien permiten hasta 200 ids por cada petición, facilitando así la sincronización de los datos.

Por ejempo con los datos de arriba podemos hacer la siguiente petición:

```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/productos?id=10564,10725,587,964&access_token=CBX_AT-TcIHdWOvdpIMx...'
```

**Consideraciones:**
* Para que la entrega de la notificación se considere exitosa, tu aplicación deberá devolver un estado HTTP 200.
* Tu aplicación debe responder antes de los 30 segundos, de lo contrario se considerará la notificación como fallida.

**Eventos:**

Los eventos a los cuales podés suscribir tus URLs son:

| Evento | Recurso |
|---|---|
| clientes | `https://cianbox.org/micuenta/api/v2/clientes` |
| pedidos | `https://cianbox.org/micuenta/api/v2/pedidos` |
| productos | `https://cianbox.org/micuenta/api/v2/productos` |
| categorias | `https://cianbox.org/micuenta/api/v2/productos/categorias` |
| marcas | `https://cianbox.org/micuenta/api/v2/productos/marcas` |
| listas_precio | `https://cianbox.org/micuenta/api/v2/productos/listas` |
| sucursales | `https://cianbox.org/micuenta/api/v2/productos/sucursales` |
| cotizaciones | `https://cianbox.org/micuenta/api/v2/general/cotizaciones` |
| estados | `https://cianbox.org/micuenta/api/v2/pedidos/estados` |

**Código de ejemplo (PHP):**
```php
<?php
header("Access-Control-Allow-Orgin: *");
header("Access-Control-Allow-Methods: *");
header('Content-Type: application/json; charset=UTF-8');
header("HTTP/1.1 200 OK");

$data = json_decode(file_get_contents("php://input"), TRUE);

$evento = $data['event'];
$ids    = $data['id'];
/*
 * Acá va el código de tu aplicación para realizar la petición a la API
 */
?>
```

### Listar eventos y webhooks

**Descripción:**
Obtiene un listado de los webhooks configurados para cada evento

**URL:**
`https://cianbox.org/{cuenta}/api/v2/general/notificaciones`
o
`https://cianbox.org/{cuenta}/api/v2/general/notificaciones/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id de los eventos ej. 2 o 1,3,4 |
| evento | NO | nombres de los eventos ej. producto o clientes,marca,categoria |

Se puede filtrar por id o por evento, pero no por los dos a la vez.
Los eventos disponibles son: clientes, pedidos, productos, categorias, marcas, listas_precio, sucursales, cotizaciones

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/general/notificaciones?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "micuenta",
    "module": "gr_config",
    "method": "GET",
    "body": [
        {
            "id": 1,
            "evento": "clientes",
            "creado": "2019-05-21 22:39:27",
            "updated": "2019-05-23 22:26:08",
            "url": "http://urldeejemplo.com.ar/ntf.php"
        },
        ...
    ]
}
```

### Asignar un webhook a eventos

**Descripción:**
Asigna un webhook a un evento

**URL:**
`https://cianbox.org/{cuenta}/api/v2/general/notificaciones/alta`

**Método:** `POST`

**Parámetros:**
```json
{
    "evento": ["clientes", "pedidos"],
    "url": "http://urldeejemplo.com.ar/ntf.php"
}
```
La variable evento puede ser un array con los nombres de los eventos o un string "all" para asignar a todos los eventos.
Los eventos disponibles son: clientes, pedidos, productos, categorias, marcas, listas_precio, sucursales, cotizaciones y estados.

**Ejemplo:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{ \
        "evento": ["productos", "marcas", "categorias"], \
        "url": "http://urldeejemplo.com.ar/ntf.php" \
    }' \
'https://cianbox.org/micuenta/api/v2/general/notificaciones/alta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "micuenta",
    "module": "gr_config",
    "method": "POST",
    "body": {
        "status": "ok",
        "descripcion": "La url se informó con éxito"
    }
}
```

### Dar de baja un webhook

**Descripción:**
Da de baja un webhook relacionado a uno o más eventos

**URL:**
`https://cianbox.org/{cuenta}/api/v2/general/notificaciones`
o
`https://cianbox.org/{cuenta}/api/v2/general/notificaciones/eliminar`

**Método:** `DELETE`

**Parámetros:**
```json
{
    "evento": ["clientes", "pedidos"]
}
```
La variable evento puede ser un array con los nombres de los eventos o un string "all" para asignar a todos los eventos.

**Ejemplo:**
```bash
curl -X DELETE -H "Content-Type: application/json" \
-d '{ \
        "evento": ["productos", "marcas", "categorias"] \
    }' \
'https://cianbox.org/micuenta/api/v2/general/notificaciones?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "micuenta",
    "module": "gr_config",
    "method": "DELETE",
    "body": {
        "status": "ok",
        "descripcion": "La url se eliminó con éxito"
    }
}
```

---

## General

### Obtener Cotizaciones

**Descripción:**
Obtiene una cotización o lista de cotizaciones

**URL:**
`https://cianbox.org/{cuenta}/api/v2/general/cotizaciones`
o
`https://cianbox.org/{cuenta}/api/v2/general/cotizaciones/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id de la cotización ej. 1 o 1,3,4 |
| moneda | NO | id de la moneda ej. 1 o 1,2 |
| fecha | NO | fecha de la cotización ej. 2019-01-01 |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/general/cotizaciones?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "micuenta",
    "module": "gr_config",
    "method": "GET",
    "body": [
        {
            "id": 1,
            "moneda": "Dolar",
            "compra": "44.00",
            "venta": "46.00",
            "fecha": "2019-05-21 10:00:00"
        }
    ]
}
```

---

## Productos

### Obtener Productos

**Descripción:**
Obtiene un producto o lista de productos

**URL:**
`https://cianbox.org/{cuenta}/api/v2/productos`
o
`https://cianbox.org/{cuenta}/api/v2/productos/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id del producto ej. 100 o 100,101,102 |
| codigo | NO | código del producto ej. PROD-01 |
| id_marca | NO | id de la marca |
| id_categoria | NO | id de la categoría |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/productos?limit=5&access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "micuenta",
    "module": "pv_productos",
    "method": "GET",
    "body": [
        {
            "id": 100,
            "codigo": "PROD-100",
            "producto": "Producto de Ejemplo",
            "stock": 50,
            "precio": 1500.00,
            ...
        }
    ]
}
```

### Obtener Marcas

**Descripción:**
Obtiene una marca o lista de marcas

**URL:**
`https://cianbox.org/{cuenta}/api/v2/productos/marcas`
o
`https://cianbox.org/{cuenta}/api/v2/productos/marcas/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id de la marca ej. 5 o 5,6,7 |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/productos/marcas?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": [
        {
            "id": 5,
            "marca": "Marca Ejemplo"
        }
    ]
}
```

### Obtener Categorías

**Descripción:**
Obtiene una categoría o lista de categorías

**URL:**
`https://cianbox.org/{cuenta}/api/v2/productos/categorias`
o
`https://cianbox.org/{cuenta}/api/v2/productos/categorias/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id de la categoría ej. 10 o 10,11 |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/productos/categorias?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": [
        {
            "id": 10,
            "categoria": "Categoría Ejemplo",
            "padre": 0
        }
    ]
}
```

### Obtener Listas de Precio

**Descripción:**
Obtiene una lista de precios o todas

**URL:**
`https://cianbox.org/{cuenta}/api/v2/productos/listas`
o
`https://cianbox.org/{cuenta}/api/v2/productos/listas/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id de la lista ej. 1 o 1,2 |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/productos/listas?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": [
        {
            "id": 1,
            "lista": "Lista Minorista",
            "moneda": "Pesos"
        }
    ]
}
```

### Obtener Temporadas

**Descripción:**
Obtiene una temporada o lista de temporadas

**URL:**
`https://cianbox.org/{cuenta}/api/v2/productos/temporadas`
o
`https://cianbox.org/{cuenta}/api/v2/productos/temporadas/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id de la/s temporada/s ej. 1 o 1,2,3 |
| page | NO | Página solicitada |
| limit | NO | Límite de ítems por petición |
| fields | NO | Cualquiera de los listados en **available_fields** separados por comas |
| order | NO | Ordena el listado, acepta `name-asc`, `name-desc`, `id-asc`, `id-desc` |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/productos/temporadas?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "module": "pv_productos",
    "available_fields": ["id", "temporada", "vigente"],
    "body": [
        { "id": 1, "temporada": "OTOÑO", "vigente": true },
        { "id": 2, "temporada": "INVIERNO", "vigente": true },
        { "id": 3, "temporada": "VERANO", "vigente": true }
    ]
}
```

### Cargar un Ajuste de Stock

**Descripción:**
Realiza un ajuste de stock en una sucursal, sobre varios productos

**URL:**
`https://cianbox.org/{cuenta}/api/v2/productos/ajuste_stock/alta`

**Método:** `POST`

**Parámetros:**
```json
{
    "id_sucursal": 5,
    "id_usuario": 3,
    "ajustes": [
        {"id_producto": 265, "stock": 10.00},
        {"id_producto": 761, "stock": 2.00},
        {"id_producto": 17, "stock": 0.00}
    ]
}
```

**Ejemplo:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{ \
    "id_sucursal": 5,\
    "id_usuario": 3,\
    "ajustes": [\
        {"id_producto": 265, "stock": 10.00},\
        {"id_producto": 761, "stock": 2.00},\
        {"id_producto": 17, "stock": 0.00}\
    ]\
    }'\
'https://cianbox.org/micuenta/api/v2/productos/ajuste_stock/alta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": {
        "status": "ok",
        "description": "El ajuste de stock se cargó correctamente"
    }
}
```

---

## Clientes

### Obtener Clientes

**Descripción:**
Obtiene un cliente o lista de clientes

**URL:**
`https://cianbox.org/{cuenta}/api/v2/clientes`
o
`https://cianbox.org/{cuenta}/api/v2/clientes/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id del cliente ej. 100 o 100,101 |
| documento | NO | documento del cliente |
| email | NO | email del cliente |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/clientes?limit=5&access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": [
        {
            "id": 100,
            "razon": "Juan Perez",
            "documento": "20123456789",
            "email": "juan@example.com",
            ...
        }
    ]
}
```

### Obtener Categorías de Cliente

**Descripción:**
Obtiene una categoría de cliente o lista de categorías de cliente.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/clientes/categorias`
o
`https://cianbox.org/{cuenta}/api/v2/clientes/categorias/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id de la/s categoría/s de cliente ej. 15 o 33,17,19 |
| page | NO | Página solicitada |
| limit | NO | Límite de ítems por petición |
| fields | NO | Cualquiera de los listados en **available_fields** separados por comas |
| order | NO | Ordena el listado, acepta `name-asc`, `name-desc`, `id-asc`, `id-desc` |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/clientes/categorias?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "module": "pv_clientes",
    "available_fields": ["id", "categoria", "vigente"],
    "body": [
        { "id": 1, "categoria": "FERRETERIA", "vigente": true },
        { "id": 2, "categoria": "ELECTRONICA", "vigente": true },
        { "id": 3, "categoria": "BAZAR", "vigente": true }
    ]
}
```

### Cargar una Categoría de Cliente

**Descripción:**
Crea una nueva categoría de cliente.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/clientes/categorias/alta`

**Método:** `POST`

**Parámetros:**
```json
{
    "categoria": "FERRETERIA"
}
```

**Ejemplo:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{"categoria": "FERRETERIA"}' \
'https://cianbox.org/micuenta/api/v2/clientes/categorias/alta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": {
        "status": "ok",
        "description": "Categorias: 1 nueva(s) y 0 editada(s)",
        "id": 20
    }
}
```

### Editar una Categoría de Cliente

**Descripción:**
Edita una categoría de cliente existente.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/clientes/categorias/editar`

**Método:** `PUT`

**Parámetros vía URL:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | SI | id de la categoría a editar |

**Parámetros vía JSON:**
```json
{
    "categoria": "FERRETERIA"
}
```

**Ejemplo:**
```bash
curl -X PUT -H "Content-Type: application/json" \
-d '{"categoria": "FERRETERIA"}' \
'https://cianbox.org/micuenta/api/v2/clientes/categorias/editar?id=2&access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": {
        "status": "ok",
        "description": "Categorias: 0 nueva(s) y 1 editada(s)",
        "id": 2
    }
}
```

### Cargar un Cliente

**Descripción:**
Da de alta un nuevo cliente

**URL:**
`https://cianbox.org/{cuenta}/api/v2/clientes/alta`

**Método:** `POST`

**Parámetros:**
(Ver documentación completa para lista de campos)

**Ejemplo:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{"razon":"Nuevo Cliente", "documento":"20112233445"}' \
'https://cianbox.org/micuenta/api/v2/clientes/alta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": {
        "id": 105,
        "status": "ok",
        "description": "El cliente se cargó correctamente"
    }
}
```

### Cuenta Corriente de Cliente

**Descripción:**
Obtiene la cuenta corriente de un cliente (movimientos, saldo, comprobantes imputados).

> **TODO:** documentación oficial pendiente. Este endpoint no está publicado en el repo público [cianbox/api-docs](https://github.com/cianbox/api-docs). Parámetros, filtros y estructura de respuesta deben confirmarse con el proveedor o mediante pruebas contra la API real.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/clientes/ctacte`

**Método:** `GET` (a confirmar)

**Parámetros conocidos:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id_cliente | probable | ID del cliente del que se consulta la cuenta corriente |

**Ejemplo tentativo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/clientes/ctacte?id_cliente=100&access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Notas:**
- Validar parámetros aceptados (fechas, paginación, filtros de saldo) antes de usar en producción.
- Verificar si soporta también POST para operaciones sobre la cuenta corriente.

### Editar un Cliente

**Descripción:**
Edita un cliente existente

**URL:**
`https://cianbox.org/{cuenta}/api/v2/clientes/editar`

**Método:** `PUT`

**Parámetros:**
Requiere `id` del cliente y campos a modificar.

**Ejemplo:**
```bash
curl -X PUT -H "Content-Type: application/json" \
-d '{"id": 105, "razon":"Cliente Modificado"}' \
'https://cianbox.org/micuenta/api/v2/clientes/editar?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": {
        "status": "ok",
        "description": "El cliente se modificó correctamente"
    }
}
```

---

## Pedidos/Órdenes

### Obtener Pedidos

**Descripción:**
Obtiene un pedido o lista de pedidos

**URL:**
`https://cianbox.org/{cuenta}/api/v2/pedidos`
o
`https://cianbox.org/{cuenta}/api/v2/pedidos/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id del pedido ej. 1000 |
| id_cliente | NO | id del cliente |
| fecha_desde | NO | fecha desde (creación) |
| fecha_hasta | NO | fecha hasta (creación) |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/pedidos?limit=5&access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": [
        {
            "id": 1000,
            "fecha": "2019-05-20",
            "cliente": "Juan Perez",
            "total": 5000.00,
            ...
        }
    ]
}
```

### Crear un Pedido

**Descripción:**
Crea un nuevo pedido

**URL:**
`https://cianbox.org/{cuenta}/api/v2/pedidos/alta`

**Método:** `POST`

**Parámetros:**
(Ver documentación completa para estructura JSON)

**Ejemplo:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{ "id_cliente": 100, "items": [...] }' \
'https://cianbox.org/micuenta/api/v2/pedidos/alta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": {
        "id": 1001,
        "status": "ok",
        "description": "El pedido se cargó correctamente"
    }
}
```

### Obtener Estados de Pedidos

**Descripción:**
Obtiene los estados de pedidos disponibles

**URL:**
`https://cianbox.org/{cuenta}/api/v2/pedidos/estados`

**Método:** `GET`

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/pedidos/estados?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": [
        {
            "id": 1,
            "estado": "Pendiente",
            "color": "#ff0000"
        }
    ]
}
```

### Cargar un Estado de Pedidos

**Descripción:**
Crea un nuevo estado de pedido

**URL:**
`https://cianbox.org/{cuenta}/api/v2/pedidos/estados/alta`

**Método:** `POST`

**Ejemplo:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{"estado":"En Preparacion", "color":"#00ff00"}' \
'https://cianbox.org/micuenta/api/v2/pedidos/estados/alta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

### Editar un Estado de Pedidos

**Descripción:**
Edita un estado de pedido

**URL:**
`https://cianbox.org/{cuenta}/api/v2/pedidos/estados/editar`

**Método:** `PUT`

### Dar de baja un Estado de Pedidos

**Descripción:**
Elimina un estado de pedido

**URL:**
`https://cianbox.org/{cuenta}/api/v2/pedidos/estados/eliminar`

**Método:** `DELETE`

### Dar de baja un Pedido

**Descripción:**
Elimina (anula) un pedido

**URL:**
`https://cianbox.org/{cuenta}/api/v2/pedidos/eliminar`

**Método:** `DELETE`

**Parámetros:**
`id`: ID del pedido a eliminar.

**Ejemplo:**
```bash
curl -X DELETE -H "Content-Type: application/json" \
-d '{"id": 1001}' \
'https://cianbox.org/micuenta/api/v2/pedidos/eliminar?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

### Editar Observaciones de un Pedido

**Descripción:**
Edita las observaciones de un pedido existente

**URL:**
`https://cianbox.org/{cuenta}/api/v2/pedidos/editar-observaciones`

**Método:** `PUT`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido (query param) |
| id | SI | ID del pedido (query param) |

**Payload:**
```json
{
  "observaciones": "Esta es una observación de prueba"
}
```

**Ejemplo:**
```bash
curl -X PUT -H "Content-Type: application/json" \
-d '{"observaciones": "Esta es una observación de prueba"}' \
'https://cianbox.org/micuenta/api/v2/pedidos/editar-observaciones?id=1001&access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": {
        "status": "ok",
        "description": "Las observaciones se modificaron correctamente"
    }
}
```

---

## Ventas

### Obtener Ventas

**Descripción:**
Obtiene una venta o lista de ventas

**URL:**
`https://cianbox.org/{cuenta}/api/v2/ventas`
o
`https://cianbox.org/{cuenta}/api/v2/ventas/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | id de la venta |
| fecha_desde | NO | fecha desde |
| fecha_hasta | NO | fecha hasta |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/ventas?limit=5&access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": [
        {
            "id": 5000,
            "fecha": "2019-05-20",
            "total": 1500.00,
            ...
        }
    ]
}
```

### Recursos Auxiliares para Alta de Ventas

Endpoints auxiliares para preparar el alta de ventas por API v2. Todos utilizan el mismo `access_token` del resto de la API.

| Endpoint | Uso |
|---|---|
| `GET /api/v2/ventas/puntos_venta` | Consulta talonarios habilitados para ventas |
| `GET /api/v2/ventas/puntos_venta_cobro` | Consulta talonarios habilitados para cobros/recibos |
| `GET /api/v2/ventas/formas_entrega` | Consulta remitos/formas de entrega y opciones especiales (`pendiente`, `no_stock`) |
| `GET /api/v2/ventas/domicilios_entrega?id_cliente=...` | Consulta domicilios de entrega del cliente |
| `GET /api/v2/ventas/empresas_transporte` | Consulta transportistas |
| `GET /api/v2/ventas/tarjetas` | Consulta tarjetas habilitadas |
| `GET /api/v2/ventas/entidades` | Consulta entidades emisoras/bancarias |
| `GET /api/v2/ventas/planes_tarjeta?id_tarjeta=...&id_entidad=...` | Consulta planes vigentes para una tarjeta y entidad |
| `GET /api/v2/ventas/cuotas_plan?id_plan=...` | Consulta cuotas configuradas para un plan |
| `GET /api/v2/ventas/cuentas_bancarias` | Consulta cuentas bancarias para los movimientos bancarios de `contado_mixto` |
| `GET /api/v2/ventas/tipos_retencion` | Consulta tipos de retención habilitados para `contado_mixto` |
| `GET /api/v2/impuestos/tipos_percepcion` | Consulta tipos de percepción disponibles |
| `GET /api/v2/impuestos/regimenes` | Consulta regímenes impositivos |

**Ejemplos:**
```bash
# Puntos de venta
curl 'https://cianbox.org/micuenta/api/v2/ventas/puntos_venta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...&id_cliente=257'

# Puntos de venta de cobro
curl 'https://cianbox.org/micuenta/api/v2/ventas/puntos_venta_cobro?access_token=CBX_AT-TcIHdWOvdpIMNsXG...&id_punto_venta_venta=1'

# Tarjetas
curl 'https://cianbox.org/micuenta/api/v2/ventas/tarjetas?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'

# Planes de tarjeta
curl 'https://cianbox.org/micuenta/api/v2/ventas/planes_tarjeta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...&id_tarjeta=32&id_entidad=1'

# Cuotas de un plan
curl 'https://cianbox.org/micuenta/api/v2/ventas/cuotas_plan?access_token=CBX_AT-TcIHdWOvdpIMNsXG...&id_plan=1'

# Cuentas bancarias
curl 'https://cianbox.org/micuenta/api/v2/ventas/cuentas_bancarias?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'

# Tipos de retención
curl 'https://cianbox.org/micuenta/api/v2/ventas/tipos_retencion?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'

# Tipos de percepción
curl 'https://cianbox.org/micuenta/api/v2/impuestos/tipos_percepcion?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Notas:**
- Los listados siguen el esquema estándar de paginación de API v2 (`page`, `limit`, `total_pages`).
- `planes_tarjeta` y `cuotas_plan` pueden devolver un `body` vacío si la cuenta todavía no configuró planes.
- Para `contado_mixto`, el `id_punto_venta` del bloque `cobro` debe salir de un talonario de cobros/recibos, no de uno de ventas.
- `puntos_venta_cobro?id_punto_venta_venta=...` ayuda a encontrar sólo talonarios de cobro con la misma fiscalidad que la venta.
- `formas_entrega` ya incluye los talonarios reales de remito y también las opciones especiales `0` (remito interno), `pendiente` y `no_stock`.
- `cuentas_bancarias` se usa para `depositos[]`, que cubre tanto depósitos como transferencias bancarias.
- No existe un array separado `transferencias[]` en el contrato público de `ventas/alta`; todos los movimientos bancarios van por `depositos[]`.
- `tipos_retencion` se usa para completar `retenciones[]`.
- Para `valores[]`, la entidad bancaria sale de `GET /api/v2/ventas/entidades`; no hay un recurso separado de "tipos de valor" porque el contrato de alta replica los campos que consume el recibo del sistema.

### Preview de Percepciones

**Descripción:**
Resuelve las percepciones automáticas de una venta utilizando la misma lógica funcional que usa la interfaz de ventas. La respuesta devuelve `percepciones[]` listo para reutilizarse en `POST /api/v2/ventas/alta`.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/ventas/percepciones/preview`

**Método:** `POST`

**Parámetros:**
```json
{
    "fecha": "2026-04-01",
    "id_cliente": 257,
    "id_punto_venta": 1,
    "productos": [
        {
            "id": 95,
            "cantidad": 1,
            "neto_uni": 16211,
            "alicuota": 21
        },
        {
            "id": 0,
            "tipo": "concepto_libre",
            "detalle": "Instalacion",
            "cantidad": 1,
            "neto_uni": 1000,
            "alicuota": 21
        }
    ]
}
```

**Ejemplo:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{
        "fecha": "2026-04-01",
        "id_cliente": 257,
        "id_punto_venta": 1,
        "productos": [
            {
                "id": 95,
                "cantidad": 1,
                "neto_uni": 16211,
                "alicuota": 21
            }
        ]
    }' \
'https://cianbox.org/micuenta/api/v2/ventas/percepciones/preview?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "http",
    "host": "cianbox.test",
    "account": "northsa",
    "module": "pv_ventas",
    "method": "POST",
    "body": {
        "status": "ok",
        "percepciones": [
            {
                "id_tipo_percepcion": 18,
                "id_regimen": 0,
                "id_impuesto": 3,
                "base_imponible": 17211,
                "monto": 516.33,
                "alicuota": 3
            }
        ]
    }
}
```

**Uso recomendado:**
1. Consultar el preview.
2. Tomar el array `body.percepciones`.
3. Enviarlo sin cambios dentro de `percepciones[]` al endpoint `POST /api/v2/ventas/alta`.

**Notas:**
- Si para el cliente/punto de venta no corresponde aplicar percepciones automáticas, la respuesta será `percepciones: []`.
- Este endpoint no crea ventas ni modifica datos comerciales.

### Crear una Venta

**Descripción:**
Crea una nueva venta reutilizando el circuito real del sistema. Soporta venta directa, facturación de pedido y facturación de venta de Mercado Libre.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/ventas/alta`

**Método:** `POST`

**Payload base:**
```json
{
    "fecha": "2026-04-01",
    "origen": {
        "tipo": "directa"
    },
    "id_cliente": 256,
    "id_canal_venta": 1,
    "forma_pago": "cuenta_corriente",
    "id_punto_venta": 1,
    "id_moneda": 1,
    "cotizacion": 1,
    "observaciones": "Venta generada por integracion API",
    "fecha_vencimiento": "2026-04-30",
    "entrega": {
        "id_pv_remito": 0,
        "id_domicilio_entrega": 0,
        "id_empresa_transporte": 0,
        "cantidad_bultos": 0,
        "observaciones_remito": ""
    },
    "productos": [
        {
            "id": 95,
            "cantidad": 1,
            "neto_uni": 16211,
            "alicuota": 21
        }
    ],
    "percepciones": []
}
```

**Formas de pago soportadas:**

| Valor | Descripción |
|---|---|
| `cuenta_corriente` | Genera la venta en cuenta corriente |
| `contado_mixto` | Genera la venta y luego un recibo/cobro imputando la misma venta |
| `efectivo` | Genera la venta cobrada en efectivo |
| `tarjeta_credito` | Requiere bloque `tarjeta` |
| `tarjeta_debito` | Requiere bloque `tarjeta` |
| `financiacion_propia` | Requiere bloque `financiacion` |

**Orígenes soportados:**

| Campo | Uso |
|---|---|
| `tipo = "directa"` | Venta normal con `productos[]` obligatorios |
| `tipo = "pedido"` | Debe informar `id_pedido`; los ítems se reconstruyen desde el pedido |
| `tipo = "mercadolibre"` | Debe informar `id_venta_ml`; los ítems se reconstruyen desde la venta de ML. `id_cliente` puede omitirse si la venta de ML tiene datos suficientes para detectar o crear el cliente. `forma_pago` y, si corresponde, `tarjeta.*` deben venir siempre por payload. `entrega.id_pv_remito` no toma defaults desde la configuración de ML |

**Bloques condicionales:**

#### Bloque `tarjeta`

Se usa con `forma_pago = "tarjeta_credito"` o `forma_pago = "tarjeta_debito"`.

```json
{
    "tarjeta": {
        "id_tarjeta": 32,
        "id_entidad": 1,
        "cuotas": 2,
        "cupon": "998877",
        "numero_lote": "77"
    }
}
```

#### Bloque `financiacion`

Se usa con `forma_pago = "financiacion_propia"`.

```json
{
    "financiacion": {
        "entrega": 1000,
        "periodicidad": "mensual",
        "cuotas": 3,
        "primer_vencimiento": "2026-05-10",
        "dia_vencimiento": 10,
        "dia_semana": "monday",
        "dia_anio": "10/05",
        "codigo_autorizacion": "AUT-123"
    }
}
```

#### Bloque `cobro`

Se usa con `forma_pago = "contado_mixto"`.

```json
{
    "cobro": {
        "id_punto_venta": 7,
        "fecha": "2026-04-01",
        "observaciones": "Cobro contado mixto",
        "efectivo": 10000,
        "tarjetas": [
            {
                "tipo": "credito",
                "id_tarjeta": 32,
                "id_entidad": 1,
                "cuotas": 2,
                "cupon": "551122",
                "numero_lote": "88",
                "monto": 9615.31
            }
        ],
        "depositos": [],
        "retenciones": [],
        "valores": []
    }
}
```

**Reglas importantes:**
- `id_punto_venta` es obligatorio y siempre se numera automáticamente.
- La factura electrónica se genera siempre en forma diferida.
- Las percepciones automáticas no se resuelven dentro del alta. Deben obtenerse antes con `POST /api/v2/ventas/percepciones/preview`.
- `percepciones[]` es opcional y puede incluir percepciones manuales o las obtenidas desde el preview.
- `entrega.id_pv_remito = 0` (o el campo omitido) significa remito interno. Si la venta tiene ítems que mueven stock, el sistema genera igualmente el remito con `id_punto_venta = 0`.
- `entrega.id_pv_remito > 0` usa un talonario real de remito.
- `entrega.id_pv_remito = "pendiente"` no genera remito y no mueve stock.
- `entrega.id_pv_remito = "no_stock"` no genera remito y no mueve stock.
- En `contado_mixto`, `cobro.id_punto_venta` debe ser un talonario de cobros/recibos y debe coincidir en fiscalidad con la venta.
- La suma de `efectivo + tarjetas + depositos + retenciones + valores` debe ser igual al total final de la venta.
- `cobro.depositos[]` usa el shape (`id_cuenta`, `fecha`, `numero`, `monto`) y cubre tanto depósitos como transferencias bancarias.
- `cobro.transferencias[]` ya no forma parte del contrato público y devuelve error. Todos los movimientos bancarios deben informarse en `cobro.depositos[]`.
- `cobro.retenciones[]` requiere `id_retencion` o `id_tipo_retencion`, `punto_venta`, `numero`, `monto_sujeto` y `monto_retenido`. Los demás campos son opcionales.
- `cobro.valores[]` requiere `id_entidad`, `numero_cheque`, `fecha_cheque`, `fecha_vencimiento` y `monto`. Los demás campos del cheque/valor son opcionales.
- `forma_pago = "tarjeta_credito"` y `forma_pago = "tarjeta_debito"` persisten el dato de la tarjeta en la venta, no generan recibo.
- `forma_pago = "financiacion_propia"` genera la estructura de cuotas en cuenta corriente según el bloque `financiacion`.

**Casos de uso:**

#### 1. Venta directa en cuenta corriente
```json
{
    "fecha": "2026-04-01",
    "origen": { "tipo": "directa" },
    "id_cliente": 256,
    "id_canal_venta": 1,
    "forma_pago": "cuenta_corriente",
    "id_punto_venta": 1,
    "id_moneda": 1,
    "cotizacion": 1,
    "productos": [
        { "id": 95, "cantidad": 1, "neto_uni": 16211, "alicuota": 21 }
    ],
    "percepciones": []
}
```

#### 2. Venta directa con percepciones manuales
```json
{
    "fecha": "2026-04-01",
    "origen": { "tipo": "directa" },
    "id_cliente": 257,
    "id_canal_venta": 1,
    "forma_pago": "cuenta_corriente",
    "id_punto_venta": 1,
    "id_moneda": 1,
    "cotizacion": 1,
    "productos": [
        { "id": 95, "cantidad": 1, "neto_uni": 16211, "alicuota": 21 }
    ],
    "percepciones": [
        {
            "id_tipo_percepcion": 1,
            "id_regimen": 0,
            "id_impuesto": 4,
            "base_imponible": 16211,
            "monto": 100,
            "alicuota": 0.6169
        }
    ]
}
```

#### 3. Facturación de pedido
```json
{
    "fecha": "2026-04-01",
    "origen": {
        "tipo": "pedido",
        "id_pedido": 77416
    },
    "id_cliente": 256,
    "id_canal_venta": 1,
    "forma_pago": "cuenta_corriente",
    "id_punto_venta": 1,
    "id_moneda": 1,
    "cotizacion": 1,
    "percepciones": []
}
```

#### 4. Facturación de venta de Mercado Libre
```json
{
    "fecha": "2026-04-01",
    "origen": {
        "tipo": "mercadolibre",
        "id_venta_ml": 427496
    },
    "forma_pago": "tarjeta_credito",
    "id_punto_venta": 67,
    "entrega": {
        "id_pv_remito": 0
    },
    "tarjeta": {
        "id_tarjeta": 32,
        "id_entidad": 1,
        "cuotas": 1,
        "cupon": "ML-427496",
        "numero_lote": "1"
    },
    "percepciones": []
}
```

#### 5. Venta con tarjeta de crédito
```json
{
    "fecha": "2026-04-01",
    "origen": { "tipo": "directa" },
    "id_cliente": 256,
    "id_canal_venta": 1,
    "forma_pago": "tarjeta_credito",
    "id_punto_venta": 4,
    "id_moneda": 1,
    "cotizacion": 1,
    "productos": [
        { "id": 95, "cantidad": 1, "neto_uni": 16211, "alicuota": 21 }
    ],
    "tarjeta": {
        "id_tarjeta": 32,
        "id_entidad": 1,
        "cuotas": 2,
        "cupon": "998877",
        "numero_lote": "77"
    }
}
```

#### 6. Venta con tarjeta de débito
```json
{
    "fecha": "2026-04-01",
    "origen": { "tipo": "directa" },
    "id_cliente": 256,
    "id_canal_venta": 1,
    "forma_pago": "tarjeta_debito",
    "id_punto_venta": 4,
    "id_moneda": 1,
    "cotizacion": 1,
    "productos": [
        { "id": 95, "cantidad": 1, "neto_uni": 16211, "alicuota": 21 }
    ],
    "tarjeta": {
        "id_tarjeta": 32,
        "id_entidad": 1,
        "cuotas": 1,
        "cupon": "998878",
        "numero_lote": "78"
    }
}
```

#### 7. Venta con financiación propia
```json
{
    "fecha": "2026-04-01",
    "origen": { "tipo": "directa" },
    "id_cliente": 256,
    "id_canal_venta": 1,
    "forma_pago": "financiacion_propia",
    "id_punto_venta": 4,
    "id_moneda": 1,
    "cotizacion": 1,
    "productos": [
        { "id": 95, "cantidad": 1, "neto_uni": 16211, "alicuota": 21 }
    ],
    "financiacion": {
        "entrega": 1000,
        "periodicidad": "mensual",
        "cuotas": 3,
        "primer_vencimiento": "2026-05-10",
        "dia_vencimiento": 10,
        "dia_semana": "monday",
        "dia_anio": "10/05",
        "codigo_autorizacion": "AUT-123"
    }
}
```

#### 8. Venta con contado mixto simple
```json
{
    "fecha": "2026-04-01",
    "origen": { "tipo": "directa" },
    "id_cliente": 257,
    "id_canal_venta": 1,
    "forma_pago": "contado_mixto",
    "id_punto_venta": 1,
    "id_moneda": 1,
    "cotizacion": 1,
    "productos": [
        { "id": 95, "cantidad": 1, "neto_uni": 16211, "alicuota": 21 }
    ],
    "percepciones": [],
    "cobro": {
        "id_punto_venta": 7,
        "fecha": "2026-04-01",
        "efectivo": 10000,
        "tarjetas": [
            {
                "tipo": "credito",
                "id_tarjeta": 32,
                "id_entidad": 1,
                "cuotas": 2,
                "cupon": "551122",
                "numero_lote": "88",
                "monto": 9615.31
            }
        ],
        "depositos": [],
        "retenciones": [],
        "valores": []
    }
}
```

#### 9. Venta con contado mixto completo
```json
{
    "fecha": "2026-04-01",
    "origen": { "tipo": "directa" },
    "id_cliente": 257,
    "id_canal_venta": 1,
    "forma_pago": "contado_mixto",
    "id_punto_venta": 1,
    "id_moneda": 1,
    "cotizacion": 1,
    "productos": [
        { "id": 95, "cantidad": 1, "neto_uni": 16211, "alicuota": 21 }
    ],
    "percepciones": [],
    "cobro": {
        "id_punto_venta": 7,
        "fecha": "2026-04-01",
        "efectivo": 3000,
        "tarjetas": [
            {
                "tipo": "credito",
                "id_tarjeta": 32,
                "id_entidad": 1,
                "cuotas": 2,
                "cupon": "771199",
                "numero_lote": "991",
                "monto": 5000
            }
        ],
        "depositos": [
            {
                "id_cuenta": 1,
                "fecha": "2026-04-01",
                "numero": "22334",
                "monto": 2000
            },
            {
                "id_cuenta": 1,
                "fecha": "2026-04-01",
                "numero": "33445",
                "monto": 4000
            }
        ],
        "retenciones": [
            {
                "id_retencion": 1,
                "fecha": "2026-04-01",
                "fecha_afec": "2026-04-01",
                "punto_venta": 1,
                "numero": 12345678,
                "monto_sujeto": 19615.31,
                "monto_retenido": 3115.31
            }
        ],
        "valores": [
            {
                "id_entidad": 1,
                "sucursal": 0,
                "codigo_postal": 0,
                "emitio": "API TEST",
                "cuit_emitio": "20123456789",
                "numero_cheque": "44556677",
                "numero_cuenta": 123456,
                "fecha_cheque": "2026-04-01",
                "fecha_vencimiento": "2026-04-15",
                "no_a_la_orden": false,
                "echeque": false,
                "cruzado": false,
                "monto": 2500
            }
        ]
    }
}
```

**Ejemplo genérico con `curl`:**
```bash
curl -X POST -H "Content-Type: application/json" \
-d '<payload del caso de uso elegido>' \
'https://cianbox.org/micuenta/api/v2/ventas/alta?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "http",
    "host": "cianbox.test",
    "account": "northsa",
    "module": "pv_ventas",
    "method": "POST",
    "body": {
        "status": "ok",
        "description": "La venta se cargo correctamente",
        "id": 565738,
        "id_recibo": 600922
    }
}
```

| Valor | Descripción |
|---|---|
| `status` | Estado de la petición: **ok** o **error** |
| `description` | Descripción del resultado |
| `id` | ID de la venta |
| `id_recibo` | ID del recibo/cobro generado cuando `forma_pago = contado_mixto` |

**Webhooks de ventas:**
- El alta de venta dispara el evento `ventas`.
- Las modificaciones principales de ventas y remitos vinculados también disparan el evento `ventas`.
- El alta del webhook se realiza desde el módulo de notificaciones de API usando `evento = ["ventas"]`.

---

## Stock / Remitos Internos

Módulo para gestionar transferencias de stock entre sucursales (remitos internos). Permite listar, crear y marcar como recibidos los remitos internos reutilizando el mismo circuito del sistema (imputación PEPS/UEPS, números de serie, datos de despacho/importación).

### Listar Remitos Internos

**Descripción:**
Obtiene un remito interno o lista de remitos internos de transferencia entre sucursales.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/stock/remito_interno`
o
`https://cianbox.org/{cuenta}/api/v2/stock/remito_interno/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id | NO | ID del/los remitos internos. Acepta uno o varios separados por coma |
| id_usuario_emitio | NO | Filtra por usuario que emitió/cargó el remito. También se acepta `id_usuario` por compatibilidad |
| id_usuario_recibio | NO | Filtra por usuario que recibió el remito |
| id_sucursal_origen | NO | Filtra por sucursal de origen |
| id_sucursal_destino | NO | Filtra por sucursal de destino |
| id_punto_venta | NO | Filtra por talonario |
| entregado | NO | Filtra por estado de recepción. Acepta `true` o `false` |
| fecha_desde | NO | Fecha mínima del comprobante (`YYYY-MM-DD`) |
| fecha_hasta | NO | Fecha máxima del comprobante (`YYYY-MM-DD`) |
| fecha_carga_desde | NO | Fecha mínima de carga (`YYYY-MM-DD`) |
| fecha_carga_hasta | NO | Fecha máxima de carga (`YYYY-MM-DD`) |
| fecha_recepcion_desde | NO | Fecha mínima de recepción (`YYYY-MM-DD`) |
| fecha_recepcion_hasta | NO | Fecha máxima de recepción (`YYYY-MM-DD`) |
| search | NO | Busca por número, observaciones, talonario, sucursales o usuario |
| page | NO | Página solicitada |
| limit | NO | Límite de ítems por petición. Máximo 200 |
| fields | NO | Cualquiera de los listados en **available_fields** separados por comas. Por defecto devuelve el resumen sin `detalles` |
| order | NO | Ordena el listado. Acepta `create-date-asc`, `create-date-desc`, `modified-date-asc`, `modified-date-desc`, `id-asc`, `id-desc` |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/stock/remito_interno?access_token=CBX_AT-TcIHdWOvdpIMNsXG...&id=12345&fields=id,fecha,fecha_carga,fecha_entrega,fecha_recepcion,entregado,vigente,anulado,id_usuario_emitio,usuario_emitio,id_usuario_recibio,usuario_recibio,id_punto_venta,talonario,comprobante,punto_venta,numero,id_sucursal_origen,sucursal_origen,id_sucursal_destino,sucursal_destino,observaciones,detalles'
```

**Respuesta:**
```json
{
    "status": "ok",
    "scheme": "https",
    "host": "cianbox.org",
    "account": "micuenta",
    "module": "pv_stock",
    "method": "GET",
    "available_fields": [
        "id", "fecha", "fecha_carga", "fecha_entrega", "fecha_recepcion",
        "entregado", "vigente", "anulado",
        "id_usuario_emitio", "usuario_emitio", "id_usuario_recibio", "usuario_recibio",
        "id_punto_venta", "talonario", "comprobante", "punto_venta", "numero",
        "id_sucursal_origen", "sucursal_origen",
        "id_sucursal_destino", "sucursal_destino",
        "observaciones", "detalles"
    ],
    "page": 1,
    "total_pages": 1,
    "body": [
        {
            "id": 12345,
            "fecha": "2026-04-09",
            "fecha_carga": "2026-04-09 10:15:22",
            "fecha_entrega": "2026-04-09",
            "fecha_recepcion": "2026-04-09 13:42:11",
            "entregado": true,
            "vigente": true,
            "anulado": false,
            "id_usuario_emitio": 99,
            "usuario_emitio": "Usuario Demo",
            "id_usuario_recibio": 104,
            "usuario_recibio": "Operador Demo",
            "id_punto_venta": 7,
            "talonario": "REMITO INTERNO DEMO",
            "comprobante": "REM [X] 0007-00001234",
            "punto_venta": "0007",
            "numero": "00001234",
            "id_sucursal_origen": 1,
            "sucursal_origen": "Sucursal Depósito A",
            "id_sucursal_destino": 2,
            "sucursal_destino": "Sucursal Venta B",
            "observaciones": "Transferencia interna de ejemplo",
            "detalles": [
                {
                    "id": 50001,
                    "id_producto": 1001,
                    "detalle": "Producto de ejemplo sin serie",
                    "cantidad": 2,
                    "numeros_serie": []
                },
                {
                    "id": 50002,
                    "id_producto": 1002,
                    "detalle": "Producto de ejemplo con serie",
                    "cantidad": 1,
                    "numeros_serie": ["SERIE-DEMO-0001"]
                }
            ]
        }
    ]
}
```

**Notas:**
- Para incluir el detalle de productos, pedir `fields=...,detalles`.
- `fecha` corresponde a `fecha_comprobante`.
- `entregado=true` indica que el remito ya fue recibido en la sucursal destino.

### Crear Remito Interno

**Descripción:**
Crea un remito interno de transferencia de stock entre sucursales reutilizando el circuito real del sistema.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/stock/remito_interno/alta`

**Método:** `POST`

**Payload base:**
```json
{
    "fecha": "2026-04-09",
    "id_sucursal_origen": 1,
    "id_punto_venta": 7,
    "id_sucursal_destino": 3,
    "observaciones": "Transferencia de mercaderia entre sucursales",
    "entregado": false,
    "productos": [
        {
            "id_producto": 265,
            "cantidad": 10
        },
        {
            "id_producto": 761,
            "cantidad": 2,
            "numeros_serie": [
                { "sn": "NS-0001", "aduana": "", "anio": "", "id_pais": 0, "id_uso": 0 },
                { "sn": "NS-0002", "aduana": "", "anio": "", "id_pais": 0, "id_uso": 0 }
            ]
        }
    ]
}
```

**Reglas importantes:**
- `fecha` es obligatoria y debe tener formato `YYYY-MM-DD`.
- `id_sucursal_origen` es obligatorio.
- `id_punto_venta` es obligatorio y debe corresponder a un talonario interno habilitado para el usuario autenticado.
- `id_sucursal_destino` es obligatoria.
- `id_sucursal_destino` debe ser distinta de `id_sucursal_origen`.
- El número de comprobante siempre se asigna automáticamente.
- `entregado` es obligatorio y acepta `true` o `false`.
- `productos[]` es obligatorio y acepta hasta 200 ítems.
- Cada ítem debe informar `id_producto` y `cantidad`.
- `numeros_serie[]` es opcional y sólo puede enviarse para productos que manejen números de serie.
- La cantidad de números de serie de un ítem nunca puede superar la cantidad del producto.
- Los productos deben estar asociados a la integración API autenticada.
- Si la cuenta no tiene habilitado `forzar_entrega_remito`, el endpoint valida stock disponible en la sucursal de origen antes de ejecutar el alta.
- El alta mantiene la misma lógica del sistema para imputación PEPS/UEPS, números de serie y datos de despacho/importación.

**Respuesta exitosa:**
```json
{
    "status": "ok",
    "description": "El remito interno se cargó correctamente",
    "id": 12345
}
```

**Casos de uso:**

#### 1. Remito interno pendiente de recibir
```json
{
    "fecha": "2026-04-09",
    "id_sucursal_origen": 1,
    "id_punto_venta": 7,
    "id_sucursal_destino": 3,
    "observaciones": "Envio a deposito central",
    "entregado": false,
    "productos": [
        { "id_producto": 265, "cantidad": 10 }
    ]
}
```

#### 2. Remito interno entregado al momento del alta
```json
{
    "fecha": "2026-04-09",
    "id_sucursal_origen": 1,
    "id_punto_venta": 7,
    "id_sucursal_destino": 3,
    "entregado": true,
    "productos": [
        { "id_producto": 761, "cantidad": 2 }
    ]
}
```

### Recibir Remito Interno

**Descripción:**
Recibe un remito interno pendiente, ingresando el stock en la sucursal destino mediante el mismo circuito del sistema.

**URL:**
`https://cianbox.org/{cuenta}/api/v2/stock/remito_interno/recibir`

**Método:** `POST`

**Payload:**
```json
{
    "id": 12345
}
```

**Reglas importantes:**
- `id` es obligatorio. También se acepta `id_remito` por compatibilidad.
- El remito interno debe existir y encontrarse vigente.
- El remito interno no debe haber sido recibido previamente.
- La recepción reutiliza la misma lógica del sistema para ingreso de stock, traspaso de números de serie y conservación de datos de despacho/importación.

**Respuesta exitosa:**
```json
{
    "status": "ok",
    "description": "El remito interno se ingresó correctamente",
    "id": 12345
}
```

---

## Mercado Libre

### Obtener Ventas ML

**Descripción:**
Obtiene una venta o lista de ventas de Mercado Libre

**URL:**
`http://cianbox.test/{cuenta}/api/v2/mercadolibre/ventas`
o
`http://cianbox.test/{cuenta}/api/v2/mercadolibre/ventas/lista`

**Método:** `GET`

**Parámetros:**

| Parámetro | Requerido | Descripción |
|---|---|---|
| access_token | SI | Token de acceso válido |
| id_usuario_externo | NO | id del usuario que provee MercadoLibre |
| id_venta_ml | NO | Filtra por id de venta traído de ML |
| id_user_ml | NO | Filtra por id de la cuenta (integración) brindado por MercadoLibre |
| id_publicacion_ml | NO | Filtra por id de la publicacion ML |
| vigente | NO | Filtrar por las ventas vigentes / no vigentes |
| cancelada | NO | Filtrar por las ventas canceladas |
| limit | NO | Límite de ítems por petición |
| page | NO | Página solicitada |

**Ejemplo:**
```bash
curl -X GET 'https://cianbox.org/micuenta/api/v2/mercadolibre/ventas?access_token=CBX_AT-TcIHdWOvdpIMNsXG...'
```

**Respuesta:**
```json
{
    "status": "ok",
    "body": [
        {
            "id": 25422,
            "id_venta_ml": "2000009999999999",
            "detalle": { ... },
            ...
        }
    ]
}
```
