# CLAUDE.md

## Qué es
Sistema de fidelidad con tarjetas de sellos en Apple Wallet y Google Wallet para cafeterías y restaurantes. MVP.

## Fuente de verdad
- Las specs en `/specs` mandan. Lee `specs/00-producto.md` y la spec de la tarea antes de tocar código.
- Si cambia un comportamiento, actualiza la spec en el MISMO commit.
- Nada que esté en "Fuera de alcance" de `00-producto.md` sin preguntarme.
- Usa los nombres del glosario de `00-producto.md`. Specs en español; código en inglés.

## Stack
Next.js (App Router, última versión) · TypeScript estricto · Tailwind · Supabase (Postgres + Auth) · Resend (SMTP del magic link) · Vitest · Vercel

## Comandos
- `pnpm dev`: desarrollo
- `pnpm test`: tests (Vitest)
- `pnpm lint` / `pnpm typecheck`
- `pnpm build`: debe pasar antes de cada commit

## Forma de trabajar
1. Orden: spec → plan → código.
   - La spec (qué y por qué) vive en `/specs`. Si la tarea cambia un comportamiento que no está en ninguna spec, para y pregúntame antes de seguir.
   - El plan (cómo: archivos, pasos, tests) es por tarea y no se guarda. Propónmelo y espera mi OK antes de escribir código.
2. Una tarea = un cambio pequeño = un commit.
3. TDD para la lógica de dominio (sellos, cooldown, canjes, generación del pase). Tests en inglés; cada test referencia su regla: `it("R2: rejects a second stamp within 60 min")`. No hace falta TDD para la UI.
4. Lógica de dominio en `src/domain/`, sin dependencias de Next ni de Supabase.
5. Gestor de paquetes: pnpm (nunca npm ni yarn). No añadas dependencias sin preguntarme.
6. Nunca subas secretos al repo; usa `.env.local` y documenta las variables en `.env.example`.
7. Nunca escribas comentarios en el código, bajo ningún concepto. El código debe explicarse solo con buenos nombres.

## Estructura
```
specs/        specs del producto (fuente de verdad)
src/app/      rutas Next.js
src/domain/   reglas de negocio puras (testeadas)
src/lib/      clientes: supabase, wallet, email
supabase/     migraciones
```
