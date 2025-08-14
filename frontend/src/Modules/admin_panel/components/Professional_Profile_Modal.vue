<script setup lang="ts">
import { ref, watch, computed } from 'vue';
import type { UsersProfessionalsPanelAdminDto } from '@/dto/AdminPanel';

const props = defineProps<{
  professional: UsersProfessionalsPanelAdminDto;
}>();

const emit = defineEmits<{
  close: [];
  save: [data: any];
}>();

// Estados del formulario
const formData = ref({
  id: props.professional.id,
  names: props.professional.names || '',
  lastnames: props.professional.lastnames || '',
  email: props.professional.email || '',
  document: props.professional.document || '',
  phone: props.professional.phone || '',
  experience: props.professional.experience || 0,
  profilePhoto: props.professional.profilePhoto || null,
  specialties: props.professional.specialties || [],
  offices: props.professional.offices || [],
  prepaidMedicine: props.professional.prepaidMedicine || []
});

// Estados para campos de acción
const newSpecialty = ref('');
const selectedPrepaid = ref('');

// Foto de perfil
const profileImageUrl = ref(props.professional.profilePhoto || '');
const defaultProfileImage = '/src/assets/images/default-profile.png';

// Opciones de prepagadas
const prepaidOptions = [
  'Axa Colpatria',
  'Compensar',
  'Seguros Bolivar',
  'Sanitas',
  'Sura',
  'Nueva EPS',
  'Coomeva',
  'Famisanar'
];

// Métodos
function handleFileUpload(event: Event) {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      profileImageUrl.value = e.target?.result as string;
      formData.value.profilePhoto = e.target?.result as string;
    };
    reader.readAsDataURL(file);
  }
}

function addSpecialty() {
  if (newSpecialty.value.trim()) {
    if (!formData.value.specialties) {
      formData.value.specialties = [];
    }
    formData.value.specialties.push(newSpecialty.value.trim());
    newSpecialty.value = '';
  }
}

function removeSpecialty(index: number) {
  formData.value.specialties?.splice(index, 1);
}

function addPrepaid() {
  if (selectedPrepaid.value && !formData.value.prepaidMedicine?.includes(selectedPrepaid.value)) {
    if (!formData.value.prepaidMedicine) {
      formData.value.prepaidMedicine = [];
    }
    formData.value.prepaidMedicine.push(selectedPrepaid.value);
    selectedPrepaid.value = '';
  }
}

function removePrepaid(prepaid: string) {
  const index = formData.value.prepaidMedicine?.indexOf(prepaid);
  if (index !== undefined && index > -1) {
    formData.value.prepaidMedicine?.splice(index, 1);
  }
}

function addOffice() {
  // Lógica para agregar consultorio
  // Por ahora solo un placeholder
  if (!formData.value.offices) {
    formData.value.offices = [];
  }
  formData.value.offices.push({
    id: Date.now(),
    description: 'Nuevo Consultorio',
    address: 'Cl. 140 #10 A 61, Bogotá, Colombia'
  });
}

function removeOffice(index: number) {
  formData.value.offices?.splice(index, 1);
}

function downloadPDF() {
  // Lógica para descargar PDF
  window.open('/api/professionals/pdf/' + props.professional.id, '_blank');
}

function saveChanges() {
  emit('save', formData.value);
}

function close() {
  emit('close');
}
</script>

