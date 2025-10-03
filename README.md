# ViajaYa API - Sistema de Transporte Compartido

**Autor:** Santiago Romero - Ricardo Palomino

---

## Preguntas Guía

### 1. ¿Qué entiendes por "endpoint" en el contexto de una API?

Un endpoint es una URL específica en una API que representa un punto de acceso para realizar operaciones sobre un recurso determinado. Cada endpoint define una acción específica que se puede realizar (crear, leer, actualizar, eliminar) sobre un recurso mediante métodos HTTP específicos.

### 2. ¿Cuál es la diferencia entre un endpoint público y uno privado?

- **Endpoint público**: No requiere autenticación, accesible por cualquier cliente (ej: registro de usuarios, información general de la ciudad)
- **Endpoint privado**: Requiere autenticación mediante token JWT, solo usuarios autenticados pueden acceder (ej: solicitar viaje, ver historial)

### 3. ¿Qué información de un usuario consideras confidencial y no debería exponerse?

- Contraseñas (hash)
- Números de tarjetas de crédito completos
- Ubicación en tiempo real (solo durante viajes activos)
- Información financiera detallada
- Datos de contacto personal (teléfono, email) - solo para usuarios autorizados

### 4. ¿Por qué es importante definir bien los métodos HTTP (GET, POST, PUT/PATCH, DELETE) en cada endpoint?

Los métodos HTTP definen la semántica de la operación:
- **GET**: Operaciones de lectura (idempotente)
- **POST**: Creación de recursos
- **PUT/PATCH**: Actualización de recursos
- **DELETE**: Eliminación de recursos

Esto garantiza consistencia, facilita el cacheo y mejora la comprensión de la API.

### 5. ¿Qué tipo de información requiere autenticación en este sistema?

- Solicitud y gestión de viajes
- Acceso a historial personal
- Gestión de perfil de usuario
- Procesamiento de pagos
- Calificaciones y reseñas
- Configuraciones de cuenta

### 6. ¿Cómo manejarías la seguridad de la ubicación de conductores y pasajeros?

- Encriptación de coordenadas en tránsito
- Acceso limitado solo durante viajes activos
- Anonimización de ubicaciones históricas
- Permisos granulares por rol y contexto
- Logs de acceso para auditoría

### 7. ¿Qué pasaría si un viaje es solicitado y no hay conductores disponibles? ¿Cómo debería responder la API?

La API debería:
- Retornar estado "NO_DRIVERS_AVAILABLE"
- Sugerir horarios alternativos
- Ofrecer opciones de transporte público
- Mantener la solicitud en cola por tiempo limitado
- Notificar cuando haya conductores disponibles

### 8. ¿Cómo identificarías los recursos principales de esta aplicación?

Los recursos principales son entidades que representan conceptos del dominio:
- **users**: Usuarios del sistema
- **rides**: Viajes/solicitudes
- **payments**: Transacciones de pago
- **ratings**: Calificaciones y reseñas
- **vehicles**: Vehículos de conductores
- **locations**: Ubicaciones y zonas

### 9. ¿Qué ventajas tendría versionar la API (por ejemplo, `/v1/...`) desde el inicio?

- **Compatibilidad**: Permite evolución sin romper clientes existentes
- **Migración gradual**: Facilita transiciones entre versiones
- **Documentación clara**: Separa funcionalidades por versión
- **Testing**: Permite probar nuevas versiones en paralelo
- **Rollback**: Facilita revertir cambios problemáticos

### 10. ¿Por qué es importante documentar las respuestas de error y no solo las exitosas?

- **Debugging**: Facilita identificar problemas específicos
- **UX**: Permite mostrar mensajes de error claros al usuario
- **Desarrollo**: Guía a los desarrolladores en el manejo de casos edge
- **Monitoreo**: Permite categorizar y analizar errores
- **SLA**: Define expectativas claras de comportamiento

---

## Roles y Permisos

### Pasajero
- Registrarse y autenticarse
- Solicitar viajes
- Cancelar viajes (con penalización según tiempo)
- Calificar conductores
- Gestionar perfil personal
- Ver historial de viajes
- Gestionar métodos de pago
- Reportar problemas

### Conductor
- Registrarse como conductor (con validaciones adicionales)
- Autenticarse
- Actualizar estado (disponible/ocupado)
- Aceptar/rechazar solicitudes de viaje
- Actualizar ubicación en tiempo real
- Finalizar viajes
- Calificar pasajeros
- Gestionar perfil y vehículo
- Ver historial de viajes y ganancias
- Gestionar métodos de pago

