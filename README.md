# Gestión de Tareas por Oleadas

Aplicación web de seguimiento de proyectos de desarrollo por oleadas, con Firebase Firestore en tiempo real.

## Stack

- HTML + JS vanilla (sin frameworks, sin build step)
- Firebase Firestore (base de datos en tiempo real)
- Firebase Auth con Google
- Firebase Hosting

---

## Setup: primeras veces

### 1. Crear proyecto en Firebase Console

1. Ve a [console.firebase.google.com](https://console.firebase.google.com)
2. **Crear un proyecto** → pon un nombre (ej: `gestion-oleadas`) → continuar
3. Puedes desactivar Google Analytics si no lo necesitas

### 2. Habilitar Firestore

1. En el menú lateral: **Build → Firestore Database**
2. Haz clic en **Crear base de datos**
3. Elige **Modo producción** (las reglas del repo ya están configuradas)
4. Selecciona la región más cercana (ej: `europe-west1`)

### 3. Habilitar Authentication con Google

1. En el menú lateral: **Build → Authentication**
2. Pestaña **Sign-in method**
3. Haz clic en **Google** → habilitar → guarda el correo de soporte → **Guardar**

### 4. Obtener la configuración de tu app web

1. En la pantalla de inicio del proyecto, haz clic en el icono **</>** (Web)
2. Registra la app (nombre libre, activa "También configura Firebase Hosting")
3. Copia el objeto `firebaseConfig` que aparece

### 5. Pegar la config en el código

Busca este bloque **en ambos archivos** (`public/index.html` y `public/proyecto.html`) y reemplaza los valores:

```js
const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "TU_PROJECT_ID.firebaseapp.com",
  projectId: "TU_PROJECT_ID",
  ...
};
```

### 6. Actualizar `.firebaserc`

Reemplaza `TU_PROJECT_ID` con el ID real de tu proyecto Firebase:

```json
{
  "projects": {
    "default": "gestion-oleadas-xxxxx"
  }
}
```

---

## Despliegue

### Requisitos

```bash
npm install -g firebase-tools
firebase login
```

### Primer deploy

```bash
cd gestiontareas
firebase deploy
```

Esto despliega:
- El hosting (los HTML en `public/`)
- Las reglas de Firestore (`firestore.rules`)

La URL de tu app será: `https://TU_PROJECT_ID.web.app`

### Deploys posteriores

```bash
firebase deploy
```

O solo el hosting:

```bash
firebase deploy --only hosting
```

---

## Estructura de datos en Firestore

```
projects/{projectId}
  name: string
  ownerUid: string
  createdAt: timestamp

  sistemas/{sistemaId}
    name: string
    order: number
    open: boolean

    oleadas/{oleadaId}
      name: string
      order: number
      open: boolean

      tasks/{taskId}
        desc: string
        ticket: string
        person: string
        status: 'tercero' | 'en-curso' | 'bloqueado' | 'hecho'
        note: string
        order: number
        createdAt: timestamp
```

## Reglas de seguridad

Solo el usuario autenticado que creó el proyecto puede leer y escribir sus datos. Las reglas están en `firestore.rules`.
