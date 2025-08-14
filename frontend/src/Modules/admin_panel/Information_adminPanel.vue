<script setup lang="ts">
import { use_adminPanel_information } from '@/store/stores_admin_panel';
import { computed, onMounted } from 'vue';

const props = defineProps<{
  changePanel: (option: AdminPanelOptionEnum) => void;
}>();
const adminInformation = use_adminPanel_information();

/** 
 * Todo import images - YA NO SE NECESITAN, USAREMOS ICONOS SVG
 * **/
// Comentar o eliminar estos imports de imágenes
// import ProfessionalRegisters from "@/assets/imageIcons/profesionalesreg.png"
// import ImageRegistersUser from "@/assets/imageIcons/usuarioreg.png"
// etc...

/** 
 * Todo import components
 * **/
import itemInformation from './itemInformation.vue';
import { AdminPanelOptionEnum } from './Admin_types_panels';

onMounted(() => {
  adminInformation.getInformation();
});
const shedules_datas_office = computed(() => adminInformation.informationdata || null);
</script>

<template>
  <div class="w-full bg-gray-50 min-h-screen" v-if="shedules_datas_office != null">
    <!-- Header -->
    <div class="bg-white shadow-sm border-b border-gray-200">
      <h1 class="text-xl font-semibold text-gray-800 py-6 px-8">
        Bienvenido a Doc Visual Administrador
      </h1>
    </div>

    <!-- Dashboard Grid -->
    <div class="p-8">
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <itemInformation 
          :icon="'professionals'" 
          :value="shedules_datas_office.numberProfessionals"
          :label="'Profesionales Registrados'"
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.PROFESSIONAL_REGISTER)" 
        />
        <itemInformation 
          :icon="'patients'" 
          :value="shedules_datas_office.countUsers"
          :label="'Pacientes Registrados'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.USER_REGISTER)"
        />
        <itemInformation 
          :icon="'appointments'" 
          :value="shedules_datas_office.reservations"
          :label="'Citas Programadas'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.QUOTES)"
        />
        <itemInformation 
          :icon="'subscriptions'" 
          :value="shedules_datas_office.numberProfessionals"
          :label="'Suscripciones'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.SUBSCRIPTIONS)"
        />
        <itemInformation 
          :icon="'rating'" 
          :value="shedules_datas_office.numberProfessionals"
          :label="'Rating Profesionales'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.PROFESSIONAL_RATING)"
        />
        <itemInformation 
          :icon="'deleted_patients'" 
          :value="shedules_datas_office.deleteUsers"
          :label="'Pacientes Eliminados'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.DELETION_REQUESTS)"
          :color="'#EF4444'"
        />
        <itemInformation 
          :icon="'deleted_professionals'" 
          :value="shedules_datas_office.deleteUsersPartner"
          :label="'Profesionales Eliminados'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.DELETION_REQUESTS)"
          :color="'#EF4444'"
        />
        <itemInformation 
          :icon="'pending_approval'" 
          :value="shedules_datas_office.pendingPartner"
          :label="'Profesionales Por aprobar'" 
          :gotoPanel="() => changePanel(AdminPanelOptionEnum.PROFESSIONALS_PENDINGS)"
          :color="'#F59E0B'"
        />
      </div>
    </div>

    <!-- Footer con copyright -->
    <div class="mt-auto px-8 py-4 text-center text-sm text-gray-500 border-t border-gray-200 bg-white">
      <p>2025 DocVisual© Todos los derechos reservados</p>
    </div>
  </div>

  <!-- Loading state -->
  <div v-else class="w-full h-screen flex items-center justify-center bg-gray-50">
    <div class="text-center">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-[var(--blue-1)] mx-auto"></div>
      <p class="mt-4 text-gray-600">Cargando información...</p>
    </div>
  </div>
</template>