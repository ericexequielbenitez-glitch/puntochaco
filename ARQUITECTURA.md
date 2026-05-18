# PUNTO CHACO — Arquitectura Técnica Completa

## Visión General

Marketplace local + comunidad de emprendedores de la provincia del Chaco, Argentina.

---

## Tecnologías Recomendadas

### Frontend
- **Next.js 14** (App Router) — SSR/SSG para SEO óptimo
- **React 18** — UI componentes
- **TailwindCSS** — Estilos utilitarios
- **Framer Motion** — Animaciones
- **Zustand** — Estado global
- **React Hook Form + Zod** — Formularios y validaciones

### Backend / BaaS
- **Firebase** (Firestore, Auth, Storage, Functions)
- **Firebase Cloud Functions** — Lógica server-side (validaciones, moderación)

### Extras
- **Algolia o Typesense** — Búsqueda avanzada
- **Vercel** — Deploy del frontend
- **Cloudinary** — Optimización de imágenes

---

## Estructura de Carpetas (Next.js)

```
punto-chaco/
├── app/
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── registro/page.tsx
│   ├── (marketplace)/
│   │   ├── page.tsx                 # Landing
│   │   ├── explorar/page.tsx        # Listado negocios
│   │   ├── negocio/[slug]/page.tsx  # Perfil negocio
│   │   ├── categorias/page.tsx
│   │   └── servicios/page.tsx
│   ├── (dashboard)/
│   │   ├── mi-cuenta/page.tsx
│   │   ├── mis-publicaciones/page.tsx
│   │   ├── turnos/page.tsx
│   │   └── configuracion/page.tsx
│   └── admin/
│       ├── page.tsx                 # Dashboard admin
│       ├── publicaciones/page.tsx
│       ├── usuarios/page.tsx
│       ├── reportes/page.tsx
│       └── categorias/page.tsx
├── components/
│   ├── ui/                          # Componentes base
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Card.tsx
│   │   ├── Modal.tsx
│   │   └── Badge.tsx
│   ├── layout/
│   │   ├── Navbar.tsx
│   │   ├── Footer.tsx
│   │   └── Sidebar.tsx
│   ├── marketplace/
│   │   ├── BusinessCard.tsx
│   │   ├── ProductCard.tsx
│   │   ├── SearchBar.tsx
│   │   └── FilterPanel.tsx
│   ├── forms/
│   │   ├── RegisterForm.tsx
│   │   ├── PublicationForm.tsx
│   │   └── ScheduleForm.tsx
│   └── booking/
│       ├── Calendar.tsx
│       ├── TimeSlots.tsx
│       └── BookingModal.tsx
├── lib/
│   ├── firebase.ts
│   ├── auth.ts
│   ├── firestore.ts
│   └── validators.ts
├── hooks/
│   ├── useAuth.ts
│   ├── useBusinesses.ts
│   └── useBooking.ts
├── store/
│   └── authStore.ts
├── types/
│   └── index.ts
└── constants/
    ├── categories.ts
    ├── localities.ts
    └── blockedWords.ts
```

---

## Modelo de Base de Datos (Firestore)

### Colección: `users`
```json
{
  "uid": "string",
  "firstName": "string",
  "lastName": "string",
  "email": "string",
  "birthDate": "timestamp",
  "age": "number",
  "role": "buyer | seller | admin",
  "createdAt": "timestamp",
  "isActive": "boolean",
  "isBanned": "boolean"
}
```