### Administrador
- Gestionar usuarios (activar/desactivar cuentas)
- Ver estadísticas generales
- Gestionar tarifas dinámicas
- Resolver disputas
- Gestionar zonas de servicio
- Ver logs de seguridad
- Gestionar promociones
- Moderar calificaciones

---

## Recursos Principales

1. **users** - Gestión de usuarios (pasajeros, conductores, administradores)
2. **rides** - Solicitudes y gestión de viajes
3. **payments** - Procesamiento y historial de pagos
4. **ratings** - Sistema de calificaciones bidireccional
5. **vehicles** - Información de vehículos de conductores
6. **locations** - Ubicaciones, zonas de servicio y tarifas
7. **notifications** - Sistema de notificaciones
8. **promotions** - Códigos promocionales y descuentos
9. **reports** - Reportes y estadísticas
10. **zones** - Zonas de servicio y restricciones

---

## Tabla de Endpoints

### 🔐 Autenticación y Autorización

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `POST` | `/v1/auth/register` | Registro de nuevo usuario | `Body:` email, password, name, phone, role | 🌐 Público |
| `POST` | `/v1/auth/login` | Autenticación de usuario | `Body:` email, password | 🌐 Público |
| `POST` | `/v1/auth/refresh` | Renovar token de acceso | `Body:` refresh_token | 🌐 Público |

### 👤 Gestión de Usuarios

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `GET` | `/v1/users/profile` | Obtener perfil del usuario autenticado | `Headers:` Authorization | 🔑 Token |
| `PUT` | `/v1/users/profile` | Actualizar perfil del usuario | `Body:` name, phone, preferences | 🔑 Token |
| `POST` | `/v1/users/upload-document` | Subir documentos (conductores) | `Body:` document_type, file | 🚗 Conductor |

### 🚗 Gestión de Viajes

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `GET` | `/v1/rides/nearby-drivers` | Buscar conductores cercanos | `Query:` lat, lng, radius | 🔑 Token |
| `POST` | `/v1/rides/request` | Solicitar un viaje | `Body:` pickup_location, destination, vehicle_type | 👤 Pasajero |
| `GET` | `/v1/rides/{ride_id}` | Obtener detalles de un viaje | `Path:` ride_id | 🔑 Token |
| `PUT` | `/v1/rides/{ride_id}/accept` | Aceptar solicitud de viaje | `Path:` ride_id | 🚗 Conductor |
| `PUT` | `/v1/rides/{ride_id}/reject` | Rechazar solicitud de viaje | `Path:` ride_id, `Body:` reason | 🚗 Conductor |
| `PUT` | `/v1/rides/{ride_id}/start` | Iniciar viaje | `Path:` ride_id | 🚗 Conductor |
| `PUT` | `/v1/rides/{ride_id}/complete` | Finalizar viaje | `Path:` ride_id, `Body:` final_location | 🚗 Conductor |
| `PUT` | `/v1/rides/{ride_id}/cancel` | Cancelar viaje | `Path:` ride_id, `Body:` reason | 🔑 Token |
| `GET` | `/v1/rides/history` | Historial de viajes del usuario | `Query:` page, limit, status | 🔑 Token |

### 💳 Gestión de Pagos

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `POST` | `/v1/payments/methods` | Agregar método de pago | `Body:` type, card_number, expiry, cvv | 🔑 Token |
| `GET` | `/v1/payments/methods` | Listar métodos de pago | - | 🔑 Token |
| `DELETE` | `/v1/payments/methods/{method_id}` | Eliminar método de pago | `Path:` method_id | 🔑 Token |
| `POST` | `/v1/payments/process` | Procesar pago de viaje | `Body:` ride_id, payment_method_id | 🔑 Token |
| `GET` | `/v1/payments/history` | Historial de pagos | `Query:` page, limit, date_from, date_to | 🔑 Token |

### ⭐ Sistema de Calificaciones

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `POST` | `/v1/ratings` | Calificar viaje | `Body:` ride_id, rating, comment, type | 🔑 Token |
| `GET` | `/v1/ratings/{user_id}` | Ver calificaciones de usuario | `Path:` user_id | 🔑 Token |

### 🚙 Gestión de Conductores

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `PUT` | `/v1/drivers/status` | Actualizar estado del conductor | `Body:` status, location | 🚗 Conductor |
| `GET` | `/v1/drivers/earnings` | Ver ganancias del conductor | `Query:` date_from, date_to | 🚗 Conductor |