<template>
  <div class="bg-white rounded-lg h-full flex flex-col">
    <!-- Header -->
    <div class="px-6 py-4 border-b border-gray-200 flex justify-between items-center">
      <h2 class="text-xl font-semibold text-gray-800">Perfil del Profesional</h2>
      <div class="flex gap-3">
        <button 
          @click="downloadPDF"
          class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 transition-colors"
        >
          Descargar PDF
        </button>
        <button 
          @click="close"
          class="text-gray-500 hover:text-gray-700"
        >
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
          </svg>
        </button>
      </div>
    </div>
    
    <!-- Body - Scrollable -->
    <div class="flex-1 overflow-y-auto px-6 py-4">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <!-- Columna Izquierda -->
        <div class="space-y-4">
          <!-- Nombres -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Nombres</label>
            <input 
              v-model="formData.names"
              type="text" 
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
            >
          </div>
          
          <!-- Apellidos -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Apellidos</label>
            <input 
              v-model="formData.lastnames"
              type="text" 
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
            >
          </div>
          
          <!-- Correo electrónico -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Correo electrónico</label>
            <input 
              v-model="formData.email"
              type="email" 
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
            >
          </div>
          
          <!-- Cédula -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Cédula</label>
            <input 
              v-model="formData.document"
              type="text" 
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
            >
          </div>
          
          <!-- Foto de Perfil -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Foto de Perfil</label>
            <div class="flex items-center gap-4">
              <div class="w-24 h-24 rounded-full overflow-hidden bg-gray-100">
                <img 
                  :src="profileImageUrl || defaultProfileImage" 
                  alt="Perfil"
                  class="w-full h-full object-cover"
                >
              </div>
              <label class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg cursor-pointer transition-colors">
                Añadir Foto
                <input 
                  type="file" 
                  accept="image/*"
                  class="hidden"
                  @change="handleFileUpload"
                >
              </label>
            </div>
          </div>
        </div>
        
        <!-- Columna Derecha -->
        <div class="space-y-4">
          <!-- Años de experiencia -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Años de experiencia</label>
            <input 
              v-model.number="formData.experience"
              type="number" 
              min="0"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
            >
          </div>
          
          <!-- Principales campos de acción -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Principales campos de acción</label>
            <div class="flex gap-2">
              <input 
                v-model="newSpecialty"
                type="text" 
                placeholder="Escribe una acción..."
                class="flex-1 px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
                @keyup.enter="addSpecialty"
              >
              <button 
                @click="addSpecialty"
                class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors"
              >
                Agregar
              </button>
            </div>
            
            <!-- Lista de especialidades -->
            <div class="mt-2 space-y-1">
              <div 
                v-for="(specialty, index) in formData.specialties" 
                :key="index"
                class="flex items-center justify-between bg-gray-100 px-3 py-2 rounded-lg"
              >
                <span class="text-sm">{{ specialty }}</span>
                <button 
                  @click="removeSpecialty(index)"
                  class="text-red-500 hover:text-red-700"
                >
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
                  </svg>
                </button>
              </div>
            </div>
          </div>
          
          <!-- Ubicación Consultorio -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Ubicación Consultorio</label>
            <button 
              @click="addOffice"
              class="w-full bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors"
            >
              Nuevo consultorio
            </button>
            
            <!-- Lista de consultorios -->
            <div class="mt-2 space-y-2">
              <div 
                v-for="(office, index) in formData.offices" 
                :key="index"
                class="bg-gray-100 p-3 rounded-lg"
              >
                <p class="text-sm font-medium">{{ office.description }}</p>
                <p class="text-xs text-gray-600">{{ office.address }}</p>
                <button 
                  @click="removeOffice(index)"
                  class="text-red-500 hover:text-red-700 text-xs mt-1"
                >
                  Eliminar
                </button>
              </div>
            </div>
          </div>
          
          <!-- Convenios o prepagadas -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Convenios o prepagadas</label>
            <select 
              v-model="selectedPrepaid"
              @change="addPrepaid"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
            >
              <option value="">Seleccione una opción</option>
              <option 
                v-for="option in prepaidOptions" 
                :key="option"
                :value="option"
                :disabled="formData.prepaidMedicine?.includes(option)"
              >
                {{ option }}
              </option>
            </select>
            
            <!-- Lista de prepagadas seleccionadas -->
            <div class="mt-2 flex flex-wrap gap-2">
              <div 
                v-for="prepaid in formData.prepaidMedicine" 
                :key="prepaid"
                class="bg-blue-500 text-white px-3 py-1 rounded-full flex items-center gap-2"
              >
                <span class="text-sm">{{ prepaid }}</span>
                <button 
                  @click="removePrepaid(prepaid)"
                  class="text-white hover:text-red-200"
                >
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
                  </svg>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Footer -->
    <div class="px-6 py-4 border-t border-gray-200 flex justify-end gap-3">
      <button 
        @click="close"
        class="px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors"
      >
        Cancelar
      </button>
      <button 
        @click="saveChanges"
        class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors"
      >
        Guardar Cambios
      </button>
    </div>
  </div>
</template>