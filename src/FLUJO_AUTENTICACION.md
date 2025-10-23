# 🔐 Flujo de Autenticación - ReporteBuenaventura

## 📊 Diagrama de Flujo Completo

### 1️⃣ Registro con Email/Password

```
Usuario visita app
       ↓
Hace clic en "Registrar"
       ↓
Completa formulario (nombre, email, password)
       ↓
Hace clic en "Registrar"
       ↓
[BACKEND] Valida datos
       ↓
[BACKEND] ¿Email ya existe?
       ├─ SÍ → ❌ Error: "Ya existe cuenta con este email"
       └─ NO → Continúa
              ↓
       [BACKEND] Crea usuario en Supabase Auth
              ↓
       [BACKEND] email_confirm = false
              ↓
       [SUPABASE] Envía email de verificación automáticamente
              ↓
       ✅ Mensaje: "¡Cuenta creada! Revisa tu email"
              ↓
       [FRONTEND] Limpia formulario
              ↓
       [FRONTEND] Espera 5 segundos
              ↓
       [FRONTEND] Cambia a vista de Login
              ↓
       ⏳ Usuario espera recibir email...
              ↓
       📧 Usuario recibe email de Supabase
              ↓
       Usuario hace clic en "Confirmar Email"
              ↓
       [SUPABASE] Marca email como verificado
              ↓
       ✅ Email confirmado en Supabase Auth
```

### 2️⃣ Login con Email/Password (Usuario Verificado)

```
Usuario ingresa email y contraseña
       ↓
Hace clic en "Iniciar Sesión"
       ↓
[BACKEND] Valida credenciales con Supabase Auth
       ↓
[BACKEND] ¿Credenciales correctas?
       ├─ NO → [BACKEND] ¿Email existe?
       │        ├─ NO → ❌ "No existe cuenta"
       │        └─ SÍ → ❌ "Contraseña incorrecta"
       └─ SÍ → Continúa
              ↓
       [BACKEND] ¿Email verificado? (email_confirmed_at existe)
              ├─ NO → ❌ "Por favor verifica tu email"
              └─ SÍ → Continúa
                     ↓
              [BACKEND] ¿Usuario existe en KV Store?
                     ├─ NO → Crea entrada en KV Store
                     └─ SÍ → Usa datos existentes
                            ↓
                     Genera accessToken
                            ↓
                     Retorna {accessToken, user}
                            ↓
              [FRONTEND] Guarda token en localStorage
                            ↓
              [FRONTEND] Actualiza estado de usuario
                            ↓
              ✅ Usuario entra a la aplicación
```

### 3️⃣ Login con Email/Password (Usuario NO Verificado)

```
Usuario ingresa email y contraseña
       ↓
Hace clic en "Iniciar Sesión"
       ↓
[BACKEND] Valida credenciales
       ↓
✅ Credenciales correctas
       ↓
[BACKEND] Verifica email_confirmed_at
       ↓
❌ email_confirmed_at = null
       ↓
[BACKEND] Cierra sesión (signOut)
       ↓
❌ Error: "Por favor verifica tu correo electrónico..."
       ↓
👉 Usuario debe revisar su email y hacer clic en enlace
```

### 4️⃣ Google OAuth (Auto-verificado)

```
Usuario hace clic en "Continuar con Google"
       ↓
[FRONTEND] Llama signInWithGoogle()
       ↓
[SUPABASE] Redirige a Google OAuth
       ↓
Usuario selecciona cuenta de Google
       ↓
Google autentica usuario
       ↓
[GOOGLE] Verifica email del usuario
       ↓
[SUPABASE] Crea sesión OAuth
       ↓
[GOOGLE] Redirige de vuelta a la app
       ↓
[FRONTEND] Detecta sesión de Google
       ↓
[FRONTEND] Extrae datos del usuario (email, nombre, id)
       ↓
[FRONTEND] ¿Usuario existe en sistema?
       ├─ SÍ → [BACKEND] Login normal
       │              ↓
       │       Retorna accessToken
       │              ↓
       │       ✅ "Bienvenido de nuevo, [nombre]"
       │
       └─ NO → [BACKEND] Signup automático
                      ↓
              Password = "google-oauth-[googleId]"
                      ↓
              email_confirm = true ← ✅ Auto-confirmado
                      ↓
              Crea entrada en KV Store inmediatamente
                      ↓
              [BACKEND] Login automático
                      ↓
              ✅ "Cuenta creada. ¡Bienvenido, [nombre]!"
                      ↓
              ✅ Usuario entra a la aplicación
```

### 5️⃣ Recuperación de Contraseña

```
Usuario hace clic en "¿Has olvidado la contraseña?"
       ↓
Ingresa su email
       ↓
Hace clic en "Enviar Enlace"
       ↓
[SUPABASE] resetPasswordForEmail()
       ↓
[SUPABASE] Envía email con enlace de reset
       ↓
✅ "Te hemos enviado un enlace..."
       ↓
📧 Usuario recibe email
       ↓
Usuario hace clic en enlace de reset
       ↓
[SUPABASE] Redirige a /reset-password
       ↓
[FRONTEND] Componente ResetPassword
       ↓
Usuario ingresa nueva contraseña
       ↓
[SUPABASE] Actualiza contraseña
       ↓
✅ "Contraseña actualizada"
       ↓
Usuario puede hacer login con nueva contraseña
```

