---
name: the-center-dev
description: Guía de desarrollo para el proyecto The Center (e-commerce Flask + React). Usar al implementar features, auth, API, estilos o cuando se pregunte por convenciones del proyecto.
---

# The Center - Desarrollo

## Stack

- **Backend:** Flask, SQLAlchemy, PostgreSQL, CORS. Puerto 5000.
- **Frontend:** React (Vite), React Router, Axios. Puerto 5173. CSS plano (sin Tailwind).

## Estructura clave

- `backend/app.py`: rutas y configuración. `backend/models.py`: modelos (User, Product). `backend/extensions.py`: instancia `db`.
- `frontend/src/App.jsx`: rutas. `frontend/src/api.js`: cliente Axios. `frontend/src/pages/` y `frontend/src/components/`.

## Autenticación

- Backend: modelo User (email, password_hash). Registrar con hash; login devuelve JWT. Secret en `JWT_SECRET` (.env).
- Frontend: AuthContext con user, token, login(), register(), logout(). Token en localStorage; cabecera Authorization en peticiones.

## CSS

- Global: `index.css` (variables, reset, body). Por componente/página: archivo propio (ej. `Login.css`) importado en el JSX. Convención de clases: `.page-*`, `.layout-*`, `.auth-form`.

## Reglas del proyecto

- Ver `.cursor/rules/`: convenciones generales, backend Flask y frontend React. Respetar idioma (código en inglés, comentarios/docs en español) y no commitear `.env`.
