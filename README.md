# 📆 ReservasEC

**ReservasEC** es una plataforma fullstack de gestión de reservas desarrollada con una arquitectura de microservicios. Permite a los usuarios registrarse, iniciar sesión, gestionar su perfil, crear y cancelar reservas, y recibir notificaciones. El sistema está dockerizado para facilitar el despliegue local.

**Link Telegram**

[Link invitacion al grupo de telegram](https://t.me/+_X2qcq3m3fRiOWYx)

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

## Análisis de seguridad en el pipeline

El pipeline CI/CD ejecuta dos análisis de seguridad automatizados en cada push/PR a `main`, `test` y `dev`, con notificaciones a Telegram.

### SAST — SonarQube + Trivy

Workflow: [`sonarqube-sast.yml`](.github/workflows/sonarqube-sast.yml)

**Infraestructura:**
- Servicio SonarQube 10 Community en contenedor Docker (`sonarqube:10-community`), expuesto en puerto `9000`, con health-check vía `/api/system/status` (hasta 20 reintentos cada 15s).
- Timeout global de 45 minutos.

**Calidad — Quality Gate dinámico:**
1. Genera un token de análisis vía `POST /api/user_tokens/generate` con credenciales `admin:admin`.
2. Crea un Quality Gate con nombre único (`SAST-Standard-<timestamp>`) y le asigna condiciones:
   - `new_bugs` < 1
   - `new_vulnerabilities` < 1
   - `new_security_hotspots_reviewed` = 100%
   - `new_code_smells` < 10 (warning en 5)
   - `new_duplicated_lines_density` < 10% (warning en 5%)
   - `new_maintainability_rating` ≤ B (2)
   - `new_reliability_rating` ≤ B (2)
   - `new_security_rating` ≤ B (2)
3. Elimina condiciones previas y establece el Quality Gate como default.
4. El escáner espera el resultado (`sonar.qualitygate.wait=true`, timeout 300s).

**Cobertura:**
- Tests con Jest en cada microservicio (`auth-service`, `booking-service`, `notification-service`, `user-service`) y `frontend`, generando reportes `lcov.info` (con `--coverage --forceExit`). Si no hay tests, se crea un archivo vacío para evitar errores del escáner.

**Escaneo de dependencias — Trivy:**
- `aquasecurity/trivy-action@master` con `scan-type: fs`, escaneando todo el repositorio.
- Severidades: CRITICAL, HIGH, MEDIUM, LOW.
- Salida en JSON (`trivy-report.json`), subido como artefacto por 7 días.
- El resumen se parsea con `jq` contando vulnerabilidades por severidad.

**Métricas extraídas post-scan:**
- Consulta a `GET /api/measures/component` con métricas: `bugs`, `vulnerabilities`, `code_smells`, `duplicated_lines_density`, `ncloc`, `security_rating`, `reliability_rating`, `sqale_rating`.
- Consulta a `GET /api/qualitygates/project_status` para estado del Quality Gate.
- Ratings numéricos (1-5) convertidos a letras (A-E).

**Reportes:**
- Step Summary de GitHub Actions con tabla de métricas, ratings y vulnerabilidades Trivy.
- Notificación a Telegram vía `appleboy/telegram-action` con formato HTML, incluyendo enlace al dashboard de SonarQube y a los logs de la acción.

### DAST — OWASP ZAP

Workflow: [`owasp-zap-dast.yml`](.github/workflows/owasp-zap-dast.yml)

**Infraestructura:**
- Se levanta toda la aplicación con `docker compose up -d --build`.
- Espera activa hasta 60 intentos (5s c/u) verificando que `http://localhost:4000/api/login` y `http://localhost:3000` respondan (código HTTP distinto de 000).
- Timeout global de 45 minutos.

**Preparación de autenticación:**
1. Registra usuario `dast@test.com` / `Dast123!` vía `POST /api/register`.
2. Inicia sesión con `POST /api/login` y extrae el JWT de la respuesta con `jq -r '.token'`.
3. Almacena el token en `$GITHUB_ENV` como `AUTH_TOKEN`.

**Escaneo — Frontend + APIs (autenticado):**
- Imagen: `ghcr.io/zaproxy/zaproxy:stable`.
- Comando: `zap-full-scan.py` (escaneo completo pasivo + activo).
- Target: `http://localhost:3000`.
- Network: `host` para alcanzar los servicios locales.
- Parámetros:
  - `-m 30`: tiempo máximo de escaneo 30 minutos.
  - `-a`: modo ataque (activo).
  - Profundidad de crawleo AJAX: `ajaxSpider.maxCrawlDepth=10`.
  - Hilos activos: `ascan.threadPerHost=5`.
  - Inyección del token Bearer vía `replacer` de ZAP: configura un reemplazo global para el header `Authorization: Bearer <token>` en todas las peticiones.
  - Modo vista: `attack`.
- Directorio de salida montado como volumen: `reports/` (mapeado a `/zap/wrk/`).

**Escaneo — Notification Service (sin autenticación):**
- Misma imagen y modo `zap-full-scan.py`.
- Target: `http://localhost:5002`.
- `-m 15`: tiempo máximo 15 minutos.
- Sin token de autenticación (servicio expuesto sin auth).

**Procesamiento de resultados:**
- Ambos escaneos ejecutados con `continue-on-error: true` para no bloquear el pipeline si ZAP encuentra problemas.
- Se parsea `dast-frontend.json` y `dast-notification.json` con `jq` contando alertas por `riskcode`:
  - `"3"` → High
  - `"2"` → Medium
  - `"1"` → Low
  - `"0"` → Informational
- Fallback a XML si no existe JSON (grep por `riskcode="3"` etc.).
- Resultados consolidados en variables de entorno.

**Reportes:**
- Los archivos `dast-frontend.{html,json,md,xml}` y `dast-notification.{html,json,md,xml}` se suben como artefacto (`dast-reports`) con retención de 30 días.
- Step Summary con tabla de vulnerabilidades por severidad, rating general (CRÍTICO/MEDIO/BAJO/INFORMATIVO), y detalle de hallazgos en notification-service.
- Notificación a Telegram con resumen de vulnerabilidades y enlace a los logs.

**Limpieza:**
- `docker compose down --remove-orphans` y eliminación de contenedores ZAP (`docker rm`).

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
