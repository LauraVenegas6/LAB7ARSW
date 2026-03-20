# Lab P4 — BluePrints en Tiempo Real (Sockets & STOMP)

> **Repositorio:** `DECSIS-ECI/Lab_P4_BluePrints_RealTime-Sokets`  
> **Front:** React + Vite (Canvas, CRUD, y selector de tecnología RT)  
> **Backends guía (elige uno o compáralos):**
> - **Socket.IO (Node.js):** https://github.com/DECSIS-ECI/example-backend-socketio-node-/blob/main/README.md
> - **STOMP (Spring Boot):** https://github.com/DECSIS-ECI/example-backend-stopm/tree/main

## 🎯 Objetivo del laboratorio
Implementar **colaboración en tiempo real** para el caso de BluePrints. El Front consume la API CRUD de la Parte 3 (o equivalente) y habilita tiempo real usando **Socket.IO** o **STOMP**, para que múltiples clientes dibujen el mismo plano de forma simultánea.

Al finalizar, el equipo debe:
1. Integrar el Front con su **API CRUD** (listar/crear/actualizar/eliminar planos, y total de puntos por autor).
2. Conectar el Front a un backend de **tiempo real** (Socket.IO **o** STOMP) siguiendo los repos guía.
3. Demostrar **colaboración en vivo** (dos pestañas navegando el mismo plano).

---

## 🧩 Alcance y criterios funcionales
- **CRUD** (REST):
  - `GET /api/blueprints?author=:author` → lista por autor (incluye total de puntos).
  - `GET /api/blueprints/:author/:name` → puntos del plano.
  - `POST /api/blueprints` → crear.
  - `PUT /api/blueprints/:author/:name` → actualizar.
  - `DELETE /api/blueprints/:author/:name` → eliminar.
- **Tiempo real (RT)** (elige uno):
  - **Socket.IO** (rooms): `join-room`, `draw-event` → broadcast `blueprint-update`.
  - **STOMP** (topics): `@MessageMapping("/draw")` → `convertAndSend(/topic/blueprints.{author}.{name})`.
- **UI**:
  - Canvas con **dibujo por clic** (incremental).
  - Panel del autor: **tabla** de planos y **total de puntos** (`reduce`).
  - Barra de acciones: **Create / Save/Update / Delete** y **selector de tecnología** (None / Socket.IO / STOMP).
- **DX/Calidad**: código limpio, manejo de errores, README de equipo.

---

## 🏗️ Arquitectura (visión rápida)

```
React (Vite)
 ├─ HTTP (REST CRUD + estado inicial) ───────────────> Tu API (P3 / propia)
 └─ Tiempo Real (elige uno):
     ├─ Socket.IO: join-room / draw-event ──────────> Socket.IO Server (Node)
     └─ STOMP: /app/draw -> /topic/blueprints.* ────> Spring WebSocket/STOMP
```

**Convenciones recomendadas**  
- **Plano como canal/sala**: `blueprints.{author}.{name}`  
- **Payload de punto**: `{ x, y }`

---

## 📦 Repos guía (clona/consulta)
- **Socket.IO (Node.js)**: https://github.com/DECSIS-ECI/example-backend-socketio-node-/blob/main/README.md  
  - *Uso típico en el cliente:* `io(VITE_IO_BASE, { transports: ['websocket'] })`, `join-room`, `draw-event`, `blueprint-update`.
- **STOMP (Spring Boot)**: https://github.com/DECSIS-ECI/example-backend-stopm/tree/main  
  - *Uso típico en el cliente:* `@stomp/stompjs` → `client.publish('/app/draw', body)`; suscripción a `/topic/blueprints.{author}.{name}`.

---

## ⚙️ Variables de entorno (Front)
Crea `.env.local` en la raíz del proyecto **Front**:
```bash
# REST (tu backend CRUD)
VITE_API_BASE=http://localhost:8080

# Tiempo real: apunta a uno u otro según el backend que uses
VITE_IO_BASE=http://localhost:3001     # si usas Socket.IO (Node)
VITE_STOMP_BASE=http://localhost:8080  # si usas STOMP (Spring)
```
En la UI, selecciona la tecnología en el **selector RT**.

---

## 🚀 Puesta en marcha

### 1) Backend RT (elige uno)

**Opción A — Socket.IO (Node.js)**  
Sigue el README del repo guía:  
https://github.com/DECSIS-ECI/example-backend-socketio-node-/blob/main/README.md
```bash
npm i
npm run dev
# expone: http://localhost:3001
# prueba rápida del estado inicial:
curl http://localhost:3001/api/blueprints/juan/plano-1
```

**Opción B — STOMP (Spring Boot)**  
Sigue el repo guía:  
https://github.com/DECSIS-ECI/example-backend-stopm/tree/main
```bash
./mvnw spring-boot:run
# expone: http://localhost:8080
# endpoint WS (ej.): /ws-blueprints
```

### 2) Front (este repo)
```bash
npm i
npm run dev
# http://localhost:5173
```
En la interfaz: selecciona **Socket.IO** o **STOMP**, define `author` y `name`, abre **dos pestañas** y dibuja en el canvas (clics).

