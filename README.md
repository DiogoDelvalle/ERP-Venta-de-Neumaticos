# ERP – Venta de Neumáticos

Sistema ERP web para una empresa de **venta de neumáticos**, desarrollado en equipo como proyecto académico en FIUNI. Centraliza en una sola aplicación las áreas administrativas del negocio: **compras, ventas, recursos humanos, contabilidad y tesorería**, con acceso diferenciado por rol.

## Qué es

Una aplicación full-stack con un frontend en React y una API REST en Node.js/Express que persiste en una base de datos **PostgreSQL gestionada con Supabase**. Cada módulo funcional tiene su propio flujo de pantallas en el frontend y su propia capa `routes → controllers → services` en el backend.

| Módulo | Qué cubre |
|---|---|
| **Compras** | Pedidos, cotizaciones de proveedores, órdenes de compra, facturas de compra, notas de crédito (devoluciones), órdenes de pago y proveedores |
| **Ventas** | Presupuestos, venta directa, facturas y notas de crédito de venta |
| **RRHH** | Gestión de personal y gestión salarial |
| **Contabilidad** | Plan de cuentas, asientos, libro diario, libro mayor y balances |
| **Tesorería** | Cuentas bancarias, movimientos, depósitos y conciliación |
| **Autenticación** | Login con JWT y control de acceso por rol (`admin`, `compras`, `ventas`, `rrhh`, `contabilidad`, `tesoreria`) |

## Tecnologías

- **Frontend:** React 18, React Router 7, Create React App
- **Backend:** Node.js, Express 5, `jsonwebtoken`, `cors`, `dotenv`
- **Base de datos:** PostgreSQL en **Supabase** (`@supabase/supabase-js`)

## Aporte de Diogo Delvalle

Diogo fue el responsable principal del **módulo de Compras**, en particular de todo el **ciclo de abastecimiento**, desde que se solicita un producto hasta que se recibe, se factura y, si hace falta, se devuelve. Realizó **26 commits propios entre el 05/05 y el 05/06/2026**, más los merges correspondientes, y trabajó tanto en el **frontend como en el backend** de cada etapa.

### El flujo de compras que construyó

```
Pedido ──▶ Cotización ──▶ Orden de compra ──▶ Carga de factura ──▶ Orden de pago
                                  │                   │
                                  └── Devolución (nota de crédito) ◀──┘
```

**1. Pedidos de compra**
- Backend completo de pedidos: `pedidos.service.js`, `pedidos.controller.js` y `pedidos.routes.js` (alta de pedido con sus detalles, listado para tabla, vista completa con estados y cotizaciones asociadas, y cambio de estado).
- Pantallas `Pedidos`, `NuevosPedidos`, `DetallePedido` y el modal `NuevoProductoModal`, con el flujo de creación de un pedido contra la base de datos y un detalle que muestra el pedido seleccionado con sus productos y cotizaciones.
- Pantalla de carga mientras se esperan los datos del backend.

**2. Cotizaciones**
- Pantallas de **carga de cotizaciones** (`CargarCotizacionModal`) y de **detalle de cotizaciones** (`DetalleCotizaciones`), donde se comparan las ofertas de los proveedores sobre un pedido.
- Ajustes en el servicio y controlador de cotizaciones para integrarlos con el resto del flujo.

**3. Órdenes de compra**
- Servicio de órdenes de compra (`ordenesCompra.service.js`): genera las órdenes **agrupadas por proveedor** a partir de las cotizaciones elegidas, y al confirmarlas pasa la cotización y el pedido a estado *confirmado*. También verifica si un pedido o una cotización ya tiene una orden asociada y crea la factura vacía (*placeholder*) que se completará al recibir la mercadería.
- Refactor de `InformacionOrden`, la pantalla de detalle de una orden de compra con sus ítems y facturas.

**4. Carga de factura y recepción de mercadería**
- Modal `CargarFacturaModal` (desarrollado íntegramente por él) y la lógica de `facturas.service.js` para confirmar la factura *placeholder* con los datos reales del proveedor (timbrado, número, fechas, importes y líneas).
- **Recepciones parciales:** si la orden se entrega por completo pasa a *completado*; si quedan productos pendientes, genera automáticamente una nueva factura *placeholder* solo con lo que falta.
- Vinculación de la factura con su orden de pago en estado de espera.

**5. Devoluciones / notas de crédito de compra**
- Servicio completo de notas de crédito de compras (`notasCredito.service.js`, 449 líneas, 100 % suyo), su controlador y sus rutas, y el modal `NotaCreditoModal`.
- Al registrar una devolución, el sistema: guarda la nota y sus detalles, **revierte el stock** en inventario, descuenta lo recibido en la orden de compra, **anula la factura** si la devolución es total, ajusta la orden de pago asociada, genera una nueva factura pendiente por lo que quedó sin entregar y **reabre la orden** si estaba completada.

**6. Órdenes de pago, facturas y proveedores**
- Ajustes en métodos de pago, órdenes de pago y la selección de facturas a pagar.
- Eliminó los datos *hardcodeados* de las pantallas de **Facturas** y **Proveedores**, conectándolas a la base de datos real.
- Varias rondas de corrección de bugs en órdenes de pago y compras.

### Componentes reutilizables y estructura

- **`Searchbar`**: barra de búsqueda de productos que desarrolló para los pedidos y que luego se reutilizó en otras partes de la aplicación (selección de proveedores, órdenes de pago).
- Nuevos iconos del sistema (`Icons`) y mejoras al componente `Lista`.
- **Estructura inicial del repositorio** (05/05/2026): subió el esqueleto de carpetas de BackEnd y FrontEnd (rutas, controladores y servicios por módulo) para ordenar el trabajo del equipo, y corrigió el ruteo de `index.js` y `App.jsx`.

### En números

Según `git blame` sobre el estado actual del repositorio, escribió la gran mayoría del código de:

| Archivo | Líneas de su autoría |
|---|---|
| `notasCredito.service.js` (compras) | 449 de 449 |
| `ordenesCompra.service.js` | 380 de 383 |
| `pedidos.service.js` | 192 de 200 |
| `pedidos.controller.js` / `pedidos.routes.js` | 82 de 82 / 14 de 14 |
| `facturas.service.js` (compras) | 331 de 506 |
| `CargarFacturaModal.jsx` | 647 de 647 |
| `DetalleCotizaciones.jsx` | 466 de 505 |
| `DetallePedido.jsx` | 447 de 484 |
| `NuevosPedidos.jsx` | 438 de 478 |
| `NuevoProductoModal.jsx` | 396 de 406 |
| `Searchbar.jsx` | 321 de 324 |
| `NotaCreditoModal.jsx` | 340 de 400 |

## Equipo

Proyecto desarrollado por Diogo Delvalle, Juan Francken, Nicole Quiñonez, Sheila Villanueva, Martin Franchini y Arturo Alonso.

| Integrante | Áreas principales |
|---|---|
| **Diogo Delvalle** | Ciclo de compras: pedidos, cotizaciones, órdenes de compra, facturación y devoluciones; componente `Searchbar` |
| Juan Francken | Autenticación, capa de servicios/API, `fetchConToken` y middleware, y backend de varios módulos (RRHH, tesorería, contabilidad, ventas) |
| Nicole Quiñonez | Pantallas iniciales de compras y módulo de contabilidad |
| Sheila Villanueva | Órdenes de compra (listado), proveedores, RRHH y tesorería (frontend) |
| Martin Franchini | Ruteo y navegación, roles, módulo de ventas y notas de crédito |
| Arturo Alonso | Presupuestos y ventas, conexión de órdenes de compra, tesorería |
