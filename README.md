# 🛍️ LDP Dashboard

Panel de Gestión de Productos construido con Vue 3, Vite y Supabase.

![Vue.js](https://img.shields.io/badge/Vue.js-3.4-4FC08D?style=flat&logo=vue.js&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-5.0-646CFF?style=flat&logo=vite&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Auth%20%26%20Database-3ECF8E?style=flat&logo=supabase&logoColor=white)

## ✨ Características

- 🔐 **Autenticación de Usuarios** - Registro, inicio de sesión y cierre de sesión con Supabase Auth
- 👤 **Perfiles de Usuario** - Gestión completa de perfiles con carga de avatar
- 📦 **Gestión de Productos** - Operaciones CRUD completas para productos
- 🔍 **Búsqueda y Filtros** - Búsqueda en tiempo real y filtrado por categoría/estado
- 📊 **Dashboard** - Vista general de productos limpia e intuitiva
- 🖼️ **Carga de Avatar** - Subida de foto de perfil con Supabase Storage
- 🔒 **Seguridad a Nivel de Fila** - Cada usuario solo puede acceder a sus propios datos
- 📱 **Diseño Responsive** - Funciona en dispositivos de escritorio y móviles

## 🚀 Inicio Rápido

### Requisitos Previos

- Node.js 18+ instalado
- Gestor de paquetes npm o yarn
- Cuenta de Supabase

### Instalación

1. **Clonar el repositorio**
```bash
git clone https://github.com/TU-USUARIO/nombre-repositorio.git
cd retailbiz-dashboard
```

2. **Instalar dependencias**
```bash
npm install
```

3. **Configurar variables de entorno**

Crea un archivo `.env` en la raíz del proyecto:
```env
VITE_SUPABASE_URL=tu_url_de_proyecto_supabase
VITE_SUPABASE_ANON_KEY=tu_clave_anonima_supabase
```

4. **Configurar Supabase**

Ejecuta este SQL en el Editor SQL de Supabase:

```sql
-- Crear tabla de perfiles
CREATE TABLE profiles (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  email TEXT,
  full_name TEXT,
  username TEXT UNIQUE,
  avatar_url TEXT,
  phone TEXT,
  company TEXT,
  address TEXT,
  city TEXT,
  country TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Crear tabla de productos
CREATE TABLE products (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  sku TEXT UNIQUE,
  category TEXT,
  status TEXT DEFAULT 'active',
  unit_price DECIMAL(10, 2) NOT NULL,
  quantity INTEGER DEFAULT 1,
  total DECIMAL(10, 2) GENERATED ALWAYS AS (unit_price * quantity) STORED,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  user_id UUID REFERENCES auth.users(id)
);

-- Habilitar Row Level Security
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Políticas para perfiles
CREATE POLICY "Los usuarios pueden ver su propio perfil"
  ON profiles FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Los usuarios pueden actualizar su propio perfil"
  ON profiles FOR UPDATE USING (auth.uid() = id);

CREATE POLICY "Los usuarios pueden insertar su propio perfil"
  ON profiles FOR INSERT WITH CHECK (auth.uid() = id);

-- Políticas para productos
CREATE POLICY "Los usuarios pueden ver sus propios productos"
  ON products FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Los usuarios pueden insertar sus propios productos"
  ON products FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Los usuarios pueden actualizar sus propios productos"
  ON products FOR UPDATE USING (auth.uid() = user_id);

CREATE POLICY "Los usuarios pueden eliminar sus propios productos"
  ON products FOR DELETE USING (auth.uid() = user_id);

-- Crear perfil automáticamente al registrarse
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, email, full_name)
  VALUES (NEW.id, NEW.email, NEW.raw_user_meta_data->>'full_name');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

5. **Crear Bucket de Storage**

En el Panel de Supabase:
- Ve a Storage
- Crea un nuevo bucket llamado `avatars`
- Hazlo público
- Ejecuta las políticas de storage (ver documentación)

6. **Iniciar servidor de desarrollo**
```bash
npm run dev
```

Abre [http://localhost:5173](http://localhost:5173) en tu navegador.

## 🏗️ Estructura del Proyecto

```
retailbiz-dashboard/
├── src/
│   ├── assets/          # Recursos estáticos
│   ├── components/      # Componentes de Vue
│   │   └── Navbar.vue
│   ├── views/          # Componentes de página
│   │   ├── Login.vue
│   │   ├── Register.vue
│   │   ├── Dashboard.vue
│   │   └── Profile.vue
│   ├── router/         # Configuración de Vue Router
│   │   └── index.js
│   ├── lib/            # Funciones de utilidad
│   │   └── supabase.js
│   ├── App.vue
│   ├── main.js
│   └── style.css
├── .env.example
├── .gitignore
├── index.html
├── package.json
├── vite.config.js
└── README.md
```

## 🛠️ Stack Tecnológico

- **Framework Frontend:** Vue 3 (Composition API)
- **Herramienta de Construcción:** Vite
- **Enrutador:** Vue Router 4
- **Backend/Base de Datos:** Supabase
- **Autenticación:** Supabase Auth
- **Almacenamiento:** Supabase Storage
- **Estilos:** CSS personalizado

## 🔑 Variables de Entorno

| Variable | Descripción |
|----------|-------------|
| `VITE_SUPABASE_URL` | URL de tu proyecto de Supabase |
| `VITE_SUPABASE_ANON_KEY` | Clave anónima de Supabase |

## 📦 Scripts Disponibles

```bash
# Iniciar servidor de desarrollo
npm run dev

## 👤 Autor

Brayan Gamboa - [@linkedin](https://www.linkedin.com/in/brayan-gamboa-delgado/)


## 🙏 Agradecimientos

- [Vue.js](https://vuejs.org/)
- [Vite](https://vitejs.dev/)
- [Supabase](https://supabase.com/)
- [Lucide Icons](https://lucide.dev/)