---

## 🔌 Protocolos de Tiempo Real (detalle mínimo)

### A) Socket.IO
- **Unirse a sala**
  ```js
  socket.emit('join-room', `blueprints.${author}.${name}`)
  ```
- **Enviar punto**
  ```js
  socket.emit('draw-event', { room, author, name, point: { x, y } })
  ```
- **Recibir actualización**
  ```js
  socket.on('blueprint-update', (upd) => { /* append points y repintar */ })
  ```

### B) STOMP
- **Publicar punto**
  ```js
  client.publish({ destination: '/app/draw', body: JSON.stringify({ author, name, point }) })
  ```
- **Suscribirse a tópico**
  ```js
  client.subscribe(`/topic/blueprints.${author}.${name}`, (msg) => { /* append points y repintar */ })
  ```

---

## 🧪 Casos de prueba mínimos
- **Estado inicial**: al seleccionar plano, el canvas carga puntos (`GET /api/blueprints/:author/:name`).  
- **Dibujo local**: clic en canvas agrega puntos y redibuja.  
- **RT multi-pestaña**: con 2 pestañas, los puntos se **replican** casi en tiempo real.  
- **CRUD**: Create/Save/Delete funcionan y refrescan la lista y el **Total** del autor.

---

## 📊 Entregables del equipo
1. Código del Front integrado con **CRUD** y **RT** (Socket.IO o STOMP).  
2. **Video corto** (≤ 90s) mostrando colaboración en vivo y operaciones CRUD.  
3. **README del equipo**: setup, endpoints usados, decisiones (rooms/tópicos), y (opcional) breve comparativa Socket.IO vs STOMP.

---

## 🧮 Rúbrica sugerida
- **Funcionalidad (40%)**: RT estable (join/broadcast), aislamiento por plano, CRUD operativo.  
- **Calidad técnica (30%)**: estructura limpia, manejo de errores, documentación clara.  
- **Observabilidad/DX (15%)**: logs útiles (conexión, eventos), health checks básicos.  
- **Análisis (15%)**: hallazgos (latencia/reconexión) y, si aplica, pros/cons Socket.IO vs STOMP.

---

## 🩺 Troubleshooting
- **Pantalla en blanco (Front)**: revisa consola; confirma `@vitejs/plugin-react` instalado y que `AppP4.jsx` esté en `src/`.  
- **No hay broadcast**: ambas pestañas deben hacer `join-room` al **mismo** plano (Socket.IO) o suscribirse al **mismo tópico** (STOMP).  
- **CORS**: en dev permite `http://localhost:5173`; en prod, **restringe orígenes**.  
- **Socket.IO no conecta**: fuerza transporte WebSocket `{ transports: ['websocket'] }`.  
- **STOMP no recibe**: verifica `brokerURL`/`webSocketFactory` y los prefijos `/app` y `/topic` en Spring.

---

## 🔐 Seguridad (mínimos)
- Validación de payloads (p. ej., zod/joi).  
- Restricción de orígenes en prod.  
- Opcional: **JWT** + autorización por plano/sala.

---

## 📄 Licencia
MIT (o la definida por el curso/equipo).  

--- 
# REPORTE DE LABORATORIO
    - Laura Alejandra Venegas Piraban
    - Sergio Alejandro Idarraga Torres
---

## Descripción del Proyecto

Aplicación fullstack para gestión de BluePrints con autenticación JWT (OAuth 2.0) y
colaboración en tiempo real via WebSockets. El backend corre con Spring Boot 3 y Java 21
como Resource Server seguro, y el frontend está construido con React 18 + Redux Toolkit.

---
# Video
El video se encuentra en /assets/LAB7-ARSW.mp4.
---

## Repositorios

| Capa | Repositorio |
|------|-------------|
| Backend | https://github.com/ALEJO21005/Lab_P2_BluePrints_Java21_API_Security_JWT.git |
| Frontend | https://github.com/ALEJO21005/LAB6_ARSW.git |

---

## Tecnologías

| Capa | Stack |
|------|-------|
| Backend | Spring Boot 3, Java 21, PostgreSQL |
| Frontend | React 18 (Vite), Redux Toolkit, React Router |
| Tiempo real | STOMP (@stomp/stompjs) + SockJS |
| HTTP | Axios con interceptores JWT |
| Testing | Vitest + Testing Library |

---

## Setup

### Requisitos

- Java 21 (JDK)
- Maven 3.9+
- PostgreSQL (la config está en el pom.xml)
- Node.js 18+
- Git

### Backend
```bash
git clone https://github.com/ALEJO21005/Lab_P2_BluePrints_Java21_API_Security_JWT.git
cd Lab_P2_BluePrints_Java21_API_Security_JWT

mvn spring-boot:run

# Verificar que esté corriendo
curl http://localhost:8080/actuator/health
```

### Frontend
```bash
git clone https://github.com/ALEJO21005/LAB6_ARSW.git
cd LAB6_ARSW

npm install
npm run dev       # Desarrollo en puerto 5173
npm run build     # Build de producción
npm test          # Pruebas con Vitest
npm run test:ui   # UI de pruebas
```

