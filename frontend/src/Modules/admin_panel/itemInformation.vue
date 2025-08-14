<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
    gotoPanel: () => void;
    icon?: string;
    label: string;
    value: number;
}>();

// Iconos SVG simples
const iconMap: Record<string, string> = {
  professionals: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/>
    </svg>
  `,
  patients: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M16 11c1.66 0 2.99-1.34 2.99-3S17.66 5 16 5c-1.66 0-3 1.34-3 3s1.34 3 3 3zm-8 0c1.66 0 2.99-1.34 2.99-3S9.66 5 8 5C6.34 5 5 6.34 5 8s1.34 3 3 3zm0 2c-2.33 0-7 1.17-7 3.5V19h14v-2.5c0-2.33-4.67-3.5-7-3.5zm8 0c-.29 0-.62.02-.97.05 1.16.84 1.97 1.97 1.97 3.45V19h6v-2.5c0-2.33-4.67-3.5-7-3.5z"/>
    </svg>
  `,
  appointments: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M19 3h-1V1h-2v2H8V1H6v2H5c-1.11 0-1.99.9-1.99 2L3 19c0 1.1.89 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 16H5V8h14v11zM7 10h5v5H7z"/>
    </svg>
  `,
  subscriptions: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M20 4H4c-1.11 0-1.99.89-1.99 2L2 18c0 1.11.89 2 2 2h16c1.11 0 2-.89 2-2V6c0-1.11-.89-2-2-2zm0 14H4v-6h16v6zm0-10H4V6h16v2z"/>
    </svg>
  `,
  rating: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 17.27L18.18 21l-1.64-7.03L22 9.24l-7.19-.61L12 2 9.19 8.63 2 9.24l5.46 4.73L5.82 21z"/>
    </svg>
  `,
  deleted_patients: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm5 11H7v-2h10v2z"/>
    </svg>
  `,
  deleted_professionals: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm5 11H7v-2h10v2z"/>
    </svg>
  `,
  pending_approval: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
    </svg>
  `,
  default: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/>
    </svg>
  `
};

const currentIcon = computed(() => {
  if (props.icon && iconMap[props.icon]) {
    return iconMap[props.icon];
  }
  return iconMap.default;
});

// Color del círculo del icono según el tipo
const iconBgColor = computed(() => {
  if (props.icon === 'deleted_patients' || props.icon === 'deleted_professionals') {
    return '#fee2e2'; // rojo claro
  }
  if (props.icon === 'pending_approval') {
    return '#fef3c7'; // amarillo claro
  }
  return '#f3f4f6'; // gris claro por defecto
});

const iconColor = computed(() => {
  if (props.icon === 'deleted_patients' || props.icon === 'deleted_professionals') {
    return '#dc2626'; // rojo
  }
  if (props.icon === 'pending_approval') {
    return '#d97706'; // amarillo oscuro
  }
  return '#6b7280'; // gris
});
</script>

<template>
    <div 
        @click="gotoPanel"
        class="bg-white rounded-lg px-5 py-4 shadow-sm hover:shadow-md transition-shadow cursor-pointer border border-gray-100"
    >
        <div class="flex items-center justify-between">
            <!-- Icono en círculo -->
            <div 
                class="flex-shrink-0 w-12 h-12 rounded-full flex items-center justify-center"
                :style="{ backgroundColor: iconBgColor }"
            >
                <div class="w-6 h-6" :style="{ color: iconColor }" v-html="currentIcon"></div>
            </div>
            
            <!-- Texto en el centro -->
            <div class="flex-1 px-4">
                <p class="text-xs text-gray-600 leading-tight">
                    {{ label }}
                </p>
            </div>
            
            <!-- Número a la derecha -->
            <div class="text-right flex-shrink-0">
                <p class="text-2xl font-semibold text-gray-800">{{ value }}</p>
            </div>
        </div>
    </div>
</template>