# Mural de la boda — Marian & Borao

Página web donde los invitados entran con su nombre, suben fotos y vídeos sin límite, y todo aparece en un mural compartido. Cada invitado tiene también una pestaña "Mi perfil" con solo lo que ha subido él.

## Cómo funciona por dentro
Como GitHub Pages solo aloja archivos estáticos (no tiene servidor propio), esta página usa **Supabase** (gratis) para:
- **Base de datos (PostgreSQL)**: una tabla `posts` con quién subió cada foto, el comentario y la fecha.
- **Storage**: guarda los archivos de fotos y vídeos.

El plan gratuito de Supabase incluye 500 MB de base de datos y 1 GB de almacenamiento de archivos — de sobra para una boda.

## Paso 1 — Crear tu proyecto de Supabase (5 minutos)

1. Ve a [supabase.com](https://supabase.com) y entra con GitHub o Google.
2. Clic en **"New project"**. Ponle un nombre, por ejemplo `boda-marian-borao`, elige una contraseña para la base de datos (guárdala, aunque no la necesitarás para esta web) y una región cercana (ej. Europa).
3. Espera 1-2 minutos a que se cree el proyecto.

## Paso 2 — Crear la tabla `posts`

1. En el menú lateral, ve a **"SQL Editor"** → **"New query"**.
2. Pega esto y dale a **"Run"**:
```sql
create table posts (
  id uuid default gen_random_uuid() primary key,
  nickname text not null,
  url text not null,
  type text not null,
  caption text,
  created_at timestamp with time zone default now()
);

alter table posts enable row level security;

create policy "Cualquiera puede leer el mural"
  on posts for select
  using (true);

create policy "Cualquiera puede subir fotos"
  on posts for insert
  with check (nickname is not null and length(nickname) > 0);
```
Esto crea la tabla y deja el mural abierto a cualquiera con el enlace (como una red social simple), sin necesidad de contraseñas.

## Paso 3 — Crear el almacenamiento de fotos

1. Ve a **"Storage"** en el menú lateral → **"Create a new bucket"**.
2. Nómbralo exactamente `fotos-boda` (o cambia el nombre en el código si prefieres otro).
3. Márcalo como **"Public bucket"** (para que las fotos se puedan ver en el mural).
4. Ve a la pestaña **"Policies"** de ese bucket y añade una política que permita subir archivos a cualquiera:
```sql
create policy "Cualquiera puede subir a fotos-boda"
  on storage.objects for insert
  with check (bucket_id = 'fotos-boda');

create policy "Cualquiera puede ver fotos-boda"
  on storage.objects for select
  using (bucket_id = 'fotos-boda');
```
(Puedes pegar esto también en el SQL Editor.)

## Paso 4 — Conectar la web a tu proyecto

1. En Supabase, ve a **"Project Settings"** (engranaje) → **"API"**.
2. Copia el **"Project URL"** y la clave **"anon public"**.
3. Abre `index.html` en un editor de texto, busca esto cerca del final del archivo:
   ```js
   const SUPABASE_URL = "https://TU_PROYECTO.supabase.co";
   const SUPABASE_ANON_KEY = "TU_ANON_KEY";
   ```
4. Sustituye esos valores por los tuyos y guarda.

## Paso 5 — Publicar en GitHub Pages

1. Crea un repositorio nuevo en GitHub (público), por ejemplo `boda-marian-borao`.
2. Sube el archivo `index.html` a ese repositorio (arrastrarlo en la web de GitHub funciona).
3. Ve a **Settings → Pages** del repositorio.
4. En "Source", elige la rama `main` y la carpeta `/root`, guarda.
5. En un par de minutos tu web estará disponible en:
   `https://tu-usuario.github.io/boda-marian-borao/`

Comparte ese enlace con tus invitados (por ejemplo con un código QR en las mesas o en la invitación).

## Notas
- No hay límite de fotos ni de invitados: cada uno pone su nombre y sube lo que quiera.
- El nombre se guarda en el propio móvil del invitado, así que si vuelve a entrar no se lo vuelve a pedir (puede cambiarlo con el enlace "cambiar" junto a su nombre).
- El mural se actualiza en tiempo real: si alguien sube una foto, aparece automáticamente en el mural de los demás sin recargar la página.
- Si tras la boda quieres cerrar las subidas, borra la política "Cualquiera puede subir fotos" en el SQL Editor.
