<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';
import { toast } from 'vue3-toastify';

const props = defineProps<{
  professional: any;
}>();

const emit = defineEmits<{
  close: [];
  save: [data: any];
}>();

// Estados del formulario basados en la estructura real de datos
const formData = ref({
  id: props.professional.id,
  names: props.professional.names || props.professional.name || '',
  lastnames: props.professional.lastnames || '',
  email: props.professional.email || '',
  document: props.professional.document || '',
  phone: props.professional.phone || '',
  experience: props.professional.experience || 0,
  perfilPhoto: props.professional.perfilPhoto || '', // Campo correcto de la BD
  title: props.professional.title || '',
  description: props.professional.description || '',
  specialists: props.professional.specialists || [],
  offices: props.professional.offices || [],
  prepagadas: props.professional.prepagadas || [],
  web: props.professional.web || '',
  facebook: props.professional.facebook || '',
  instagram: props.professional.instagram || '',
  linkedin: props.professional.linkedin || '',
  youtube: props.professional.youtube || '',
  type_of_payment: props.professional.type_of_payment || 'CLINIC',
  actions: props.professional.actions || []
});

// Estados para campos de acción
const newSpecialty = ref('');
const selectedPrepaid = ref('');
const loadingImage = ref(false);

// URL de la imagen con fallback correcto
const profileImageUrl = computed(() => {
  const url = formData.value.perfilPhoto || props.professional.perfilPhoto;
  
  if (!url || url === '') {
    return 'https://res.cloudinary.com/dirsusbyy/image/upload/v1742425056/fidungtrcbetkco1tqqz.png';
  }
  
  // Si la URL es relativa, añadir el dominio base si es necesario
  if (url.startsWith('/')) {
    return window.location.origin + url;
  }
  
  return url;
});

// Imagen temporal para preview
const tempImageUrl = ref('');

onMounted(() => {
  // Inicializar la imagen temporal con la URL actual
  tempImageUrl.value = profileImageUrl.value;
  
  // Log para debug
  console.log('Professional data:', props.professional);
  console.log('perfilPhoto field:', props.professional.perfilPhoto);
  console.log('Image URL:', profileImageUrl.value);
});

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
async function handleFileUpload(event: Event) {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  
  if (!file) return;
  
  // Validar tamaño del archivo (máximo 5MB)
  if (file.size > 5 * 1024 * 1024) {
    toast.error('La imagen no debe superar los 5MB');
    return;
  }
  
  // Validar tipo de archivo
  const validTypes = ['image/jpeg', 'image/jpg', 'image/png', 'image/webp'];
  if (!validTypes.includes(file.type)) {
    toast.error('Solo se permiten imágenes JPG, PNG o WEBP');
    return;
  }
  
  loadingImage.value = true;
  
  try {
    const formDataUpload = new FormData();
    formDataUpload.append('file', file);
    
    // Subir imagen al servidor usando el endpoint correcto del repositorio
    const response = await axios.post('/files/upload', formDataUpload, {
      headers: { 
        'Content-Type': 'multipart/form-data' 
      }
    });
    
    if (response.data.url) {
      // Actualizar la URL de la imagen
      tempImageUrl.value = response.data.url;
      formData.value.perfilPhoto = response.data.url;
      
      toast.success('Imagen cargada correctamente');
    } else {
      throw new Error('No se recibió URL de la imagen');
    }
    
  } catch (error) {
    console.error('Error al cargar la imagen:', error);
    toast.error('Error al cargar la imagen. Por favor, intente nuevamente.');
    
    // Fallback: usar FileReader para preview local
    const reader = new FileReader();
    reader.onload = (e) => {
      const result = e.target?.result as string;
      tempImageUrl.value = result;
      // No actualizamos formData.perfilPhoto porque no se subió al servidor
    };
    reader.readAsDataURL(file);
  } finally {
    loadingImage.value = false;
  }
}

function addSpecialty() {
  if (newSpecialty.value.trim()) {
    if (!formData.value.specialists) {
      formData.value.specialists = [];
    }
    
    // Verificar si ya existe
    const exists = formData.value.specialists.some(
      s => s.name?.toLowerCase() === newSpecialty.value.trim().toLowerCase()
    );
    
    if (!exists) {
      formData.value.specialists.push({
        id: Date.now().toString(),
        name: newSpecialty.value.trim(),
        status: 'ACTIVE'
      });
      newSpecialty.value = '';
    } else {
      toast.warning('Esta especialidad ya fue agregada');
    }
  }
}

function removeSpecialty(index: number) {
  formData.value.specialists?.splice(index, 1);
}

function addPrepaid() {
  if (!selectedPrepaid.value) return;
  
  if (!formData.value.prepagadas) {
    formData.value.prepagadas = [];
  }
  
  const exists = formData.value.prepagadas.some(
    p => p.name === selectedPrepaid.value
  );
  
  if (!exists) {
    formData.value.prepagadas.push({
      id: Date.now().toString(),
      name: selectedPrepaid.value,
      status: 'ACTIVE',
      type: 'SITE'
    });
    selectedPrepaid.value = '';
  }
}

function removePrepaid(index: number) {
  formData.value.prepagadas?.splice(index, 1);
}

function addOffice() {
  if (!formData.value.offices) {
    formData.value.offices = [];
  }
  
  formData.value.offices.push({
    id: Date.now().toString(),
    description: 'Nuevo Consultorio',
    address: 'Cl. 140 #10 A 61, Bogotá, Colombia',
    latitude: 4.7110,
    longitude: -74.0721
  });
}

