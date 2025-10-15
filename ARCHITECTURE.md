# Arquitectura del Sistema de Autenticación

## Visión General

Este sistema implementa autenticación JWT con refresh tokens, OAuth2, y gestión completa de usuarios.

## Estructura de Directorios

```
src/
├── auth/               # Lógica de autenticación
│   ├── jwt.ts         # Generación y verificación de JWT
│   ├── password.ts    # Hash y verificación de contraseñas
│   ├── oauth.ts       # Proveedores OAuth (Google, GitHub)
│   └── session.ts     # Gestión de sesiones con Redis
├── middleware/         # Middlewares Express
│   ├── auth.ts        # Verificación de tokens
│   ├── rate-limit.ts  # Rate limiting
│   └── validate.ts    # Validación de datos (Zod)
├── routes/            # Rutas de la API
│   ├── auth.ts        # /auth/* (login, register, refresh)
│   ├── user.ts        # /user/* (perfil, update)
│   └── oauth.ts       # /oauth/* (google, github)
├── utils/             # Utilidades
│   ├── errors.ts      # Clases de error personalizadas
│   ├── logger.ts      # Winston logger
│   └── email.ts       # Envío de emails
├── types/             # Tipos TypeScript
│   ├── user.ts        # Tipos de usuario
│   ├── token.ts       # Tipos de tokens
│   └── index.ts       # Exports
├── config/            # Configuración
│   ├── database.ts    # Conexión a DB
│   ├── redis.ts       # Conexión a Redis
│   └── env.ts         # Variables de entorno
└── index.ts           # Punto de entrada
```

## Flujo de Autenticación

### 1. Registro (Sign Up)
```
Usuario → POST /auth/register → Validación → Hash password →
Guardar en DB → Email de verificación → Access + Refresh tokens
```

### 2. Login
```
Usuario → POST /auth/login → Validar credenciales →
Generar Access + Refresh tokens → Guardar refresh en Redis →
Retornar tokens
```

### 3. Refresh Token
```
Usuario → POST /auth/refresh → Validar refresh token →
Verificar en Redis → Generar nuevo Access token → Retornar
```

### 4. OAuth (Google/GitHub)
```
Usuario → GET /oauth/google → Redirect a Google →
Callback → Verificar código → Crear/Actualizar usuario →
Generar tokens → Redirect a frontend
```

## Tokens JWT

### Access Token
- Vida corta: 15 minutos
- Contiene: userId, email, role
- Se valida en cada request protegido

### Refresh Token
- Vida larga: 7 días
- Almacenado en Redis (revocable)
- Solo se usa para generar nuevos access tokens

## Seguridad

### Implementaciones clave:
- Bcrypt para hash de contraseñas (10 rounds)
- JWT con algoritmo HS256
- Rate limiting por IP
- CORS configurado
- Helmet para headers de seguridad
- Validación de entrada con Zod
- Sanitización de datos
- HTTPS obligatorio en producción

### Prevención de ataques:
- **Brute Force**: Rate limiting + account lockout
- **XSS**: Sanitización de inputs
- **CSRF**: Tokens CSRF en cookies
- **SQL Injection**: ORM/Query builders con prepared statements
- **Session Hijacking**: Refresh token rotation

## Base de Datos

### Modelo de Usuario
```typescript
interface User {
  id: string
  email: string
  password?: string  // null para OAuth users
  name: string
  role: 'user' | 'admin'
  emailVerified: boolean
  provider: 'local' | 'google' | 'github'
  providerId?: string
  createdAt: Date
  updatedAt: Date
}
```

### Modelo de Refresh Token
```typescript
interface RefreshToken {
  token: string
  userId: string
  expiresAt: Date
  createdAt: Date
}
```

## API Endpoints

### Autenticación
- POST /auth/register - Registro
- POST /auth/login - Login
- POST /auth/logout - Logout
- POST /auth/refresh - Refresh token
- POST /auth/verify-email - Verificar email
- POST /auth/forgot-password - Recuperar contraseña
- POST /auth/reset-password - Resetear contraseña

### OAuth
- GET /oauth/google - Iniciar OAuth con Google
- GET /oauth/google/callback - Callback de Google
- GET /oauth/github - Iniciar OAuth con GitHub
- GET /oauth/github/callback - Callback de GitHub

### Usuario
- GET /user/me - Obtener perfil (protegido)
- PUT /user/me - Actualizar perfil (protegido)
- DELETE /user/me - Eliminar cuenta (protegido)

## Próximos Pasos

1. Configurar Express + TypeScript
2. Implementar JWT utils
3. Crear modelos de base de datos
4. Implementar rutas de auth
5. Agregar OAuth providers
6. Testing completo
7. Documentación de API (Swagger)
8. Deploy
