# 00 · Producto

> Fuente de verdad del producto. Ninguna otra spec puede contradecirla.
> Specs en español; todo nombre de código (entidades, tablas, campos) en inglés.

## Glosario (lenguaje del dominio)
| Concepto       | Código        | Definición |
|----------------|---------------|------------|
| Cuenta         | `account`     | Perfil del dueño. 1:1 con `auth.users` (Supabase). |
| Negocio        | `business`    | Un local. Tiene nombre y `slug` (URL pública `/c/{slug}`). |
| Programa       | `program`     | Plantilla de la tarjeta: sellos necesarios, logo, colores, textos. |
| Premio         | `reward`      | Lo que se obtiene al completar los sellos. Solo texto. |
| Cliente final  | `customer`    | Lead del negocio. NO hace login (no está en `auth.users`). |
| Pase           | `pass`        | La tarjeta de un `customer` en un `program`: sellos actuales + QR. |
| Sello          | `stamp`       | Evento: +1 sello en un pase. |
| Canje          | `redemption`  | Evento: se canjea un `reward`; el pase vuelve a 0 sellos. |

## Problema
Cafeterías y restaurantes pequeños usan tarjetas de sellos en papel:
se pierden, no viven en el móvil y el negocio no obtiene el contacto del cliente.

## Solución (MVP)
Tarjeta de sellos digital en Apple Wallet y Google Wallet, con registro del
cliente (lead) y un dashboard simple para el negocio.

## Actores
- **Dueño:** login por magic link. Ve clientes, sellos y canjes. Escanea pases
  para sellar y canjear. Una sola cuenta por negocio, sin empleados.
- **Cliente final:** se registra desde el QR del local y guarda el pase en Wallet.
- **Admin (yo):** da de alta `account`, `business`, `program` y `reward` por script.

## Flujo principal
1. Admin crea la cuenta por script → el dueño recibe el magic link.
2. Cliente escanea el QR (`/c/{slug}`) → formulario (nombre, email, teléfono,
   cumpleaños opcional, consentimiento RGPD) → se crean `customer` + `pass`
   en el programa por defecto → añade el pase a Wallet.
3. Cliente consume → el dueño escanea el pase → se crea un `stamp`.
4. Pase llega a N sellos → el dueño canjea un `reward` → se crea una
   `redemption` y el pase vuelve a 0.

## Modelo
```
auth.users ─1:1─ account
account    ─1:N─ business
business   ─1:N─ program
business   ─1:N─ customer
program    ─1:N─ reward
program    ─1:N─ pass
customer   ─1:N─ pass        (único por customer + program)
pass       ─1:N─ stamp
pass       ─1:N─ redemption  (cada una referencia su reward)
```
- MVP: 1 account = 1 business = 1 program. La UI muestra 1 reward; el modelo admite varios.
- La BD es la fuente de verdad. Apple y Google solo renderizan el `pass`.
  - Google Wallet: `program` ≈ `LoyaltyClass`, `pass` ≈ `LoyaltyObject`.
  - Apple Wallet: cada `pass` genera su `.pkpass` a partir del `program`.

## Reglas clave (detalle en specs/03)
- R1. La regla para ganar un sello es texto en la tarjeta; decide el empleado.
- R2. Máx. 1 `stamp` por `pass` cada 60 min.
- R3. Solo se puede canjear si el pase tiene ≥ N sellos.
- R4. Cada `redemption` registra el `reward` canjeado y deja el pase en 0.
- R5. Un `customer` tiene como máximo 1 `pass` por `program`.

## Fuera de alcance (MVP)
- Promociones / notificaciones desde el dashboard (las envío yo a mano)
- Puntos por importe, niveles, UI de varios premios
- Editor de tarjetas, alta self-service de negocios
- Cuentas de empleados, varios locales
- Cobros (Stripe), analíticas, integración con TPV

## Métrica de éxito
- Hito 1 (semana del 30 nov – 4 dic): producto funcional en producción y
  onboarding iniciado con 1 negocio real en Sevilla.
- Hito 2 (semana del 7 – 11 dic): ese negocio usándolo con clientes reales.

## Stack
Next.js (última) · TypeScript · Tailwind · Supabase (BD + Auth) ·
Resend (SMTP del magic link) · Vitest · Vercel
