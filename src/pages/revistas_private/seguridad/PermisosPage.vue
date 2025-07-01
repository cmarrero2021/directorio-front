<template>
  <div class="q-pa-md">
    <q-card class="q-mb-md permisos-header-card">
      <q-card-section>
        <div class="text-h6">Permisos del usuario</div>
      </q-card-section>
      <q-separator />
    </q-card>
    <div class="q-pa-md row items-start q-gutter-md">
      <q-col
        v-for="perm in allPermissions"
        :key="perm.id"
        cols="12"
        sm="6"
        md="4"
        lg="3"
        xl="2"
      >
        <q-card flat bordered class="q-pa-sm">
          <div class="text-subtitle2">{{ perm.description }}</div>
          <div class="text-caption text-grey-7">{{ perm.name }}</div>
          <q-toggle
            class="q-mt-sm"
            :model-value="selectedPermissions.includes(perm.name)"
            :label="
              selectedPermissions.includes(perm.name)
                ? 'Asignado'
                : 'No asignado'
            "
            color="primary"
            @update:model-value="togglePermission(perm.name, $event)"
          />
        </q-card>
      </q-col>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { LocalStorage } from "quasar";
import axios from "axios";

const allPermissions = ref([]);
const selectedPermissions = ref([]);

const loadUserPermissions = () => {
  const perms = LocalStorage.getItem("permissions") || [];
  selectedPermissions.value = perms.map((p) => p.name);
};

const loadAllPermissions = async () => {
  try {
    const url = import.meta.env.VITE_LS_PERMISSIONS_URL;
    const { data } = await axios.get(url);
    allPermissions.value = data;
  } catch (e) {
    allPermissions.value = [];
  }
};

const togglePermission = (permName, value) => {
  if (value) {
    if (!selectedPermissions.value.includes(permName)) {
      selectedPermissions.value.push(permName);
    }
  } else {
    selectedPermissions.value = selectedPermissions.value.filter(
      (name) => name !== permName
    );
  }
  LocalStorage.set(
    "permissions",
    selectedPermissions.value.map((name) => ({ name }))
  );
};

onMounted(() => {
  loadUserPermissions();
  loadAllPermissions();
});
</script>

<style scoped>
.permisos-header-card {
  /* max-width: 1200px; */
  margin: 0 auto;
}
</style>
