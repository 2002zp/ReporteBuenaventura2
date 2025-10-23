# 🔍 DIAGNÓSTICO COMPLETO DE PROBLEMAS

## ❌ PROBLEMA PRINCIPAL

**Los reportes NO están guardándose en la base de datos PostgreSQL.**

Evidencia:
- ✅ La tabla `entities` tiene 11 registros
- ✅ La tabla `profiles` tiene usuarios
- ❌ La tabla `reports` está VACÍA (0 registros)
- ❌ La aplicación muestra reportes que NO existen en la BD
- ❌ El admin no puede cambiar el estado de los reportes

---

## 🐛 CAUSA RAÍZ

### 1. **Incompatibilidad de Campos en ReportForm.tsx**

El formulario de creación está enviando datos con **nombres de campos incorrectos**:

```typescript
// ❌ Lo que ENVIABA (INCORRECTO):
{
  entity: "Alcaldía...",        // Debería ser entityName
  address: "Calle 5...",        // Debería ser location
  image: "data:image...",       // Debería ser images (array)
  location: { lat, lng },       // Debería ser locationLat y locationLng
  // ❌ Falta category (REQUERIDO)
}

// ✅ Lo que DEBE ENVIAR (CORRECTO):
{
  category: "infraestructura",
  entityId: "uuid-123...",
  entityName: "Alcaldía...",
  location: "Calle 5...",
  locationLat: 3.8801,
  locationLng: -77.0312,
  images: ["data:image..."]  
}
```

### 2. **Inconsistencia en Nombres de Campos**

Múltiples componentes usan `report.entity` cuando el backend retorna `report.entityName`:

**Componentes afectados:**
- `AdminAnalytics.tsx` (líneas 83, 87)
- `AdminDashboard.tsx` (líneas 109, 113, 506, 522)
- `PublicReportsTracking.tsx` (líneas 75, 258, 268, 346, 424, 450)
- `MyReports.tsx` (líneas 104, 320, 396, 463, 504)
- `ReportsList.tsx` (líneas 339, 392)

### 3. **Mapeo de Interfaces**

Las interfaces `Report` en cada componente usan `entity: string` cuando deberían usar `entityName: string`.

---

## ✅ SOLUCIONES IMPLEMENTADAS

### 1. **Arreglado ReportForm.tsx**

```typescript
// ✅ AHORA ENVÍA:
const reportData = {
  title: formData.title,
  description: formData.description,
  category: category,              // ✅ Nuevo
  entityId: entityId,              // ✅ Busca UUID
  entityName: finalEntity,         // ✅ Cambio
  location: finalAddress,          // ✅ Cambio
  locationLat: formData.location.lat,  // ✅ Nuevo
  locationLng: formData.location.lng,  // ✅ Nuevo
  images: imagePreview ? [imagePreview] : [],  // ✅ Array
};
```

### 2. **Arreglado AdminReportsManagement.tsx**

- ✅ Interface actualizada con `entityName` y compatibilidad con `entity`
- ✅ Filtros usan `report.entityName || report.entity`
- ✅ Visualización usa `report.entityName || report.entity || 'Sin asignar'`
- ✅ Actualización envía `entityName` y `entityId`

### 3. **Arreglado EntityList.tsx**

- ✅ Usa `r.entityName` en lugar de `r.entity` para contar reportes
- ✅ Manejo de errores mejorado
- ✅ Función `createWhatsAppUrl` agregada

---

## 🔧 PRÓXIMOS PASOS PENDIENTES

### Paso 1: Actualizar Componentes que Usan `report.entity`

Necesito actualizar TODOS estos componentes para que usen:
```typescript
const entityName = report.entityName || report.entity || 'Sin asignar';
```

**Archivos a actualizar:**
1. ❌ `AdminAnalytics.tsx`
2. ❌ `AdminDashboard.tsx`
3. ❌ `PublicReportsTracking.tsx`
4. ❌ `MyReports.tsx`
5. ❌ `ReportsList.tsx`

### Paso 2: Actualizar Interfaces Report

Cambiar en todos los componentes:
```typescript
// ❌ ANTES
interface Report {
  entity: string;
  address?: string;
  image?: string;
}

// ✅ DESPUÉS
interface Report {
  entityName: string;
  entityId?: string;
  location?: string;
  images?: string[];
  // Compatibilidad
  entity?: string;
  address?: string;
  image?: string;
}
```

### Paso 3: Desplegar Edge Function

