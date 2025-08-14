<script setup lang="ts">
import type { UsersProfessionalsPanelAdminDto } from '@/dto/AdminPanel';
import { store_admin_professionals } from '@/store/stores_admin_panel';
import { computed, onMounted, onUnmounted, ref } from 'vue';
import Swal from 'sweetalert2';
import axios from 'axios';
import paginadeComponent from '@/common/paginade.component.vue';
import modal_Float from '@/components/modal_Float.vue';
import Professional_Profile_Modal from './components/Professional_Profile_Modal.vue';
import Professional_Documents_Modal from './components/Professional_Documents_Modal.vue';

const adminProfessioanlStore = store_admin_professionals();

// Estados
const selectedProfessional = ref<any>(null);
const showProfileModal = ref(false);
const showDocumentsModal = ref(false);
const searchTerm = ref('');

// Lifecycle
onMounted(() => {
  adminProfessioanlStore.get_all_profesionals(); // Método correcto
});

onUnmounted(() => {
  adminProfessioanlStore.reset();
});

// Computed
const all_professionals = computed(() => adminProfessioanlStore.users || []);
const meta = computed(() => adminProfessioanlStore.meta || null);

const filteredProfessionals = computed(() => {
  if (!searchTerm.value) return all_professionals.value;
  
  return all_professionals.value.filter(prof => {
    const searchLower = searchTerm.value.toLowerCase();
    return (
      String(prof.names).toLowerCase().includes(searchLower) ||
      String(prof.lastnames).toLowerCase().includes(searchLower) ||
      String(prof.email).toLowerCase().includes(searchLower)
    );
  });
});

// Métodos
async function updateUserRole(user: UsersProfessionalsPanelAdminDto, checked: boolean) {
  if (user.role === 'DELETED_USER_PARTNER' && checked) {
    const result = await Swal.fire({
      title: "¡Alerta!",
      text: `¿Quieres habilitar al profesional ${user.names} ${user.lastnames}?`,
      icon: "warning",
      showCancelButton: true,
      confirmButtonColor: "var(--blue-1)",
      cancelButtonColor: "#d33",
      confirmButtonText: "Sí, habilitar",
      cancelButtonText: "Cancelar"
    });

    if (result.isConfirmed) {
      await adminProfessioanlStore.setState_professionals(user.id, 'USER_PARTNER');
      user.role = 'USER_PARTNER';
      Swal.fire('¡Habilitado!', 'El profesional ha sido habilitado.', 'success');
    }
    return;
  }

  if (user.role === 'USER_PARTNER' && !checked) {
    const result = await Swal.fire({
      title: "¡Alerta!",
      text: `¿Quieres desactivar al profesional ${user.names} ${user.lastnames}?`,
      icon: "warning",
      showCancelButton: true,
      confirmButtonColor: "var(--blue-1)",
      cancelButtonColor: "#d33",
      confirmButtonText: "Sí, desactivar",
      cancelButtonText: "Cancelar"
    });

    if (result.isConfirmed) {
      await adminProfessioanlStore.setState_professionals(user.id, 'DELETED_USER_PARTNER');
      user.role = 'DELETED_USER_PARTNER';
      Swal.fire('¡Desactivado!', 'El profesional ha sido desactivado.', 'success');
    }
    return;
  }
}

// Descargar base de datos
async function downloadDatabase() {
  try {
    const response = await axios.post('/professionals/excel', {}, {
      responseType: 'blob',
    });
    
    const blob = new Blob([response.data], { 
      type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' 
    });
    const url = window.URL.createObjectURL(blob);
    
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', `profesionales_registrados_${new Date().toISOString().split('T')[0]}.xlsx`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    window.URL.revokeObjectURL(url);
    
    Swal.fire('¡Descargado!', 'La base de datos ha sido descargada exitosamente.', 'success');
  } catch (error) {
    console.error('Error al descargar:', error);
    Swal.fire('Error', 'No se pudo descargar la base de datos.', 'error');
  }
}

// Ver perfil - Extender el tipo para incluir campos adicionales
function viewProfile(professional: any) {
  // Agregar campos faltantes con valores por defecto
  selectedProfessional.value = {
    ...professional,
    document: professional.document || '',
    phone: professional.phone || '',
    experience: professional.experience || 0,
    profilePhoto: professional.profilePhoto || null,
    specialties: professional.specialties || [],
    offices: professional.offices || [],
    prepaidMedicine: professional.prepaidMedicine || []
  };
  showProfileModal.value = true;
}

// Ver documentos
function viewDocuments(professional: any) {
  selectedProfessional.value = {
    ...professional,
    document: professional.document || '',
    phone: professional.phone || ''
  };
  showDocumentsModal.value = true;
}

// Cerrar modales
function closeProfileModal() {
  showProfileModal.value = false;
  selectedProfessional.value = null;
}