function removeOffice(index: number) {
  formData.value.offices?.splice(index, 1);
}

function downloadPDF() {
  // Lógica para descargar PDF
  window.open('/api/professionals/pdf/' + props.professional.id, '_blank');
}

async function saveChanges() {
  try {
    // Preparar datos para guardar
    const dataToSave = {
      ...formData.value,
      perfilPhoto: formData.value.perfilPhoto || tempImageUrl.value
    };
    
    // Emitir evento con los datos actualizados
    emit('save', dataToSave);
    
    toast.success('Cambios guardados correctamente');
  } catch (error) {
    console.error('Error al guardar:', error);
    toast.error('Error al guardar los cambios');
  }
}

function close() {
  emit('close');
}

// Función para manejar error de carga de imagen
function handleImageError(event: Event) {
  const target = event.target as HTMLImageElement;
  console.error('Error al cargar imagen desde:', target.src);
  // Usar imagen por defecto
  target.src = 'https://res.cloudinary.com/dirsusbyy/image/upload/v1742425056/fidungtrcbetkco1tqqz.png';
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
          Descargar PDF instructivo crear perfil
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
              readonly
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
          
          <!-- Teléfono -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Celular</label>
            <input 
              v-model="formData.phone"
              type="text" 
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
            >
          </div>
          
          <!-- Foto de Perfil - MEJORADA CON ENDPOINT CORRECTO -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Foto de Perfil</label>
            <div class="flex items-center gap-4">
              <!-- Contenedor de la imagen -->
              <div class="relative">
                <div class="w-24 h-24 rounded-full overflow-hidden bg-gray-100 border-2 border-[var(--blue-1)]">
                  <img 
                    :src="tempImageUrl || profileImageUrl" 
                    alt="Foto de perfil"
                    class="w-full h-full object-cover"
                    @error="handleImageError"
                  >
                </div>
                <!-- Indicador de carga -->
                <div v-if="loadingImage" class="absolute inset-0 bg-black bg-opacity-50 rounded-full flex items-center justify-center">
                  <div class="animate-spin rounded-full h-6 w-6 border-b-2 border-white"></div>
                </div>
              </div>
              
              <!-- Botón de carga -->
              <div class="flex flex-col gap-2">
                <label class="bg-[var(--blue-1)] hover:bg-blue-600 text-white px-4 py-2 rounded-lg cursor-pointer transition-colors text-center">
                  {{ loadingImage ? 'Cargando...' : 'Cambiar Foto' }}
                  <input 
                    type="file" 
                    accept="image/jpeg,image/jpg,image/png,image/webp"
                    class="hidden"
                    @change="handleFileUpload"
                    :disabled="loadingImage"
                  >
                </label>
                <span class="text-xs text-gray-500">JPG, PNG o WEBP (Max. 5MB)</span>
              </div>
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
          
          <!-- Descripción -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Descripción</label>
            <textarea 
              v-model="formData.description"
              rows="3"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
              placeholder="Describe tu experiencia y especialidades..."
            ></textarea>
          </div>
          
          <!-- Especialidades -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Especialidades</label>
            <div class="flex gap-2">
              <input 
                v-model="newSpecialty"
                type="text" 
                placeholder="Agregar especialidad..."
                class="flex-1 px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500"
                @keyup.enter="addSpecialty"
              >
              <button 
                @click="addSpecialty"
                class="bg-[var(--blue-1)] hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors"
              >
                Agregar
              </button>
            </div>
            
            <!-- Lista de especialidades -->
            <div class="mt-2 space-y-1">
              <div 
                v-for="(specialty, index) in formData.specialists" 
                :key="specialty.id || index"
                class="flex items-center justify-between bg-gray-100 px-3 py-2 rounded-lg"
              >
                <span class="text-sm">{{ specialty.name || specialty }}</span>
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
            <label class="block text-sm font-medium text-gray-700 mb-1">Consultorios</label>
            <button 
              @click="addOffice"
              class="w-full bg-[var(--blue-1)] hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors"
            >
              Agregar consultorio
            </button>
            
            <!-- Lista de consultorios -->
            <div class="mt-2 space-y-2">
              <div 
                v-for="(office, index) in formData.offices" 
                :key="office.id || index"
                class="bg-gray-100 p-3 rounded-lg"
              >
                <p class="text-sm font-medium">{{ office.description || `Consultorio ${index + 1}` }}</p>
                <p class="text-xs text-gray-600">{{ office.address || 'Dirección no especificada' }}</p>
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
              >
                {{ option }}
              </option>
            </select>
            
            <!-- Lista de prepagadas seleccionadas -->
            <div class="mt-2 flex flex-wrap gap-2">
              <div 
                v-for="(prepaid, index) in formData.prepagadas" 
                :key="prepaid.id || index"
                class="bg-[var(--blue-1)] text-white px-3 py-1 rounded-full flex items-center gap-2"
              >
                <span class="text-sm">{{ prepaid.name || prepaid }}</span>
                <button 
                  @click="removePrepaid(index)"
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
        class="bg-[var(--blue-1)] hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors"
        :disabled="loadingImage"
      >
        Guardar Cambios
      </button>
    </div>
  </div>
</template>

<style scoped>
/* Asegurar que la imagen se muestre correctamente */
img {
  image-rendering: -webkit-optimize-contrast;
  image-rendering: crisp-edges;
}

/* Estilo para el botón deshabilitado */
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>