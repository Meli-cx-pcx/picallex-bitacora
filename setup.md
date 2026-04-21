# 🚀 Guía de Deploy — Bitácora Comercial PicallEx CX

## Opción A: Solo Netlify (datos locales por navegador)

Si el equipo trabaja desde el mismo dispositivo o no necesitan compartir datos en tiempo real, podés hacer deploy directo sin Supabase.

### Pasos:
1. Creá una cuenta en [github.com](https://github.com) si no tenés
2. Creá un repositorio nuevo (ej. `picallex-bitacora`)
3. Subí los archivos: `index.html`, `calculadora-precios.html`, `netlify.toml`
4. Conectá el repo a [netlify.com](https://netlify.com) → "Add new site" → "Import from Git"
5. Netlify detecta el `netlify.toml` automáticamente → Deploy ✅

⚠️ **Limitación**: Los datos que cargás (novedades, objeciones, etc.) se guardan en el navegador de cada persona por separado. Si otra persona abre el link, no ve los mismos datos.

---

## Opción B: Netlify + Supabase (datos compartidos para todo el equipo) ✅ RECOMENDADO

Con Supabase, todos los miembros del equipo ven y editan los mismos datos en tiempo real.

### Paso 1 — Crear proyecto Supabase (gratis)

1. Ir a [supabase.com](https://supabase.com) → "Start your project" → Iniciar sesión con GitHub
2. Clic en "New project"
   - Organization: (la que te crea por defecto)
   - Name: `picallex-bitacora`
   - Database Password: (generá una fuerte y guardala)
   - Region: South America (São Paulo) → más rápido para Argentina
3. Esperá 1-2 minutos que el proyecto se cree

### Paso 2 — Crear la tabla en Supabase

1. En el panel de Supabase, ir a **SQL Editor** (ícono de código en el sidebar)
2. Pegar y ejecutar este SQL:

```sql
-- Crear tabla para guardar todos los datos de la Bitácora
CREATE TABLE bitacora_data (
  id BIGINT PRIMARY KEY DEFAULT 1,
  data JSONB NOT NULL DEFAULT '{}',
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Insertar fila inicial vacía
INSERT INTO bitacora_data (id, data) VALUES (1, '{}')
ON CONFLICT (id) DO NOTHING;

-- Permitir lectura pública (cualquiera con el link puede ver)
ALTER TABLE bitacora_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Lectura pública" 
  ON bitacora_data FOR SELECT 
  USING (true);

CREATE POLICY "Escritura pública" 
  ON bitacora_data FOR INSERT 
  WITH CHECK (true);

CREATE POLICY "Actualización pública" 
  ON bitacora_data FOR UPDATE 
  USING (true);
```

3. Clic en **Run** (▶) → debería aparecer "Success"

### Paso 3 — Obtener las credenciales

1. En Supabase, ir a **Settings** (engranaje) → **API**
2. Copiar:
   - **Project URL** → algo como `https://abcxyz.supabase.co`
   - **anon / public key** → una clave larga que empieza con `eyJ...`

### Paso 4 — Configurar el archivo index.html

Abrí `index.html` con un editor de texto (Notepad++, VS Code, etc.) y buscá estas líneas cerca del final (en la sección `<script>`):

```javascript
const SUPABASE_URL = '';  // ← Pegá tu Project URL acá
const SUPABASE_KEY = '';  // ← Pegá tu anon key acá
```

Completalas así:
```javascript
const SUPABASE_URL = 'https://tu-proyecto.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIs...resto_de_la_clave';
```

### Paso 5 — Subir a GitHub y conectar a Netlify

1. Subí todos los archivos al repositorio de GitHub
2. En Netlify, conectá el repo (o si ya está conectado, el deploy se hace automático al hacer push)
3. ¡Listo! El badge de la esquina superior derecha debería decir **✅ Sincronizado** en lugar de **⚡ Local**

---

## Flujo de trabajo para actualizar contenido

### Sin Supabase:
- Entrás a la Bitácora → Modo Admin → Editás → Los cambios se guardan automáticamente en el navegador
- Problema: solo se ven en ese navegador

### Con Supabase:
- Cualquier miembro del equipo entra → Modo Admin → Edita → Los cambios se sincronizan para todos automáticamente
- El badge muestra **✅ Sincronizado** cuando se guardó correctamente

---

## ¿Cuándo hacer un nuevo deploy?

Solo necesitás hacer un nuevo deploy cuando cambiás el **código** (el archivo `.html`). Los **datos** (novedades, objeciones, etc.) se guardan en Supabase y no requieren redeploy.

| Tipo de cambio | ¿Requiere redeploy? |
|---|---|
| Agregar/editar una novedad | ❌ No (se guarda en Supabase) |
| Agregar/editar una objeción | ❌ No |
| Cambiar el diseño o agregar una sección nueva | ✅ Sí |
| Actualizar la calculadora de precios | ✅ Sí |

---

## Estructura de archivos

```
/
├── index.html              # Bitácora Comercial (hub principal)
├── calculadora-precios.html # Calculadora de precios
├── netlify.toml            # Configuración de Netlify
└── SETUP.md                # Esta guía
```

---

## Soporte

Para cualquier cambio en el diseño, nuevas secciones o funcionalidades, contactar al equipo que desarrolló esta herramienta.

**Versión:** 4.0 · Abril 2026  
**Stack:** HTML + Vanilla JS + Supabase + Netlify
