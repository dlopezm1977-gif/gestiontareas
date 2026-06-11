# Gestión de Iniciativas

Aplicación web de seguimiento de proyectos de desarrollo por iniciativas, con Firebase Realtime Database.

## Stack

- HTML + JS vanilla (sin frameworks, sin build step)
- Firebase Realtime Database (RTDB)
- Firebase Auth con email y contraseña
- Firebase Hosting
- PWA instalable (manifest + service worker)

---

## Estructura de la app

```
docs/
  index.html      — lista de proyectos del usuario autenticado
  proyecto.html   — detalle de un proyecto: sistemas, iniciativas y tareas
  manifest.json   — manifiesto PWA (nombre, icono, colores)
  sw.js           — service worker (caché offline de estáticos)
  icon.png        — icono de la app (tab del navegador, pantalla de login, icono PWA)
```

---

## Funcionalidades

### Gestión de proyectos (`index.html`)
- Crear y listar proyectos por usuario autenticado
- Renombrar proyecto (clic sobre el nombre)

### Detalle de proyecto (`proyecto.html`)
- **Sistemas**: crear, renombrar (botón ✏ en hover), colapsar/expandir
- **Iniciativas**: crear, renombrar (botón ✏ en hover), colapsar/expandir
- **Tareas**: crear, editar, eliminar, mover entre sistemas/iniciativas, ciclar estado con un clic
- **Estados**: `Pendiente` → `En curso` → `Bloqueado` → `Hecho` (ciclo al hacer clic)
- **Panel "Requieren atención"**: muestra tareas en estado Pendiente o Bloqueado con chips de iniciativa, sistema y persona alineados a ancho fijo
- **Buscador**: filtra en tiempo real por nombre de sistema, nombre de iniciativa, persona asignada o estado — afecta tanto al panel de atención como al detalle por sistema
- **Resumen de contadores** por estado en la cabecera
- **Guardado automático** en Firebase con indicador "Guardando…"

---

## Setup: primera vez

### 1. Crear proyecto en Firebase Console

1. Ve a [console.firebase.google.com](https://console.firebase.google.com)
2. **Crear un proyecto** → pon un nombre → continuar

### 2. Habilitar Realtime Database

1. En el menú lateral: **Build → Realtime Database**
2. Haz clic en **Crear base de datos**
3. Selecciona la región (ej: `europe-west1`)
4. Elige **Modo de prueba** para empezar (después ajusta las reglas)

### 3. Habilitar Authentication con email/contraseña

1. En el menú lateral: **Build → Authentication**
2. Pestaña **Sign-in method**
3. Haz clic en **Correo electrónico/Contraseña** → habilitar → **Guardar**
4. En la pestaña **Usuarios**, crea manualmente los usuarios que necesites

### 4. Obtener la configuración de tu app web

1. En la pantalla de inicio del proyecto, haz clic en el icono **</>** (Web)
2. Registra la app (nombre libre, activa "También configura Firebase Hosting")
3. Copia el objeto `firebaseConfig` que aparece

### 5. Pegar la config en el código

Busca el bloque `firebaseConfig` **en ambos archivos** (`docs/index.html` y `docs/proyecto.html`) y reemplaza los valores, asegurándote de incluir `databaseURL`:

```js
const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "TU_PROJECT_ID.firebaseapp.com",
  projectId: "TU_PROJECT_ID",
  storageBucket: "TU_PROJECT_ID.firebasestorage.app",
  messagingSenderId: "TU_SENDER_ID",
  appId: "TU_APP_ID",
  databaseURL: "https://TU_PROJECT_ID-default-rtdb.europe-west1.firebasedatabase.app"
};
```

### 6. Actualizar `.firebaserc`

```json
{
  "projects": {
    "default": "TU_PROJECT_ID"
  }
}
```

---

## Desarrollo local

Desde la raíz del proyecto, lanza un servidor HTTP simple con Node.js:

```bash
node serve-local.js
```

Abre `http://localhost:3000` en el navegador.

> `serve-local.js` sirve los archivos de `docs/` y maneja correctamente los query params (p. ej. `proyecto.html?id=...`).

---

## Despliegue

```bash
npm install -g firebase-tools
firebase login
firebase deploy --only hosting
```

La URL de tu app será: `https://TU_PROJECT_ID.web.app`

---

## Estructura de datos en Realtime Database

```
user_projects/{uid}/{projectId}
  name: string
  createdAt: timestamp

projects/{uid}/{projectId}
  name: string
  createdAt: timestamp

  sistemas/{sistemaId}
    name: string
    order: number
    open: boolean

    iniciativas/{iniciativaId}
      name: string
      order: number
      open: boolean

      tasks/{taskId}
        desc: string
        ticket: string
        person: string
        status: 'pendiente' | 'en-curso' | 'bloqueado' | 'hecho'
        note: string
        order: number
        createdAt: timestamp
```

`user_projects` almacena solo metadata ligera (nombre y fecha) para cargar la lista de proyectos sin descargar el árbol completo de sistemas/iniciativas/tareas.

---

## Reglas de seguridad (RTDB)

Las reglas actuales permiten lectura y escritura a usuarios autenticados hasta julio 2026 (modo prueba). Para producción, restringe por UID:

```json
{
  "rules": {
    "projects": {
      "$uid": {
        ".read": "auth != null && auth.uid == $uid",
        ".write": "auth != null && auth.uid == $uid"
      }
    },
    "user_projects": {
      "$uid": {
        ".read": "auth != null && auth.uid == $uid",
        ".write": "auth != null && auth.uid == $uid"
      }
    }
  }
}
```
