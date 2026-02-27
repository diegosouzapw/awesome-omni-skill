---
name: run-app
description: Ejecuta aplicaciones detectando automáticamente el tipo de proyecto (Next.js, Node.js, Python, Docker). Verifica dependencias, variables de entorno, puertos y base de datos. Usar cuando el usuario pida iniciar, correr, ejecutar o levantar la aplicación o servidor.
---

# Correr Aplicación

Habilidad para detectar y ejecutar aplicaciones de manera automática.

## Flujo de Ejecución

Sigue estos pasos en orden:

```
Progreso:
- [ ] Paso 1: Detectar tipo de proyecto
- [ ] Paso 2: Verificar dependencias
- [ ] Paso 3: Verificar variables de entorno
- [ ] Paso 4: Verificar puertos disponibles
- [ ] Paso 5: Iniciar base de datos (si aplica)
- [ ] Paso 6: Ejecutar la aplicación
```

---

## Paso 1: Detectar Tipo de Proyecto

Busca estos archivos en la raíz del proyecto:

| Archivo | Tipo de Proyecto |
|---------|------------------|
| `package.json` + `next.config.js` | Next.js |
| `package.json` (sin Next) | Node.js |
| `requirements.txt` o `pyproject.toml` | Python |
| `docker-compose.yml` | Docker Compose |
| `Dockerfile` (sin compose) | Docker |

**Prioridad**: Si existe `docker-compose.dev.yml`, usar Docker para desarrollo.

---

## Paso 2: Verificar Dependencias

### Node.js / Next.js

```bash
# Verificar si existe node_modules
ls node_modules 2>/dev/null || npm install
```

Si falla `npm install`, verificar:
- Node.js instalado: `node --version`
- npm instalado: `npm --version`

### Python

```bash
# Verificar entorno virtual
python -m venv venv 2>/dev/null
source venv/bin/activate  # Linux/Mac
# o: .\venv\Scripts\activate  # Windows

pip install -r requirements.txt
```

### Docker

```bash
# Verificar Docker instalado
docker --version
docker-compose --version
```

---

## Paso 3: Verificar Variables de Entorno

1. Buscar archivo `.env.example` o `.env.sample`
2. Si existe `.env.example` pero no `.env`:

```bash
cp .env.example .env
```

3. **Notificar al usuario** que debe configurar las variables en `.env`

Variables críticas comunes:
- `DATABASE_URL` - conexión a base de datos
- `API_KEY` / `SECRET_KEY` - claves de API
- `PORT` - puerto de la aplicación

---

## Paso 4: Verificar Puertos

Puertos comunes por tipo de aplicación:

| Tipo | Puerto Default |
|------|---------------|
| Next.js | 3000 |
| React (CRA) | 3000 |
| Node.js | 3000 o 8080 |
| Django | 8000 |
| FastAPI | 8000 |
| Flask | 5000 |
| PostgreSQL | 5432 |
| MySQL | 3306 |
| Redis | 6379 |

Verificar si el puerto está en uso:

```bash
# Windows
netstat -ano | findstr :3000

# Linux/Mac
lsof -i :3000
```

Si está ocupado, sugerir cerrar el proceso o usar puerto alternativo.

---

## Paso 5: Iniciar Base de Datos

### Con Docker Compose

Si existe `docker-compose.yml` o `docker-compose.dev.yml`:

```bash
# Solo base de datos
docker-compose up -d db

# O con archivo de desarrollo
docker-compose -f docker-compose.dev.yml up -d db
```

### Sin Docker

Verificar que el servicio de base de datos esté corriendo:
- PostgreSQL: `pg_isready`
- MySQL: `mysqladmin ping`
- MongoDB: `mongosh --eval "db.runCommand({ ping: 1 })"`

### Ejecutar Migraciones

Detectar herramienta de migraciones:

| Herramienta | Comando |
|-------------|---------|
| Drizzle | `npm run db:migrate` o `npx drizzle-kit migrate` |
| Prisma | `npx prisma migrate dev` |
| TypeORM | `npm run typeorm migration:run` |
| Django | `python manage.py migrate` |
| Alembic | `alembic upgrade head` |

---

## Paso 6: Ejecutar la Aplicación

### Next.js

```bash
# Desarrollo
npm run dev

# Producción
npm run build && npm start
```

### Node.js

```bash
# Buscar script en package.json
npm run dev    # o npm run start
```

### Python

```bash
# Django
python manage.py runserver

# FastAPI
uvicorn main:app --reload

# Flask
flask run
```

### Docker Compose

```bash
# Desarrollo (con archivo dev)
docker-compose -f docker-compose.dev.yml up

# Producción
docker-compose up -d
```

---

## Comandos Rápidos por Proyecto

### Este Proyecto (Next.js + Docker)

```bash
# Opción 1: Con Docker (recomendado)
docker-compose -f docker-compose.dev.yml up

# Opción 2: Sin Docker
npm install
cp .env.example .env  # si no existe .env
npm run dev
```

---

## Solución de Problemas Comunes

### Error: Puerto en uso

```bash
# Encontrar y matar proceso en Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# Linux/Mac
kill -9 $(lsof -t -i:3000)
```

### Error: Base de datos no conecta

1. Verificar que el contenedor/servicio esté corriendo
2. Verificar `DATABASE_URL` en `.env`
3. Verificar credenciales

### Error: Módulos no encontrados

```bash
# Eliminar y reinstalar
rm -rf node_modules package-lock.json
npm install
```

### Error: Permisos Docker (Linux)

```bash
sudo usermod -aG docker $USER
# Cerrar sesión y volver a entrar
```