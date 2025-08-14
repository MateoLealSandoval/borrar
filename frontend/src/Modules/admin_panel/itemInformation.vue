<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
    gotoPanel: () => void;
    icon?: string;
    label: string;
    value: number;
}>();

// Mapeo de iconos SVG más simples y limpios
const iconMap: Record<string, string> = {
  professionals: `
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
      <circle cx="9" cy="7" r="4"></circle>
      <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
      <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
    </svg>
  `,
  patients: `
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path>
      <circle cx="12" cy="7" r="4"></circle>
    </svg>
  `,
  appointments: `
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
      <line x1="16" y1="2" x2="16" y2="6"></line>
      <line x1="8" y1="2" x2="8" y2="6"></line>
      <line x1="3" y1="10" x2="21" y2="10"></line>
    </svg>
  `,
  subscriptions: `
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <rect x="3" y="11" width="18" height="10" rx="2" ry="2"></rect>
      <path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
    </svg>
  `,
  rating: `
    <svg viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
    </svg>
  `,
  deleted_patients: `
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <circle cx="12" cy="12" r="10"></circle>
      <line x1="15" y1="9" x2="9" y2="15"></line>
      <line x1="9" y1="9" x2="15" y2="15"></line>
    </svg>
  `,
  deleted_professionals: `
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <circle cx="12" cy="12" r="10"></circle>
      <line x1="15" y1="9" x2="9" y2="15"></line>
      <line x1="9" y1="9" x2="15" y2="15"></line>
    </svg>
  `,
  pending_approval: `
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <circle cx="12" cy="12" r="10"></circle>
      <polyline points="12 6 12 12 16 14"></polyline>
    </svg>
  `,
  default: `
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <circle cx="12" cy="12" r="10"></circle>
      <line x1="12" y1="8" x2="12" y2="12"></line>
      <line x1="12" y1="16" x2="12.01" y2="16"></line>
    </svg>
  `
};

const currentIcon = computed(() => {
  if (props.icon && iconMap[props.icon]) {
    return iconMap[props.icon];
  }
  return iconMap.default;
});

// Color del icono según el tipo
const iconColor = computed(() => {
  if (props.icon === 'deleted_patients' || props.icon === 'deleted_professionals') {
    return '#ef4444'; // rojo
  }
  if (props.icon === 'pending_approval') {
    return '#f59e0b'; // amarillo
  }
  return '#6b7280'; // gris por defecto
});
</script>

<template>
    <div 
        @click="gotoPanel"
        class="bg-white rounded-lg p-4 shadow-sm hover:shadow-md transition-shadow cursor-pointer border border-gray-100"
    >
        <div class="flex items-center justify-between h-full">
            <!-- Lado izquierdo: Icono y texto -->
            <div class="flex items-start gap-3">
                <!-- Icono -->
                <div class="flex-shrink-0 w-10 h-10 rounded-lg bg-gray-50 flex items-center justify-center"
                     :style="{ color: iconColor }">
                    <div class="w-6 h-6" v-html="currentIcon"></div>
                </div>
                
                <!-- Texto debajo del icono pero en la misma fila -->
                <div class="flex flex-col justify-center">
                    <p class="text-xs text-gray-600 leading-tight">
                        <span v-for="(word, index) in label.split(' ')" :key="index" class="block">
                            {{ word }}
                        </span>
                    </p>
                </div>
            </div>
            
            <!-- Lado derecho: Número grande -->
            <div class="text-right">
                <p class="text-2xl font-bold text-gray-900">{{ value }}</p>
            </div>
        </div>
    </div>
</template>