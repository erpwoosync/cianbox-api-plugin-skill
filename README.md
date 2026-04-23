# cianbox-api plugin

Plugin de [Claude Code](https://docs.claude.com/claude-code) que expone una skill de referencia completa de la **API REST v2 de [Cianbox](https://cianbox.org)** (plataforma argentina de gestion comercial y e-commerce).

Una vez instalado, Claude puede generar integraciones, scripts y servidores MCP para Cianbox con conocimiento experto de la API: productos, clientes, pedidos, ventas, stock, remitos internos, MercadoLibre, webhooks y autenticacion.

## Instalacion

En Claude Code:

```bash
/plugin marketplace add https://github.com/erpwoosync/cianbox-api-plugin-skill
/plugin install cianbox-api@cianbox-api-plugin-skill
```

Actualizaciones posteriores:

```bash
/plugin update cianbox-api
```

## Que incluye

Skill `cianbox-api` con:

- **SKILL.md** — guia de alto nivel: modulos disponibles, flujo de autenticacion, patrones comunes (paginacion, filtros por ID multiple, estructura de respuesta), recomendaciones para generar codigo o armar un servidor MCP.
- **references/api-reference.md** — referencia detallada de cada endpoint con parametros, ejemplos curl y respuestas JSON.

### Modulos cubiertos

| Modulo | Endpoints |
|---|---|
| Autenticacion | `auth/credentials`, `auth/refresh` |
| Productos | lista, marcas, categorias, listas de precio, temporadas, ajuste de stock |
| Clientes | lista, alta, editar, categorias de cliente (CRUD), cuenta corriente |
| Pedidos | lista, alta, eliminar, editar observaciones, estados de pedido (CRUD) |
| Ventas | lista, alta (directa/pedido/ML), recursos auxiliares, preview de percepciones |
| Stock | remitos internos: listar, crear, recibir (transferencias entre sucursales) |
| MercadoLibre | listar ventas ML con filtros especificos |
| Webhooks | lista, alta, eliminar suscripciones |
| Cotizaciones | listar cotizaciones de monedas |

## Cuando se activa

La skill se activa automaticamente cuando el usuario:

- Menciona Cianbox o `cianbox.org`.
- Pide integrarse con productos, clientes, pedidos, ventas, stock, MercadoLibre de Cianbox.
- Menciona tokens con prefijo `CBX_AT-` o `CBX_RT-`.
- Pide generar scripts o servidores MCP para Cianbox.

## Ejemplo de uso

```
Necesito un script en Python que sincronice los productos de mi cuenta Cianbox
(miempresa) con una base local. ¿Me armas el esqueleto?
```

Claude detectara Cianbox, cargara la skill y generara codigo con autenticacion, manejo de `refresh_token` y paginacion correctamente.

## Licencia

[MIT](LICENSE)

## Autor

Gabriel — [rpdigital.com.ar](https://rpdigital.com.ar)