### Colección: `businesses`
```json
{
  "id": "string",
  "ownerId": "string (ref: users)",
  "name": "string",
  "slug": "string (url-friendly)",
  "description": "string",
  "category": "string",
  "subcategory": "string",
  "locality": "string",
  "address": "string",
  "whatsapp": "string",
  "instagram": "string",
  "logoUrl": "string",
  "coverUrl": "string",
  "schedule": {
    "monday": { "open": "09:00", "close": "18:00", "isOpen": true },
    "tuesday": { "open": "09:00", "close": "18:00", "isOpen": true }
  },
  "isApproved": "boolean",
  "isFeatured": "boolean",
  "hasAppointments": "boolean",
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

### Colección: `publications`
```json
{
  "id": "string",
  "businessId": "string (ref: businesses)",
  "ownerId": "string (ref: users)",
  "title": "string",
  "description": "string",
  "price": "number",
  "priceLabel": "string",
  "category": "string",
  "imageUrl": "string",
  "locality": "string",
  "isApproved": "boolean",
  "isActive": "boolean",
  "reportCount": "number",
  "createdAt": "timestamp"
}
```

### Colección: `appointments`
```json
{
  "id": "string",
  "businessId": "string",
  "clientId": "string",
  "date": "timestamp",
  "timeSlot": "string",
  "duration": "number (minutes)",
  "status": "pending | confirmed | cancelled",
  "notes": "string",
  "createdAt": "timestamp"
}
```

### Colección: `scheduleConfig` (subcol. de businesses)
```json
{
  "businessId": "string",
  "availableDays": ["monday", "tuesday", "friday"],
  "startTime": "09:00",
  "endTime": "18:00",
  "slotDuration": 30,
  "maxPerSlot": 1,
  "isActive": "boolean"
}
```

### Colección: `reports`
```json
{
  "id": "string",
  "targetId": "string",
  "targetType": "publication | business | user",
  "reporterId": "string",
  "reason": "string",
  "status": "pending | resolved | dismissed",
  "createdAt": "timestamp"
}
```

### Colección: `categories`
```json
{
  "id": "string",
  "name": "string",
  "type": "product | service",
  "icon": "string",
  "slug": "string",
  "isActive": "boolean",
  "order": "number",
  "createdBy": "admin"
}
```

---

## Flujo de Usuarios

### Comprador
1. Llega a la landing page
2. Explora negocios / usa búsqueda o filtros
3. Ve perfil de negocio (`/negocio/[slug]`)
4. Contacta por WhatsApp o Instagram
5. (Opcional) Registra cuenta para reservar turnos
6. Reserva turno → recibe confirmación

### Vendedor
1. Registra cuenta como vendedor
2. Validación de edad (≥18) en frontend y backend
3. Completa perfil del negocio
4. Verificación de nombre único (no duplicados)
5. Espera aprobación del administrador
6. Publica hasta 10 publicaciones
7. Activa agenda de turnos (opcional)
8. Gestiona publicaciones y reservas desde el dashboard

### Administrador
1. Recibe notificaciones de nuevos negocios/publicaciones
2. Aprueba o rechaza contenido
3. Gestiona reportes de usuarios
4. Puede destacar negocios
5. Gestiona categorías (crear, editar, eliminar)
6. Ve métricas de la plataforma
7. Puede banear usuarios

---

## Validaciones Críticas

### Validación de Edad
```typescript
// lib/validators.ts
export function validateAge(birthDate: Date): boolean {
  const today = new Date();
  const age = today.getFullYear() - birthDate.getFullYear();
  const m = today.getMonth() - birthDate.getMonth();
  const adjustedAge = (m < 0 || (m === 0 && today.getDate() < birthDate.getDate()))
    ? age - 1 : age;
  return adjustedAge >= 18;
}
```

### Validación de Nombre de Negocio (Firebase Function)
```typescript
// functions/src/validateBusinessName.ts
export const validateBusinessName = functions.https.onCall(async (data) => {
  const { name } = data;
  const normalized = normalize(name); // lowercase, sin acentos
  const snapshot = await db.collection('businesses').get();
  const names = snapshot.docs.map(d => normalize(d.data().name));
  const hasDuplicate = names.some(n => similarity(n, normalized) > 0.85);
  return { available: !hasDuplicate };
});
```

### Filtro de Palabras Prohibidas
```typescript
// constants/blockedWords.ts
export const BLOCKED_WORDS = [
  'cigarrillo', 'cigarro', 'tabaco', 'alcohol', 'cerveza',
  'vino', 'fernet', 'droga', 'cocaína', 'marihuana',
  'cannabis', 'sustancia', // etc.
];

export function containsBlockedWords(text: string): boolean {
  const lower = text.toLowerCase();
  return BLOCKED_WORDS.some(word => lower.includes(word));
}
```

---

## Seguridad (Firestore Rules)

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == userId;
    }
    // Businesses
    match /businesses/{businessId} {
      allow read: if true; // público
      allow create: if request.auth != null
        && request.resource.data.ownerId == request.auth.uid;
      allow update, delete: if request.auth.uid == resource.data.ownerId
        || isAdmin();
    }
    // Publications
    match /publications/{pubId} {
      allow read: if true;
      allow create: if request.auth != null
        && businessPublicationCount() < 10;
      allow update, delete: if isOwner() || isAdmin();
    }
    // Admin only
    match /categories/{catId} {
      allow read: if true;
      allow write: if isAdmin();
    }
    function isAdmin() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
  }
}
```

---

## URLs Amigables (SEO)

```
/                              # Landing
/explorar                      # Todos los negocios
/explorar?categoria=postres    # Filtrado
/explorar?localidad=resistencia
/servicios                     # Sección servicios
/negocio/vero-pasteleria       # Perfil de negocio
/categorias                    # Listado categorías
/registro                      # Registro
/login                         # Login
/dashboard                     # Panel vendedor
/admin                         # Panel administrador
```

---

## Pantallas a Desarrollar

- [x] Landing Page (`index.html`)
- [x] Registro (`registro.html`)  
- [x] Perfil de negocio (`negocio.html`)
- [ ] Login
- [ ] Explorar negocios
- [ ] Panel vendedor
- [ ] Panel administrador
- [ ] Sistema de turnos (modal)

---

*Desarrollado para el ecosistema de emprendedores de la provincia del Chaco, Argentina.*