Asegurarse de que el Edge Function esté desplegado con el código actualizado.

### Paso 4: Verificar en Base de Datos

Después de crear un reporte de prueba, verificar:
```sql
SELECT 
  id,
  title,
  entity_name,
  entity_id,
  category,
  status,
  created_at
FROM reports
ORDER BY created_at DESC
LIMIT 5;
```

---

## 📊 ESTRUCTURA ESPERADA

### Tabla `reports` en PostgreSQL

```sql
CREATE TABLE reports (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  category TEXT NOT NULL,           -- REQUERIDO
  entity_id UUID REFERENCES entities(id),
  entity_name TEXT,                 -- Nombre de la entidad
  location_address TEXT,            -- Dirección legible
  location_lat DECIMAL,             -- Latitud
  location_lng DECIMAL,             -- Longitud
  images TEXT[],                    -- Array de imágenes
  status TEXT DEFAULT 'pendiente',
  priority TEXT DEFAULT 'media',
  is_public BOOLEAN DEFAULT true,
  is_anonymous BOOLEAN DEFAULT false,
  admin_notes TEXT,
  resolved_at TIMESTAMP,
  resolved_by UUID,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Formato de Respuesta del Edge Function

```typescript
{
  id: "uuid-123",
  userId: "uuid-456",
  userName: "Juan Pérez",
  userEmail: "juan@email.com",
  title: "Hueco en la calle",
  description: "Hay un hueco grande...",
  category: "infraestructura",
  status: "pendiente",
  priority: "media",
  entityId: "uuid-789",
  entityName: "Alcaldía de Buenaventura",
  location: "Calle 5 con Carrera 10",
  locationLat: 3.8801,
  locationLng: -77.0312,
  images: ["data:image/jpeg;base64..."],
  createdAt: "2025-10-22T...",
  updatedAt: "2025-10-22T...",
  isPublic: true,
  isAnonymous: false
}
```

---

## 🎯 CHECKLIST DE VERIFICACIÓN

### Backend (Edge Function)
- [x] Edge Function usa PostgreSQL
- [x] Endpoint POST /reports inserta en tabla `reports`
- [x] Endpoint GET /reports lee de tabla `reports`
- [x] Endpoint PUT /reports actualiza tabla `reports`
- [x] Respuesta transforma campos snake_case a camelCase

### Frontend - Envío de Datos
- [x] ReportForm envía `category` (requerido)
- [x] ReportForm envía `entityName` (no `entity`)
- [x] ReportForm envía `location` (no `address`)
- [x] ReportForm envía `locationLat` y `locationLng`
- [x] ReportForm envía `images` como array (no `image`)
- [x] ReportForm busca `entityId` de la entidad seleccionada

### Frontend - Lectura de Datos
- [ ] AdminAnalytics usa `entityName`
- [ ] AdminDashboard usa `entityName`
- [x] AdminReportsManagement usa `entityName`
- [ ] PublicReportsTracking usa `entityName`
- [ ] MyReports usa `entityName`
- [ ] ReportsList usa `entityName`

### Base de Datos
- [x] Tabla `entities` tiene 11 registros
- [x] Tabla `profiles` tiene usuarios
- [ ] Tabla `reports` recibe nuevos reportes
- [ ] Tabla `comments` recibe comentarios

---

## 🆘 SI SIGUE SIN FUNCIONAR

### 1. Verificar Edge Function en Supabase

Dashboard → Edge Functions → server → Logs

Buscar errores como:
```
Error creating report: ...
```

### 2. Verificar en Console del Navegador

F12 → Console → Network → Buscar llamada a:
```
POST https://[project-id].supabase.co/functions/v1/make-server-e2de53ff/reports
```

Ver:
- Request Payload (datos enviados)
- Response (respuesta del servidor)
- Status Code (200 OK o error)

### 3. Verificar accessToken

En Console del navegador:
```javascript
localStorage.getItem('accessToken')
// Debería retornar un token JWT, no null
```

### 4. Verificar con SQL

```sql
-- ¿Se creó algún reporte?
SELECT COUNT(*) FROM reports;

-- Ver reportes recientes
SELECT * FROM reports ORDER BY created_at DESC LIMIT 5;

-- Ver errores en logs
SELECT * FROM edge_function_logs 
WHERE function_name = 'server' 
ORDER BY timestamp DESC 
LIMIT 20;
```

---

**© 2025 ZPservicioTecnico**  
*Diagnóstico realizado el 22 de octubre de 2025*