### 🚗 Gestión de Vehículos

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `POST` | `/v1/vehicles` | Registrar vehículo | `Body:` make, model, year, plate, color | 🚗 Conductor |
| `PUT` | `/v1/vehicles/{vehicle_id}` | Actualizar información del vehículo | `Path:` vehicle_id, `Body:` vehicle_data | 🚗 Conductor |

### 👨‍💼 Panel de Administración

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `GET` | `/v1/admin/users` | Listar usuarios (admin) | `Query:` role, status, page, limit | 👨‍💼 Admin |
| `PUT` | `/v1/admin/users/{user_id}/status` | Cambiar estado de usuario | `Path:` user_id, `Body:` status | 👨‍💼 Admin |
| `GET` | `/v1/admin/statistics` | Estadísticas generales | `Query:` date_from, date_to | 👨‍💼 Admin |
| `POST` | `/v1/admin/promotions` | Crear promoción | `Body:` code, discount, expiry, conditions | 👨‍💼 Admin |

### 📍 Ubicaciones y Notificaciones

| Método | Endpoint | Descripción | Parámetros | Autenticación |
|:------:|----------|-------------|------------|:-------------:|
| `GET` | `/v1/locations/zones` | Obtener zonas de servicio | `Query:` city | 🌐 Público |
| `POST` | `/v1/notifications/send` | Enviar notificación push | `Body:` user_id, title, message, type | 🔑 Token |
| `GET` | `/v1/notifications` | Obtener notificaciones del usuario | `Query:` page, limit, read | 🔑 Token |

**Leyenda de Autenticación:**
- 🌐 **Público**: No requiere autenticación
- 🔑 **Token**: Requiere token JWT válido
- 👤 **Pasajero**: Requiere token de usuario con rol pasajero
- 🚗 **Conductor**: Requiere token de usuario con rol conductor
- 👨‍💼 **Admin**: Requiere token de usuario con rol administrador

---

## Flujos de Uso

### 🚗 Flujo 1: Solicitud, Aceptación y Finalización de Viaje

**Descripción:** Flujo completo desde la solicitud de un viaje hasta su finalización y pago.

#### Paso 1: Solicitud de Viaje
**👤 Pasajero** solicita un viaje

```http
POST /v1/rides/request
Content-Type: application/json
Authorization: Bearer <token>

{
  "pickup_location": {
    "lat": 40.7128,
    "lng": -74.0060,
    "address": "123 Main St, New York"
  },
  "destination": {
    "lat": 40.7589,
    "lng": -73.9851,
    "address": "456 Broadway, New York"
  },
  "vehicle_type": "standard"
}
```

**Respuesta:**
```json
{
  "ride_id": "ride_123",
  "status": "searching",
  "estimated_fare": 15.50,
  "estimated_wait": 5,
  "estimated_duration": 18
}
```

#### Paso 2: Búsqueda de Conductores
**🔍 Sistema** busca conductores cercanos

```http
GET /v1/rides/nearby-drivers?lat=40.7128&lng=-74.0060&radius=2
Authorization: Bearer <token>
```

**Respuesta:**
```json
{
  "drivers": [
    {
      "id": "driver_456",
      "name": "Juan Pérez",
      "rating": 4.8,
      "distance": 0.8,
      "eta": 3,
      "vehicle": {
        "make": "Toyota",
        "model": "Camry",
        "color": "White",
        "plate": "ABC-123"
      }
    }
  ]
}
```

#### Paso 3: Aceptación del Viaje
**🚗 Conductor** acepta la solicitud

```http
PUT /v1/rides/ride_123/accept
Authorization: Bearer <driver_token>
```

**Respuesta:**
```json
{
  "status": "accepted",
  "driver_info": {
    "id": "driver_456",
    "name": "Juan Pérez",
    "phone": "+1234567890",
    "rating": 4.8
  },
  "estimated_arrival": "3 min",
  "driver_location": {
    "lat": 40.7130,
    "lng": -74.0058
  }
}
```

#### Paso 4: Inicio del Viaje
**🚗 Conductor** inicia el viaje

```http
PUT /v1/rides/ride_123/start
Authorization: Bearer <driver_token>
```

**Respuesta:**
```json
{
  "status": "in_progress",
  "start_time": "2024-10-03T10:30:00Z",
  "pickup_location": {
    "lat": 40.7128,
    "lng": -74.0060,
    "address": "123 Main St, New York"
  }
}
```

#### Paso 5: Finalización del Viaje
**🚗 Conductor** finaliza el viaje

