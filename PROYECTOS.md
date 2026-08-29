## Plataformas en producción

El código de estos proyectos es privado. Lo que sigue es la arquitectura y las
decisiones técnicas de cada uno.

### WuMark — hub de integración con marketplaces
`FastAPI` · `SQLAlchemy 2.0 async` · `React` · `PostgreSQL 16` · `Redis` · `Docker`

Gestor central de catálogo que sincroniza productos, stock y órdenes contra
varios marketplaces (Mercado Libre, Falabella y otros). Cada canal expone un
modelo de datos distinto, así que la capa de integración normaliza variantes,
SKU y atributos por canal.

Backend en FastAPI con SQLAlchemy async y migraciones Alembic; los trabajos
pesados corren en un worker `arq` sobre Redis; frontend en React con Vite.
Traefik resuelve enrutamiento y TLS.

Lo interesante está en el modelo de despliegue: **cada cliente corre su propio
VPS**. Las imágenes se publican a GHCR desde GitHub Actions y los servidores solo
hacen `pull` — no tienen el código fuente, solo un `docker-compose.yml`, su
`.env` y los volúmenes de datos. Las migraciones se aplican solas al arrancar el
backend y los volúmenes nunca se tocan, así que actualizar es idempotente y no
hay estado que reconciliar entre versiones.

### Wuplify — plataforma de pedidos para restaurantes
`PHP` · `WordPress multisite` · `MySQL` · `Redis` · `LiteSpeed`

Más de 50 sitios en una instalación multisite, cada restaurante con su tienda,
su menú, sus zonas de despacho y sus métodos de pago. Panel propio fuera de
wp-admin para que el personal del local no toque WordPress.

Incluye punto de venta con turnos de caja y arqueo, pantalla de cocina por
estaciones, control de inventario con recetas y merma, cálculo de despacho por
zona dibujada en mapa o por distancia, y programas de fidelización.

Los cuellos de botella reales fueron de infraestructura: límites de recursos del
hosting bajo carga, un Action Scheduler duplicado corriendo cada minuto, y caché
de objetos con Redis. La optimización de crons y caché fue lo que sostuvo la
plataforma, no el código de aplicación.

### WuPOS — punto de venta multi-tenant
`Laravel 13` · `Livewire` · `PWA` · `IndexedDB` · `MySQL`

POS que funciona en tablet y **abre sin internet**. La caja es una PWA instalable
con un service worker que sirve el *app shell* con estrategia
stale-while-revalidate, pero que **nunca** sirve `/api/*` desde caché: los datos
de venta y catálogo llegan frescos o fallan de forma explícita, para que la app
decida qué hacer en vez de operar sobre una respuesta vieja disfrazada de nueva.
Las ventas se encolan en IndexedDB y se sincronizan al recuperar red.

Roles separados para mesero, cajero y cocina, cada uno con su propia vista.

### Ley 21.719 — plataforma de cumplimiento normativo
`Laravel` · `Livewire` · `MySQL`

Gestión de consentimiento para la ley chilena de protección de datos personales:
registro de consentimientos con trazabilidad, atención de solicitudes de los
titulares y evidencia auditable. El requisito que mandó el diseño fue la
inmutabilidad del registro: hay que poder demostrar qué se consintió y cuándo.

### CoreHost — panel de hosting
`Laravel 12` · `Blade` · `WHM/cPanel API` · `MySQL`

Panel de cliente para un hosting: contratación, facturación, saldo a favor y
enlaces de pago, integrado con Webpay y Flow. Aprovisiona cuentas contra la API
de WHM y automatiza SSL. La parte delicada es la conciliación de pagos: los
webhooks de pasarela llegan duplicados o fuera de orden, así que todo el flujo
de acreditación es idempotente.

### WuServa — SaaS de agendamiento multi-tenant
`Laravel 13` · `Livewire 3` · `MySQL`

Motor de reservas con módulos por rubro. El primer vertical es dental e incluye
odontograma interactivo. El desafío de diseño fue el aislamiento entre
inquilinos sin duplicar la lógica de negocio por cliente.

### WuMail — email marketing multi-tenant
`Laravel 13` · `Livewire` · `MySQL`

Envío masivo con relay externo, gestión de listas, plantillas y métricas de
apertura. Sustituye a Brevo para el primer cliente. El envío corre en colas con
control de tasa para no quemar la reputación del dominio.

### Condominios — administración de comunidades
`Laravel 13` · `Livewire` · `MySQL`

Gastos comunes, cobros y comunicación con residentes, multi-tenant. Opera con
doble moneda anclada al dólar por el contexto inflacionario venezolano: los
montos se registran en la moneda de referencia y se convierten al tipo de cambio
del día de emisión, no del día de pago.
