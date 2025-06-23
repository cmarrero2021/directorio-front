<template>
  <q-layout view="hHh lpR fFf">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-btn dense flat round icon="menu" @click="toggleLeftDrawer" />

        <q-toolbar-title>
          <q-avatar>
            <img src="https://cdn.quasar.dev/logo-v2/svg/logo-mono-white.svg" />
          </q-avatar>
          Directorio de Revistas Científicas Venezolanas
        </q-toolbar-title>

        <q-space />

        <q-btn flat round dense icon="logout" @click="logout" />
      </q-toolbar>
    </q-header>

    <q-drawer show-if-above v-model="leftDrawerOpen" side="left" elevated>
      <!-- Menú basado en permisos -->
      <q-list>
        <q-item-label header>Menú Principal</q-item-label>

        <q-item clickable v-ripple to="/inicio">
          <q-item-section avatar>
            <q-icon name="home" />
          </q-item-section>
          <q-item-section>Inicio</q-item-section>
        </q-item>
        <q-item
          clickable
          v-ripple
          to="/admin"
          v-if="hasPermission('view_admin')"
        >
          <q-item-section avatar>
            <q-icon name="menu_book" />
          </q-item-section>
          <q-item-section>Revistas</q-item-section>
        </q-item>

        <!-- Agrega más items según los permisos -->
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { useRouter } from "vue-router";
import { LocalStorage, Notify } from "quasar";
import axios from "axios";

const leftDrawerOpen = ref(false);
const router = useRouter();
const logoutUrl = import.meta.env.VITE_LOGOUT_URL;

const toggleLeftDrawer = () => {
  leftDrawerOpen.value = !leftDrawerOpen.value;
};

const hasPermission = (permissionName) => {
  const permissions = LocalStorage.getItem("permissions") || [];
  return permissions.some((p) => p.name === permissionName);
};

let sessionTimeoutId = null;

const checkSessionExpiration = () => {
  const expiration = LocalStorage.getItem("sessionExpiration");
  if (expiration && Date.now() > expiration) {
    Notify.create({
      message: "El tiempo de sesión ha expirado. Por favor, vuelva a ingresar.",
      color: "negative",
      position: "top",
      timeout: 4000,
    });
    logout();
  } else if (expiration) {
    // Ejecutar logout 10 segundos antes del vencimiento real
    let remaining = expiration - Date.now() - 10000;
    if (remaining < 0) remaining = 0;
    if (sessionTimeoutId) clearTimeout(sessionTimeoutId);
    sessionTimeoutId = setTimeout(() => {
      Notify.create({
        message:
          "El tiempo de sesión ha expirado. Por favor, vuelva a ingresar.",
        color: "negative",
        position: "top",
        timeout: 4000,
      });
      logout();
    }, remaining);
  }
};

const logout = async () => {
  const token = LocalStorage.getItem("token"); // <-- Obtén el token antes de limpiar
  try {
    // Llama al endpoint de logout en el backend SOLO si hay token
    if (token) {
      await axios.post(
        logoutUrl,
        {},
        {
          headers: {
            Authorization: `Bearer ${token}`,
            "Content-Type": "application/json",
          },
        }
      );
    }
  } catch (error) {
    // No es necesario notificar aquí, ya que igual se limpiará el storage
    console.error("Error al cerrar sesión:", error);
  } finally {
    // Ahora sí limpia el almacenamiento local y timeout
    LocalStorage.remove("token");
    LocalStorage.remove("permissions");
    LocalStorage.remove("sessionDuration");
    LocalStorage.remove("role");
    LocalStorage.remove("sessionExpiration");
    if (sessionTimeoutId) clearTimeout(sessionTimeoutId);

    Notify.create({
      message: "Sesión cerrada correctamente",
      color: "positive",
    });

    router.push("/login");
  }
};

onMounted(() => {
  checkSessionExpiration();
});

onBeforeUnmount(() => {
  if (sessionTimeoutId) clearTimeout(sessionTimeoutId);
});
</script>