### Variables de entorno (.env)
```env
VITE_API_BASE_URL=http://localhost:8080
VITE_WS_BASE_URL=http://localhost:8080/ws
VITE_USE_MOCK=false    # true = mock local, false = API real
```

---

## Autenticación

**POST** /auth/login — devuelve el token JWT necesario para todo lo demás.

Credenciales de prueba:
- Usuario: student
- Contraseña: student123

El frontend maneja el token automáticamente a través de interceptores en Axios.
Si el servidor responde con 401, redirige al login sin intervención manual.

---

## Endpoints

### REST (CRUD)

Todos requieren el header:
```
Authorization: Bearer <token>
```

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | /api/v1/blueprints | Lista general |
| GET | /api/v1/blueprints/{author} | Blueprints por autor |
| GET | /api/v1/blueprints/{author}/{name} | Blueprint específico |
| POST | /api/v1/blueprints | Crear blueprint |
| PUT | /api/v1/blueprints/{author}/{name}/points | Agregar punto |
| DELETE | /api/v1/blueprints/{author}/{name} | Eliminar blueprint |
| DELETE | /api/v1/blueprints/{author} | Eliminar todos los de un autor |
| DELETE | /api/v1/blueprints/{author}/{name}/points/{x}/{y} | Eliminar punto |

Body para POST:
```json
{ "name": "Mi Blueprint" }
```

### WebSocket (tiempo real)

Conexión: ws://localhost:8080/ws (SockJS + STOMP)

Tópicos:
- /topic/blueprints — cambios globales
- /topic/blueprints/{author}/{name} — escucha un blueprint puntual

Al cambiar de blueprint activo en el frontend, la suscripción cambia automáticamente.

### Swagger

Disponible en http://localhost:8080/swagger-ui/index.html.

Para probar endpoints protegidos: botón "Authorize" → pegar Bearer <token>.

---

## Cómo se integran CRUD y tiempo real

Las operaciones CRUD se despachan como async thunks desde Redux y llegan al backend
via Axios. Cuando el backend procesa un cambio, emite una notificación por WebSocket
a todos los suscriptores del tópico correspondiente. El frontend actualiza el estado
global sin necesidad de hacer un nuevo fetch.
```
# Flujo REST
Client → POST /auth/login → JWT Token
Client → /api/v1/blueprints (Bearer token) → Data

# Flujo WebSocket
Client → Connect /ws → STOMP Session
Service Layer → Operación → Notificación WebSocket
Todos los suscriptores → Reciben la actualización en tiempo real
```

---

## Por qué STOMP y no Socket.IO

La razón principal es que Spring Boot tiene soporte nativo para STOMP. Eso nos evitó
configuración extra y nos dio pub/sub con tópicos, autenticación JWT directa en los
headers de conexión y reconexión automática sin código adicional. Socket.IO requiere
más trabajo de integración del lado del servidor para lograr lo mismo.

---

## Estructura del Proyecto

### Backend
```
src/main/java/co/edu/eci/blueprints/
├── auth/
│   └── AuthController.java                    # Login y emisión de JWT
├── security/
│   ├── SecurityConfig.java                    # Configuración de seguridad
│   ├── InMemoryUserService.java               # Usuarios de prueba
│   ├── JwtKeyProvider.java                    # Generación de llaves RSA
│   └── RsaKeyProperties.java                  # Config JWT
├── api/
│   └── BlueprintController.java               # Endpoints REST protegidos
├── controllers/
│   └── BlueprintWebSocketController.java      # Handlers de WebSocket
├── services/
│   └── BlueprintsServices.java                # Lógica de negocio
├── persistence/
│   ├── BlueprintJpaRepository.java            # Repositorio JPA
│   └── BlueprintPersistence.java              # Interface de persistencia
├── dto/
│   ├── request/ & response/                   # DTOs para REST
│   └── websocket/                             # DTOs para WebSocket
├── config/
│   ├── WebSocketConfig.java                   # Config WebSocket/STOMP
│   └── OpenApiConfig.java                     # Config Swagger
├── filters/
│   └── BlueprintsFilter.java                  # Filtros de procesamiento
└── model/
    ├── Blueprint.java                         # Entidad JPA principal
    └── Point.java                             # Coordenadas
```

### Frontend
```
src/
├── components/
│   ├── WebSocketManager.jsx      # Manager de conexión WebSocket/STOMP
│   ├── BlueprintCanvas.jsx       # Canvas interactivo para dibujo
│   ├── BlueprintForm.jsx         # Formulario CRUD de creación
│   └── PrivateRoute.jsx          # Rutas protegidas con JWT
├── pages/
│   ├── BlueprintsPage.jsx        # Página principal con CRUD completo
│   └── BlueprintDetailPage.jsx   # Vista detalle individual
├── features/blueprints/
│   └── blueprintsSlice.js        # Redux: acciones async + WebSocket state
└── services/
    ├── websocketService.js       # Servicio STOMP WebSocket
    ├── blueprintsService.js      # Switch Mock/Real API
    ├── apimock.js                # Datos simulados
    └── apiclient-service.js      # Servicio API real
```