```http
PUT /v1/rides/ride_123/complete
Content-Type: application/json
Authorization: Bearer <driver_token>

{
  "final_location": {
    "lat": 40.7589,
    "lng": -73.9851,
    "address": "456 Broadway, New York"
  },
  "distance": 5.2,
  "duration": 18
}
```

**Respuesta:**
```json
{
  "status": "completed",
  "fare": 18.75,
  "distance": 5.2,
  "duration": 18,
  "payment_required": true,
  "breakdown": {
    "base_fare": 2.50,
    "distance_fare": 8.50,
    "time_fare": 4.25,
    "surge_multiplier": 1.0,
    "service_fee": 3.50
  }
}
```

#### Paso 6: Procesamiento de Pago
**💳 Sistema** procesa el pago

```http
POST /v1/payments/process
Content-Type: application/json
Authorization: Bearer <token>

{
  "ride_id": "ride_123",
  "payment_method_id": "card_789"
}
```

**Respuesta:**
```json
{
  "transaction_id": "txn_456",
  "status": "completed",
  "amount": 18.75,
  "receipt": {
    "receipt_number": "RCP-789",
    "date": "2024-10-03T10:48:00Z",
    "payment_method": "****1234"
  }
}
```

---

### ❌ Flujo 2: Cancelación de Viaje y Devolución Parcial

**Descripción:** Manejo de cancelaciones en diferentes etapas del viaje.

#### Escenario A: Cancelación Temprana (Sin Penalización)
**👤 Pasajero** cancela antes de que el conductor acepte

```http
PUT /v1/rides/ride_123/cancel
Content-Type: application/json
Authorization: Bearer <token>

{
  "reason": "change_of_plans"
}
```

**Respuesta:**
```json
{
  "status": "cancelled",
  "cancellation_fee": 0,
  "refund_amount": 0,
  "reason": "change_of_plans",
  "cancelled_at": "2024-10-03T10:32:00Z"
}
```

#### Escenario B: Cancelación Tardía (Con Penalización)
**👤 Pasajero** cancela después de que el conductor acepte

```http
PUT /v1/rides/ride_123/cancel
Content-Type: application/json
Authorization: Bearer <token>

{
  "reason": "emergency"
}
```

**Respuesta:**
```json
{
  "status": "cancelled",
  "cancellation_fee": 3.25,
  "refund_amount": 12.50,
  "reason": "emergency",
  "cancelled_at": "2024-10-03T10:35:00Z",
  "refund_eta": "3-5 business days"
}
```

#### Procesamiento de Devolución
**💳 Sistema** procesa la devolución

```http
POST /v1/payments/refund
Content-Type: application/json
Authorization: Bearer <token>

{
  "ride_id": "ride_123",
  "amount": 12.50
}
```

**Respuesta:**
```json
{
  "refund_id": "ref_789",
  "status": "processed",
  "amount": 12.50,
  "eta": "3-5 business days",
  "refund_method": "original_payment_method"
}
```

---

### ⭐ Flujo 3: Sistema de Calificaciones Bidireccional

**Descripción:** Proceso de calificación mutua entre pasajero y conductor.

#### Paso 1: Calificación del Conductor
**👤 Pasajero** califica al conductor

```http
POST /v1/ratings
Content-Type: application/json
Authorization: Bearer <token>

{
  "ride_id": "ride_123",
  "rating": 5,
  "comment": "Excelente servicio, muy puntual y amable",
  "type": "driver",
  "categories": {
    "punctuality": 5,
    "cleanliness": 5,
    "friendliness": 4,
    "driving": 5
  }
}
```

**Respuesta:**
```json
{
  "rating_id": "rating_456",
  "status": "submitted",
  "rating": 5,
  "submitted_at": "2024-10-03T10:50:00Z"
}
```

#### Paso 2: Calificación del Pasajero
**🚗 Conductor** califica al pasajero

```http
POST /v1/ratings
Content-Type: application/json
Authorization: Bearer <driver_token>

{
  "ride_id": "ride_123",
  "rating": 4,
  "comment": "Puntual y educado",
  "type": "passenger",
  "categories": {
    "punctuality": 5,
    "politeness": 4,
    "cleanliness": 4
  }
}
```

**Respuesta:**
```json
{
  "rating_id": "rating_789",
  "status": "submitted",
  "rating": 4,
  "submitted_at": "2024-10-03T10:52:00Z"
}
```

#### Paso 3: Consulta de Calificaciones
**👤 Usuario** consulta las calificaciones de un conductor

```http
GET /v1/ratings/driver_456
Authorization: Bearer <token>
```

