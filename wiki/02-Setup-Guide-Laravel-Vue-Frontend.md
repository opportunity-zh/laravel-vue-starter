# Setup Guide Vue in Laravel

## Voraussetzungen

-   Guide: [Setup Guide Laravel](/wiki/01-Setup-Guide-Laravel-API.md)
-   Node.js
-   NPM

# Installation

## 1. Vue.js, vue plugin, vue router, pinia und axios installieren

```bash
npm install vue
npm install @vitejs/plugin-vue
npm install vue-router@4
npm install pinia
npm install axios
```

## 2. Vue.js in Laravel einbinden

Wir müssen vue.js in die vite.config.js einbinden. Gleichzeitig setzen wir auch noch den relative @-Alias für die src-Ordnerstruktur.

```javascript
import { defineConfig } from "vite";
import laravel from "laravel-vite-plugin";
import vue from "@vitejs/plugin-vue"; // add the vue plugin
import { fileURLToPath, URL } from "node:url"; // add the fileURLToPath and URL

export default defineConfig({
    plugins: [
        vue(), // use the vue plugin
        laravel({
            input: ["resources/css/app.css", "resources/js/app.js"],
            refresh: true,
        }),
    ],
    // add the resolve object
    resolve: {
        alias: {
            "@": fileURLToPath(new URL("./resources/js", import.meta.url)),
        },
    },
});
```

## Vue.js Konfiguration und Basisstruktur

1. Wir erweitern die app.js um die Vue-Initialisierung und binden die notwendigen Plugins ein (Pinia).

```javascript
import "./bootstrap";

import { createApp } from "vue";
import { createPinia } from "pinia";

import App from "./App.vue";
import router from "./router";

const app = createApp(App);

app.use(createPinia());
app.use(router);

app.mount("#app");
```

2. Erstellen die Datei resources/js/App.vue und fügen den folgenden Code ein:

```vue
<script setup>
import { RouterLink, RouterView } from "vue-router";
</script>

<template>
    <header>
        <div>
            <nav>
                <RouterLink to="/">Home</RouterLink>
            </nav>
        </div>
    </header>
    <RouterView />
</template>
```

3. Füge in der .env Datei die VITE\*BASE*URL Variable hinzu, es sollt schon irgendwo eine `VITE*` Variable geben, füge die VITE_BASE_URL Variable einfach darunter hinzu.

```env
VITE_BASE_URL="${APP_URL}"
```

4. Erstellen die Datei resources/js/router/index.js und fügen den folgenden Code ein:

```javascript
import { createRouter, createWebHistory } from "vue-router";

import HomeView from "../views/HomeView.vue";

const router = createRouter({
    history: createWebHistory(import.meta.env.VITE_BASE_URL),
    routes: [
        {
            path: "/",
            name: "home",
            component: HomeView,
        },
    ],
});

export default router;
```

5. Erstelle den resources/js/views Ordner und fügen die Dateien resources/js/views/HomeView.vue und resources/js/views/DashboardView.vue hinzu.

HomeView.vue

```vue
<template>
    <div>
        <h1>Home View</h1>
    </div>
</template>
```

6. Wir müssen das Einstiegspunkt-Template in der resources/views/app.blade.php anpassen:

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />

        <title>Laravel</title>
        @vite('resources/js/app.js', 'resources/css/app.css')
    </head>

    <body>
        <div id="app"></div>
    </body>
</html>
```

Info: Das Welcome-Template kann entfernt werden. Der wichtigste Teil ist die @vite-Direktive, die die Vite-Assets einbindet um das Mounten der Vue-App (id="app") zu ermöglichen.

7. Damit alle routes über die Vue-Router laufen, müssen wir einmalig in die routes/web.php folgendes ändern:

```php
Route::get('/{any}', function () {
    return view('app'); // Change this to the view that includes your Vue app
})->where('any', '^(?!api).*$');
```

Info: Die bestehende Route welcome kann entfernt werden.

8. Starte den Vite-Server und teste ob du die Vue-App im Browser sehen kannst `http://localhost`.

```bash
npm run dev
```

## Pinia (Vue Store) einrichten

Wir möchten über Pinia einen Store für die User-Authentifizierung einrichten. Wir erstellen dazu einen Store in der Datei resources/js/store/AuthStore.js.

```javascript
import { defineStore } from "pinia";
import axios from "axios";

export const authClient = axios.create({
    baseURL: import.meta.env.VITE_BASE_URL,
    withCredentials: true, // required to handle the CSRF token
});

export const useAuthStore = defineStore("AuthStore", {
    state: () => {
        return {
            user: null,
        };
    },

    actions: {
        async register(credentials) {
            try {
                await authClient.get("/sanctum/csrf-cookie");
                let response = await authClient.post(
                    "/api/register",
                    credentials
                );
                return response;
            } catch (error) {
                this.user = null;
                throw error;
            }
        },
        async login(credentials) {
            try {
                await authClient.get("/sanctum/csrf-cookie");
                let response = await authClient.post("/api/login", credentials);
                return response;
            } catch (error) {
                this.user = null;
                throw error;
            }
        },
        async getAuthUser() {
            try {
                await authClient.get("/sanctum/csrf-cookie");
                let response = await authClient.get("/api/user");
                this.user = response.data;
                return response;
            } catch (error) {
                this.user = null;
                console.error(error);
            }
        },

        async logout() {
            try {
                await authClient.get("/sanctum/csrf-cookie");
                let response = await authClient.post("/api/logout");
                this.user = null;
                return response;
            } catch (error) {
                this.user = null;
                throw error;
            }
        },
    },

    // get Elements from store
    getters: {
        authUser: (state) => state.user,
    },
});
```

