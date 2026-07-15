# 📆 ReservasEC

**ReservasEC** es una plataforma fullstack de gestión de reservas desarrollada con una arquitectura de microservicios. Permite a los usuarios registrarse, iniciar sesión, gestionar su perfil, crear y cancelar reservas, y recibir notificaciones. El sistema está dockerizado para facilitar el despliegue local.

## 🚀 Tecnologías principales

- **Frontend:** Next.js + Tailwind CSS
- **Backend (Microservicios):**
  - Auth Service (Node.js + Express)
  - Booking Service (Node.js + Express)
  - User Service (Node.js + Express)
  - Notification Service (Node.js + Express + Nodemailer)
- **Base de datos:** MongoDB
- **Autenticación:** JSON Web Tokens (JWT)
- **Contenedores:** Docker + Docker Compose

---

## 📁 Estructura de carpetas

```plaintext
/reservas-ec
├── frontend/             # Next.js App
├── auth-service/         # Servicio de autenticación
├── user-service/         # Servicio de usuarios
├── booking-service/      # Servicio de reservas
├── notification-service/ # Servicio de notificaciones por email
└── docker-compose.yml    # Orquestación de todos los servicios
```

---

## ⚙️ Configuración del entorno

### 1. Clonar el repositorio

```bash
git clone https://github.com/AlexDaniel593/app-reservas-sast-dast.git
cd app-reservas-sast-dast
```

### 2. Variables de entorno

🔐 Frontend (frontend/.env.production.local)

```bash
NEXT_PUBLIC_API_URL=/api/auth
NEXT_PUBLIC_BOOKING_URL=/api/bookings
NEXT_PUBLIC_USER_URL=/api/users
```

🔐 Backend .env (cada microservicio)
Ejemplo para auth-service:

```bash
PORT=4000
MONGO_URI=mongodb://mongo:27017/auth-db
JWT_SECRET=supersecretkey
```

Repite para los demás servicios cambiando PORT, MONGO_URI y usando el mismo JWT_SECRET.

### 3. 🐳 Uso con Docker

1. Construir los contenedores

```bash
docker-compose build
```

3. Levantar los servicios

```bash
docker-compose up
```

La app estará disponible en http://localhost:3000

## ✅ Funcionalidades principales

- Registro e inicio de sesión de usuarios

- Perfil editable

- Creación y cancelación de reservas

- Historial de reservas activas y canceladas

- Límite de 5 reservas canceladas visibles

- Notificaciones por email (reserva y cancelación)

- Gestión de microservicios independientes

- Quality Gates con SonarQube para control de calidad de código

- Notificaciones automáticas vía Telegram sobre commits y análisis de calidad

### 4. Configurar Secrets en GitHub

Ve a **Settings > Secrets and variables > Actions > New repository secret** y añade:

| Secret | Valor |
|--------|-------|
| `TELEGRAM_BOT_TOKEN` | `bot_token` |
| `TELEGRAM_CHAT_ID` | `chat_id` |


### Roles del equipo

- **Líder de calidad:** Configura SonarQube y define los *Quality Gates*.
- **DevOps:** Encargado de los pipelines CI/CD (workflows) y la integración con Telegram.
- **Desarrolladores:** Corrigen el código para cumplir los umbrales de calidad establecidos.