**Respuesta:**
```json
{
  "user_id": "driver_456",
  "average_rating": 4.8,
  "total_ratings": 150,
  "rating_breakdown": {
    "5_stars": 120,
    "4_stars": 25,
    "3_stars": 3,
    "2_stars": 1,
    "1_star": 1
  },
  "recent_ratings": [
    {
      "rating": 5,
      "comment": "Excelente servicio",
      "date": "2024-10-03T10:50:00Z",
      "categories": {
        "punctuality": 5,
        "cleanliness": 5,
        "friendliness": 4,
        "driving": 5
      }
    }
  ]
}
```

---

## Decisiones de Diseño y Justificación

### Estructura de Endpoints
- **Versionado**: `/v1/` permite evolución sin romper compatibilidad
- **Recursos RESTful**: Siguiendo convenciones REST para claridad
- **Agrupación lógica**: Endpoints relacionados agrupados por funcionalidad

### Seguridad
- **Autenticación JWT**: Tokens seguros con refresh automático
- **Autorización por roles**: Permisos granulares según el tipo de usuario
- **Encriptación**: Datos sensibles encriptados en tránsito y reposo
- **Rate limiting**: Protección contra abuso y ataques

### Privacidad
- **Datos mínimos**: Solo información necesaria expuesta
- **Anonimización**: Ubicaciones históricas anonimizadas
- **Consentimiento**: Usuarios controlan qué datos compartir

### Versionado
- **Semántico**: Cambios breaking requieren nueva versión
- **Deprecación gradual**: Versiones anteriores mantenidas por tiempo limitado
- **Documentación**: Changelog detallado por versión

### Manejo de Errores
- **Códigos HTTP estándar**: Siguiendo convenciones REST
- **Mensajes descriptivos**: Errores claros para debugging
- **Logging estructurado**: Para monitoreo y análisis
- **Retry policies**: Para operaciones transitorias

---

## Manejo de Errores

### Esquema General de Errores
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Los datos proporcionados no son válidos",
    "details": [
      {
        "field": "email",
        "message": "El formato del email no es válido"
      }
    ],
    "timestamp": "2024-10-03T10:30:00Z",
    "request_id": "req_123456"
  }
}
```

### Errores Comunes

| Código HTTP | Código Error | Situación | Descripción |
|-------------|--------------|-----------|-------------|
| 400 | VALIDATION_ERROR | Datos de entrada inválidos | Campos requeridos faltantes o formato incorrecto |
| 401 | UNAUTHORIZED | Token inválido o expirado | Usuario no autenticado o token corrupto |
| 403 | FORBIDDEN | Permisos insuficientes | Usuario autenticado pero sin permisos para la acción |
| 404 | RIDE_NOT_FOUND | Viaje no encontrado | ID de viaje inexistente o usuario no tiene acceso |
| 409 | RIDE_CONFLICT | Estado de viaje incompatible | Intentar aceptar viaje ya aceptado por otro conductor |
| 422 | PAYMENT_FAILED | Pago rechazado | Tarjeta declinada o fondos insuficientes |
| 429 | RATE_LIMIT_EXCEEDED | Límite de solicitudes excedido | Demasiadas solicitudes en ventana de tiempo |
| 500 | INTERNAL_SERVER_ERROR | Error interno del servidor | Fallo en servicios externos o base de datos |

---

## Propuestas de Mejora

### 1. Sistema de Tarifas Dinámicas
- Implementar algoritmos de pricing basados en demanda, hora del día y eventos especiales
- Permitir a conductores establecer multiplicadores de tarifa
- Dashboard en tiempo real de tarifas por zona

### 2. Integración con Transporte Público
- API para consultar rutas de transporte público
- Opción de viajes combinados (ride + transporte público)
- Integración con sistemas de pago de transporte público

### 3. Sistema de Viajes Compartidos
- Matching de pasajeros con destinos similares
- Optimización de rutas para múltiples pasajeros
- División automática de costos entre pasajeros

### 4. Inteligencia Artificial y Machine Learning
- Predicción de demanda para optimizar disponibilidad de conductores
- Sistema de recomendaciones personalizadas
- Detección automática de fraudes y comportamientos sospechosos

### 5. Funcionalidades de Seguridad Avanzadas
- Botón de pánico con geolocalización automática
- Verificación de identidad mediante biometría
- Sistema de reportes automáticos de incidentes
- Integración con servicios de emergencia locales

### 6. Sostenibilidad y Ecología
- Tracking de emisiones de CO2 por viaje
- Incentivos para vehículos eléctricos
- Programa de compensación de carbono
- Integración con vehículos autónomos cuando estén disponibles