## Vue Routes schützen

Wir möchten die Vue-Routes schützen, sodass nur authentifizierte User auf die Views zugreifen können. Wir können ganz einfach die Vue-Router Navigation verwenden.
Unter resources/js/router/index.js fügen wir folgenden Code hinzu:

1. Wir erweitern die Router mit einer Dashboard-Route und zusätzlichen Meta-Information für den Zugriffsschutz und erstellen auch eine Login-Route.

```javascript
const router = createRouter({
    history: createWebHistory(import.meta.env.VITE_BASE_URL),
    routes: [
        {
            path: "/",
            name: "home",
            component: HomeView,
        },
        {
            path: "/dashoboard",
            name: "dashboard",
            // route level code-splitting
            // this generates a separate chunk (Dashboard.[hash].js) for this route
            // which is lazy-loaded when the route is visited.
            component: () => import("../views/DashboardView.vue"),
            // die Meta-Informationen verwenden wir um den Zugriff zu schützen
            meta: { requiresAuth: true },
        },
        {
            // Hier brauchen wir keine Meta-Informationen, da diese Route für nicht authentifizierte User zugänglich sein soll.
            path: "/login",
            name: "login",
            component: () => import("../views/LoginView.vue"),
        },
    ],
});
```

2. Direkt unterhalb des Router-Definitions fügen wir den Navigation Guard hinzu:

```javascript
// navigation guard
router.beforeEach(async (to, from, next) => {
    const { getAuthUser } = useAuthStore();
    const { authUser } = storeToRefs(useAuthStore());
    const reqAuth = to.matched.some((record) => record.meta.requiresAuth);

    if (reqAuth && !authUser.value) {
        await getAuthUser();
        if (!authUser.value) next("/login");
        next();
    } else {
        next(); // make sure to always call next()!
    }
});
```

3. Wir erstellen die Datei resources/js/views/LoginView.vue und fügen den folgenden Code ein:

```vue
<script setup>
// refs verwenden wir um die Daten zu binden um sie reaktiv zu machen
import { ref } from "vue";

// wir importieren den AuthStore damit wir die Authentifizierung durchführen können

import { useAuthStore } from "@/store/AuthStore";

// wir holen uns die login und getAuthUser Funktionen aus dem Store
const { login, getAuthUser } = useAuthStore(); /
// für das v-model binding, siehe unten im template und wir verwenden anschliessen im handleLogin die email.value
const email = ref("");
const password = ref("");

const handleLogin = async () => {
    // wir geben die email und password Werte an die login Funktion in unserem Store weiter
    await login({ email: email.value, password: password.value });

    // nach dem Login holen wir uns den authentifizierten User um ihn im Store zu speichern
    const res = await getAuthUser();

    // wenn der Status 200 ist, leiten wir den Benutzer auf die Dashboard-Seite weiter
    if (res.status === 200) router.push("/dashboard");
};
</script>
<template>
    <div>
        <h1>Login View</h1>
        <form @submit.prevent="handleLogin">
            <label for="email">Email:</label>
            <input type="text" v-model="email" />
            <label for="password">Password:</label>
            <input type="password" v-model="password" />
            <button type="submit">Login</button>
        </form>
    </div>
</template>
```

4. Wir erstellen die Datei resources/js/views/DashboardView.vue und fügen den folgenden Code ein:

```vue
<script setup>
import { storeToRefs } from "pinia";
import { useAuthStore } from "@/store/AuthStore";
import router from "@/router";
const { authUser } = storeToRefs(useAuthStore());
const { logout } = useAuthStore();

const handleLogout = () => {
    logout();
    router.push("/login");
};
</script>
<template>
    <div>
        <h1>Dashboard View</h1>
        <p>Welcome, {{ authUser.name }}</p>
        <button @click="handleLogout">Logout</button>
    </div>
</template>
```

5. Wir erweitern die Navigation in unserer App.vue um die Login-Route.

```vue
<template>
    <header>
        <div>
            <nav>
                <RouterLink to="/">Home</RouterLink>
                <RouterLink to="/dashboard">Dashboard</RouterLink>
                <RouterLink to="/login">Login</RouterLink>
            </nav>
        </div>
    </header>
    <RouterView />
</template>
```

Wenn du nun die Vue-App startest und auf die Dashboard-Route zugreifen möchtest, wirst du auf die Login-Route weitergeleitet. Nach dem Login kannst du auf die Dashboard-Route zugreifen.