function closeDocumentsModal() {
  showDocumentsModal.value = false;
  selectedProfessional.value = null;
}

// Guardar cambios del perfil
async function saveProfileChanges(updatedData: any) {
  try {
    // Aquí iría la lógica para guardar los cambios
    await axios.put(`/professionals/${updatedData.id}`, updatedData);
    
    Swal.fire('¡Guardado!', 'Los cambios han sido guardados exitosamente.', 'success');
    closeProfileModal();
    adminProfessioanlStore.get_all_profesionals(); // Recargar datos
  } catch (error) {
    Swal.fire('Error', 'No se pudieron guardar los cambios.', 'error');
  }
}

// Paginación
function changePage(page: number) {
  adminProfessioanlStore.changePage(page);
}
</script>

<template>
  <div class="w-full bg-gray-50 min-h-screen">
    <!-- Header con título y acciones -->
    <div class="bg-white px-6 py-4 border-b border-gray-200">
      <div class="flex justify-between items-center">
        <h1 class="text-lg text-gray-700">Bienvenido a Doc Visual Administrador</h1>
        <div class="flex gap-4 items-center">
          <!-- Buscador -->
          <div class="relative">
            <input 
              v-model="searchTerm"
              type="text" 
              placeholder="Buscar profesional..."
              class="pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
            >
            <svg class="absolute left-3 top-2.5 w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/>
            </svg>
          </div>
          
          <!-- Botón descargar BD -->
          <button 
            @click="downloadDatabase"
            class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 transition-colors"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"/>
            </svg>
            Descargar BD
          </button>
        </div>
      </div>
    </div>

    <!-- Tabla -->
    <div class="p-6">
      <div class="bg-white rounded-lg shadow-sm overflow-hidden">
        <table class="min-w-full">
          <thead class="bg-gray-50 border-b border-gray-200">
            <tr>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Nombre
              </th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Documento
              </th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Celular
              </th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Email
              </th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Status
              </th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Acción
              </th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-200">
            <tr v-for="professional in filteredProfessionals" :key="professional.id" class="hover:bg-gray-50">
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                {{ professional.names }} {{ professional.lastnames }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                {{ (professional as any).document || '98765456' }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                {{ (professional as any).phone || '3196578900' }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                {{ professional.email }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap">
                <!-- Toggle Status -->
                <label class="relative inline-flex items-center cursor-pointer">
                  <input 
                    type="checkbox" 
                    class="sr-only peer" 
                    :checked="professional.role === 'USER_PARTNER'"
                    @click.prevent="updateUserRole(professional, professional.role !== 'USER_PARTNER')"
                  >
                  <div class="w-11 h-6 bg-gray-200 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"></div>
                </label>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm">
                <div class="flex gap-2">
                  <!-- Botón Ver/Editar Perfil -->
                  <button 
                    @click="viewProfile(professional)"
                    class="p-2 bg-blue-100 hover:bg-blue-200 rounded-lg transition-colors group"
                    title="Ver/Editar Perfil"
                  >
                    <svg class="w-5 h-5 text-blue-600 group-hover:text-blue-700" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"/>
                    </svg>
                  </button>
                  
                  <!-- Botón Ver Documentos -->
                  <button 
                    @click="viewDocuments(professional)"
                    class="p-2 bg-green-100 hover:bg-green-200 rounded-lg transition-colors group"
                    title="Ver Documentos"
                  >
                    <svg class="w-5 h-5 text-green-600 group-hover:text-green-700" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                    </svg>
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
        
        <!-- Mensaje si no hay datos -->
        <div v-if="filteredProfessionals.length === 0" class="text-center py-8 text-gray-500">
          No se encontraron profesionales registrados
        </div>
      </div>
      
      <!-- Paginación -->
      <div v-if="meta" class="mt-4 flex justify-center">
        <paginadeComponent 
          :meta="meta"
          @onChange-page="changePage"
        />
      </div>
    </div>

    <!-- Modal de Perfil -->
    <modal_Float 
      v-if="showProfileModal && selectedProfessional" 
      :model-value="showProfileModal"
      :width-percent="80"
      :height-percent="90"
      @click-outside="closeProfileModal"
    >
      <Professional_Profile_Modal 
        :professional="selectedProfessional"
        @close="closeProfileModal"
        @save="saveProfileChanges"
      />
    </modal_Float>

    <!-- Modal de Documentos -->
    <modal_Float 
      v-if="showDocumentsModal && selectedProfessional" 
      :model-value="showDocumentsModal"
      :width-percent="70"
      :height-percent="80"
      @click-outside="closeDocumentsModal"
    >
      <Professional_Documents_Modal 
        :professional="selectedProfessional"
        @close="closeDocumentsModal"
      />
    </modal_Float>
  </div>
</template>