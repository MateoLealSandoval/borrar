<script setup lang="ts">
import { use_adminPanel_information } from '@/store/stores_admin_panel';
import { computed, onMounted } from 'vue';

const props = defineProps<{
  changePanel: (option: AdminPanelOptionEnum) => void;
}>();
const adminInformation = use_adminPanel_information();

import itemInformation from './itemInformation.vue';
import { AdminPanelOptionEnum } from './Admin_types_panels';

onMounted(() => {
  adminInformation.getInformation();
});
const shedules_datas_office = computed(() => adminInformation.informationdata || null);
</script>

<template>
  <div class="w-full min-h-full" v-if="shedules_datas_office != null">
    <!-- Título sobre fondo gris -->
    <div class="px-8 py-6">
      <h1 class="text-lg text-gray-700">
        Bienvenido a Doc Visual Administrador
      </h1>
    </div>

    <!-- Dashboard Grid -->
    <div class="px-8">
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 max-w-7xl">
        <itemInformation 
          :icon="'professionals'" 
          :value="shedules_datas_office.numberProfessionals || 0"
          :label="'Profesionales Registrados'"
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.PROFESSIONAL_REGISTER)" 
        />
        <itemInformation 
          :icon="'patients'" 
          :value="shedules_datas_office.countUsers || 0"
          :label="'Pacientes Registrados'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.USER_REGISTER)"
        />
        <itemInformation 
          :icon="'appointments'" 
          :value="shedules_datas_office.reservations || 0"
          :label="'Citas Programadas'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.QUOTES)"
        />
        <itemInformation 
          :icon="'subscriptions'" 
          :value="shedules_datas_office.numberProfessionals || 0"
          :label="'Suscripciones'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.SUBSCRIPTIONS)"
        />
        <itemInformation 
          :icon="'rating'" 
          :value="shedules_datas_office.numberProfessionals || 0"
          :label="'Rating Profesionales'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.PROFESSIONAL_RATING)"
        />
        <itemInformation 
          :icon="'deleted_patients'" 
          :value="shedules_datas_office.deleteUsers || 0"
          :label="'Pacientes Eliminados'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.DELETION_REQUESTS)"
        />
        <itemInformation 
          :icon="'deleted_professionals'" 
          :value="shedules_datas_office.deleteUsersPartner || 0"
          :label="'Profesionales Eliminados'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.DELETION_REQUESTS)"
        />
        <itemInformation 
          :icon="'pending_approval'" 
          :value="shedules_datas_office.pendingPartner || 0"
          :label="'Profesionales Por aprobar'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.PROFESSIONALS_PENDINGS)"
        />
      </div>
    </div>

    <!-- Footer -->
    <div class="mt-16 px-8 py-4 text-center text-xs text-gray-500">
      <p>2025 DocVisual® Todos los derechos reservados</p>
    </div>
  </div>

  <!-- Loading state -->
  <div v-else class="w-full h-full flex items-center justify-center">
    <div class="text-center">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-[var(--blue-1)] mx-auto"></div>
      <p class="mt-4 text-gray-600">Cargando información...</p>
    </div>
  </div>
</template>