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
git clone https://github.com/tu-usuario/reservas-ec.git
cd reservas-ec
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

---

## 🔍 Quality Gates con SonarQube (DevSecOps)

### 1. Levantar SonarQube localmente

Se incluye un `docker-compose_sonar.yml` con SonarQube + PostgreSQL + GitHub Runner self-hosted.

```bash
# 1. Crear archivo .env basado en el ejemplo
cp .env.example .env
# Editar .env con tus tokens

# 2. Iniciar los servicios
docker compose -f docker-compose_sonar.yml up -d

# 3. Acceder a SonarQube
# URL: http://localhost:9000
# Credenciales por defecto: admin / admin
# Cambia la contraseña en el primer login.
```

### 2. Configurar el Quality Gate "StrictGate"

1. Ve a **Administration > Quality Gates > Create**.
2. Importa o copia la configuración desde `qualitygate.json`.
3. Aplica el Quality Gate al proyecto `app-reservas`.

| Métrica | Condición | Umbral |
|---------|-----------|--------|
| Blocker Issues | > | 0 |
| Critical Issues | > | 0 |
| Major Issues | > | 5 |
| Security Hotspots Reviewed | < | 100% |
| Coverage | < | 80% |
| Duplicated Lines (%) | > | 3% |
| Technical Debt Ratio | > | 2.5% |
| Cyclomatic Complexity (total) | > | 50 |
| Cognitive Complexity (total) | > | 30 |

### 3. Ejecutar análisis de manera manual

```bash
# Instalar sonar-scanner globalmente (si no lo tienes)
npm install -g sonar-scanner

# Ejecutar análisis (desde la raíz del proyecto)
sonar-scanner \
  -Dsonar.projectKey=app-reservas \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=squ_691f16133c4f8c9a4c81cd00d03ee0c214152e0b
```

### 4. Configurar GitHub Runner local

1. Ve a tu repositorio en GitHub: **Settings > Actions > Runners > New self-hosted runner**.
2. Copia el **RUNNER_TOKEN** que te proporciona GitHub.
3. Pégalo en tu archivo `.env` en la variable `RUNNER_TOKEN`.
4. Reinicia el contenedor del runner:
   ```bash
   docker compose -f docker-compose_sonar.yml restart github-runner
   ```
5. Verifica en GitHub que el runner aparece como **online**.

### 5. Configurar Secrets en GitHub

Ve a **Settings > Secrets and variables > Actions > New repository secret** y añade:

| Secret | Valor |
|--------|-------|
| `SONAR_TOKEN` | `squ_691f16133c4f8c9a4c81cd00d03ee0c214152e0b` |
| `SONAR_HOST_URL` | `http://sonarqube:9000` (desde el runner Docker) |
| `TELEGRAM_BOT_TOKEN` | `8845807646:AAG8LU1ywjIwd9eiIm83v6mC8KCz2eG-mDI` |
| `TELEGRAM_CHAT_ID` | Obténlo con el script de abajo |

> ⚠️ **Nunca subas tokens o credenciales al repositorio.**

### 6. Configurar bot de Telegram y obtener Chat ID

1. En Telegram, busca **@BotFather** y crea un bot (`/newbot`). Nombre sugerido: `Dev Bot`.
2. Guarda el token HTTP proporcionado.
3. Crea un grupo de Telegram e invita al bot.
4. Envía un mensaje en el grupo.
5. Ejecuta el siguiente script para obtener el Chat ID:

```bash
# Opción 1: Con Node.js
node tools/get-telegram-chatid.js

# Opción 2: Manualmente desde el navegador
# https://api.telegram.org/bot8845807646:AAG8LU1ywjIwd9eiIm83v6mC8KCz2eG-mDI/getUpdates
```

6. Copia el **Chat ID** (generalmente un número negativo como `-1001234567890`) y guárdalo como secret `TELEGRAM_CHAT_ID` en GitHub.

### 7. Workflows de GitHub Actions

| Workflow | Archivo | Descripción |
|----------|---------|-------------|
| SonarQube | `.github/workflows/sonarqube.yml` | Análisis de calidad en push/PR usando runner self-hosted |
| Telegram Notify | `.github/workflows/telegram-notify.yml` | Notificación de commits al grupo de Telegram |

---

### Roles del equipo

- **Líder de calidad:** Configura SonarQube y define los *Quality Gates*.
- **DevOps:** Encargado de los pipelines CI/CD (workflows) y la integración con Telegram.
- **Desarrolladores:** Corrigen el código para cumplir los umbrales de calidad establecidos.
