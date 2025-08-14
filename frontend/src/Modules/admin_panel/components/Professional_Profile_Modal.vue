<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import type { user_professional_dto } from '@/dto/professional';

// Props con tipo correcto
const props = defineProps<{
  professional: user_professional_dto | null;
  showModal: boolean;
}>();

// Emits
const emit = defineEmits<{
  close: [];
  save: [data: any];
}>();

// Estado del formulario
const formData = ref({
  id: props.professional?.id || '',
  names: props.professional?.names || '',
  lastnames: props.professional?.lastnames || '',
  email: props.professional?.email || '',
  document: props.professional?.document || '',
  phone: props.professional?.phone || '',
  experience: props.professional?.experience || 0,
  description: props.professional?.description || '',
  perfilPhoto: props.professional?.perfilPhoto || '',
  prepaidMedicine: props.professional?.prepaidMedicine || []
});

// Estado para manejo de errores de imagen
const imageError = ref(false);
const defaultImage = 'https://res.cloudinary.com/dirsusbyy/image/upload/v1742425056/fidungtrcbetkco1tqqz.png';

// Computed para la URL de la imagen
const imageUrl = computed(() => {
  if (imageError.value || !formData.value.perfilPhoto) {
    return defaultImage;
  }
  return formData.value.perfilPhoto;
});

// Watchers para actualizar el formulario cuando cambian las props
watch(() => props.professional, (newVal) => {
  if (newVal) {
    formData.value = {
      id: newVal.id || '',
      names: newVal.names || '',
      lastnames: newVal.lastnames || '',
      email: newVal.email || '',
      document: newVal.document || '',
      phone: newVal.phone || '',
      experience: newVal.experience || 0,
      description: newVal.description || '',
      perfilPhoto: newVal.perfilPhoto || '',
      prepaidMedicine: newVal.prepaidMedicine || []
    };
    // Reset del error de imagen cuando se carga un nuevo profesional
    imageError.value = false;
  }
});

// Métodos
const handleImageError = () => {
  imageError.value = true;
};

const handleImageUpload = async (event: Event) => {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  
  if (file) {
    // Validar tipo de archivo
    if (!file.type.startsWith('image/')) {
      alert('Por favor, selecciona una imagen válida');
      return;
    }
    
    // Validar tamaño (max 5MB)
    if (file.size > 5 * 1024 * 1024) {
      alert('La imagen no debe superar los 5MB');
      return;
    }
    
    try {
      // Aquí iría la lógica para subir la imagen
      // Por ahora solo simulamos con un FileReader
      const reader = new FileReader();
      reader.onload = (e) => {
        formData.value.perfilPhoto = e.target?.result as string;
        imageError.value = false;
      };
      reader.readAsDataURL(file);
    } catch (error) {
      console.error('Error al cargar la imagen:', error);
      alert('Error al cargar la imagen');
    }
  }
};

const restrictToInteger = (event: Event) => {
  const input = event.target as HTMLInputElement;
  let value = input.value;
  
  // Eliminar caracteres no numéricos
  value = value.replace(/[^0-9]/g, '');
  
  // Convertir a número y actualizar
  formData.value.experience = value ? parseInt(value, 10) : 0;
  
  // Actualizar el campo de entrada
  input.value = formData.value.experience.toString();
};

const save = () => {
  // Validaciones básicas
  if (!formData.value.names || !formData.value.lastnames) {
    alert('Por favor, completa todos los campos obligatorios');
    return;
  }
  
  emit('save', formData.value);
};

const close = () => {
  emit('close');
};
</script>

<template>
  <div v-if="showModal" class="fixed inset-0 bg-black/80 flex items-center justify-center z-50">
    <div class="bg-white rounded-lg shadow-xl w-[90%] md:w-[70%] lg:w-[60%] max-h-[90vh] overflow-y-auto">
      <!-- Header -->
      <div class="sticky top-0 bg-white border-b px-6 py-4 flex justify-between items-center">
        <h2 class="text-xl font-semibold text-gray-800">
          Editar Perfil Profesional
        </h2>
        <button 
          @click="close"
          class="text-gray-500 hover:text-gray-700 transition-colors"
        >
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- Body -->
      <div class="px-6 py-4">
        <!-- Foto de perfil -->
        <div class="flex flex-col items-center mb-6">
          <div class="relative w-32 h-32 mb-4">
            <img 
              :src="imageUrl"
              @error="handleImageError"
              alt="Foto de perfil"
              class="w-full h-full rounded-full object-cover border-4 border-gray-200"
            />
            <label class="absolute bottom-0 right-0 bg-blue-500 rounded-full p-2 cursor-pointer hover:bg-blue-600 transition-colors">
              <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z" />
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 13a3 3 0 11-6 0 3 3 0 016 0z" />
              </svg>
              <input 
                type="file" 
                @change="handleImageUpload"
                accept="image/*"
                class="hidden"
              />
            </label>
          </div>
        </div>

        <!-- Formulario -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <!-- Nombres -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Nombres *
            </label>
            <input 
              v-model="formData.names"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Ingrese nombres"
            />
          </div>

          <!-- Apellidos -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Apellidos *
            </label>
            <input 
              v-model="formData.lastnames"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Ingrese apellidos"
            />
          </div>

          <!-- Email -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Correo Electrónico
            </label>
            <input 
              v-model="formData.email"
              type="email"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="correo@ejemplo.com"
            />
          </div>

          <!-- Documento -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Documento
            </label>
            <input 
              v-model="formData.document"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Número de documento"
            />
          </div>

          <!-- Teléfono -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Teléfono
            </label>
            <input 
              v-model="formData.phone"
              type="tel"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Número de teléfono"
            />
          </div>

          <!-- Experiencia -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Años de Experiencia
            </label>
            <input 
              :value="formData.experience"
              @input="restrictToInteger"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="0"
            />
          </div>

          <!-- Descripción -->
          <div class="md:col-span-2">
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Descripción
            </label>
            <textarea 
              v-model="formData.description"
              rows="4"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Descripción profesional..."
            />
          </div>

          <!-- Medicina Prepagada -->
          <div class="md:col-span-2">
            <label class="block text-sm font-medium text-gray-700 mb-1">
              Medicina Prepagada
            </label>
            <div class="flex flex-wrap gap-2">
              <span 
                v-for="(item, index) in formData.prepaidMedicine" 
                :key="index"
                class="inline-flex items-center px-3 py-1 rounded-full text-sm bg-blue-100 text-blue-800"
              >
                {{ item }}
              </span>
              <span v-if="!formData.prepaidMedicine?.length" class="text-gray-500 text-sm">
                No hay medicina prepagada registrada
              </span>
            </div>
          </div>
        </div>
      </div>

      <!-- Footer -->
      <div class="sticky bottom-0 bg-white border-t px-6 py-4 flex justify-end gap-3">
        <button 
          @click="close"
          class="px-4 py-2 text-gray-700 bg-gray-200 rounded-md hover:bg-gray-300 transition-colors"
        >
          Cancelar
        </button>
        <button 
          @click="save"
          class="px-4 py-2 text-white bg-blue-500 rounded-md hover:bg-blue-600 transition-colors"
        >
          Guardar Cambios
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Animación de entrada para el modal */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.95);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.fixed > div {
  animation: fadeIn 0.2s ease-out;
}
</style>