## 🔒 Validaciones de Seguridad

### En el Registro (Signup)
- ✅ Email válido (formato)
- ✅ Contraseña mínimo 6 caracteres
- ✅ Nombre no vacío
- ✅ Email único (no duplicados)
- ✅ Email de verificación enviado

### En el Login (Signin)
- ✅ Email y contraseña no vacíos
- ✅ Credenciales válidas en Supabase Auth
- ✅ **Email verificado** (email_confirmed_at existe)
- ✅ Usuario no bloqueado
- ✅ Token generado correctamente

### En Google OAuth
- ✅ Sesión válida de Google
- ✅ Email verificado por Google
- ✅ Auto-confirmación de email
- ✅ Password único por usuario

## 📋 Estados de Usuario

### Estado en Supabase Auth

| Estado | email_confirmed_at | Puede Login | Descripción |
|--------|-------------------|-------------|-------------|
| **Registrado** | `null` | ❌ NO | Email no verificado |
| **Verificado** | `2025-10-21...` | ✅ SÍ | Email confirmado |
| **Google OAuth** | `2025-10-21...` | ✅ SÍ | Auto-confirmado |

### Estado en KV Store

| Condición | Existe en KV | Cuándo se Crea |
|-----------|-------------|----------------|
| Usuario normal sin verificar | ❌ NO | Nunca |
| Usuario normal verificado, primer login | ✅ SÍ | Al hacer login |
| Usuario normal verificado, segundo login | ✅ SÍ | Ya existe |
| Usuario Google OAuth | ✅ SÍ | Al registrarse |

## 🎯 Mensajes de Usuario

### Mensajes de Éxito ✅

```
[Registro]
"¡Cuenta creada exitosamente! Por favor revisa tu correo electrónico 
y haz clic en el enlace de verificación antes de iniciar sesión. 
Revisa también tu carpeta de spam."

[Login Normal]
"Bienvenido, Juan Pérez"

[Google OAuth - Nueva Cuenta]
"Cuenta creada. ¡Bienvenido, Juan Pérez!"

[Google OAuth - Usuario Existente]
"Bienvenido de nuevo, Juan Pérez"
```

### Mensajes de Error ❌

```
[Email No Verificado]
"Por favor verifica tu correo electrónico antes de iniciar sesión. 
Revisa tu bandeja de entrada y carpeta de spam."

[Email Ya Existe]
"Ya existe una cuenta con este correo electrónico. 
Por favor inicia sesión o recupera tu contraseña."

[Contraseña Incorrecta]
"Contraseña incorrecta. Si olvidaste tu contraseña, 
usa la opción '¿Has olvidado la contraseña?'"

[Email No Registrado]
"No existe una cuenta con este correo electrónico. 
Por favor regístrate primero."

[Campos Vacíos]
"Por favor completa todos los campos"

[Contraseña Corta]
"La contraseña debe tener al menos 6 caracteres"
```

## 🔄 Sincronización KV Store ↔ Supabase Auth

### ¿Por qué dos sistemas?

- **Supabase Auth**: Maneja autenticación y seguridad
- **KV Store**: Almacena datos adicionales del usuario (role, reportes, etc.)

### Flujo de Sincronización

```
Usuario se registra
       ↓
Crea en Supabase Auth ← ✅ Sistema de autenticación
       ↓
Email NO verificado → NO se crea en KV Store
       ↓
Usuario verifica email
       ↓
Usuario hace login
       ↓
Login exitoso → SE crea en KV Store ← ✅ Datos de aplicación
       ↓
Futuros logins → Usa KV Store existente
```

### Para Google OAuth

```
Usuario hace login con Google
       ↓
Crea en Supabase Auth (email_confirm = true)
       ↓
Inmediatamente crea en KV Store
       ↓
Usuario puede acceder inmediatamente
```

## 🚀 Optimizaciones Implementadas

1. **Auto-limpieza de formulario** después de registro exitoso
2. **Auto-switch a login** después de 5 segundos
3. **Detección inteligente** de Google OAuth vs Email/Password
4. **Creación diferida** de KV Store (solo usuarios verificados)
5. **Mensajes descriptivos** en español para cada caso
6. **Protección contra spam** (usuario debe verificar email)

## 📞 Soporte

**Si tienes problemas:**
- 📖 Lee `/SOLUCION_VERIFICACION_EMAIL.md`
- 🧪 Prueba con `/TEST_VERIFICACION_EMAIL.html`
- 📧 Contacta: johnvalenciazp@gmail.com

---
**Actualizado:** 21 de Octubre, 2025  
**Versión:** 2.0 (Con verificación de email)  
**Desarrollado por:** ZPservicioTecnico
