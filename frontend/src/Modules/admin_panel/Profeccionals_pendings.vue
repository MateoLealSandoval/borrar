<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';
import paginadeComponent from '@/common/paginade.component.vue';
import axios from 'axios';
import Swal from 'sweetalert2';
import modal_Float from '@/components/modal_Float.vue';
import iconClose from '@/assets/imageIcons/admin_icons/icons8-close.webp'
import { store_admin_pendings } from '@/store/stores_admin_panel/pendings.store';

/**
 * ! icons 
 * **/
import iconedit from "@/assets/imageIcons/admin_icons/icons8-editar.webp"
import icondatas from "@/assets/imageIcons/admin_icons/documents.webp"
import icon_good from "@/assets/imageIcons/admin_icons/good_document.webp"
import excelIcon from "@/assets/imageIcons/admin_icons/excel-icon.webp"

import type { userPendingsDto } from '@/models/admin_panel/users_pendings';
type StatusType = 'PENDING' | 'ACCEPTED' | 'REJECTED';

/**
 * @COMPONENTES 
 * **/
import Documents_panel from './components/documents_panel.vue';

const adminProfessioanlStore = store_admin_pendings()
const type = ref<StatusType>('PENDING')
const searchTerm = ref('')

onMounted(() => {
    adminProfessioanlStore.get_all_users(type.value);
});

onUnmounted(() => {
    adminProfessioanlStore.reset();
});

const all_users = computed(() => adminProfessioanlStore.users || null);
const meta = computed(() => adminProfessioanlStore.meta || null);

// Función para filtrar usuarios
const filteredUsers = computed(() => {
    if (!all_users.value) return null;
    if (!searchTerm.value) return all_users.value;
    
    return all_users.value.filter(user => {
        const search = searchTerm.value.toLowerCase();
        return user.names?.toLowerCase().includes(search) ||
               user.lastnames?.toLowerCase().includes(search) ||
               user.email?.toLowerCase().includes(search);
    });
});

function formatDate(fechaIso: string): string {
    const fecha = new Date(fechaIso);
    const dia = fecha.getDate().toString().padStart(2, '0');
    const mes = (fecha.getMonth() + 1).toString().padStart(2, '0');
    const anio = fecha.getFullYear();
    return `${dia}/${mes}/${anio}`;
}

const selectItem = ref<userPendingsDto | null>(null);
const statusOptions: { value: StatusType; label: string }[] = [
    { value: 'PENDING', label: 'Pendiente' },
    { value: 'ACCEPTED', label: 'Aceptado' },
    { value: 'REJECTED', label: 'Rechazado' },
];

const panels = ref({
    viewDocuments: false,
    editModal: false,
});

watch(type, (newValue) => {
    if (newValue) {
        adminProfessioanlStore.reset();
        adminProfessioanlStore.get_all_users(newValue);
    }
});

// Función para descargar Excel con todos los pendientes
async function downloadExcel() {
    try {
        const response = await axios.post('/auth-pending/excel', { status: 'PENDING' }, {
            responseType: 'blob',
        });
        const blob = new Blob([response.data], { 
            type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' 
        });
        const url = window.URL.createObjectURL(blob);

        const link = document.createElement('a');
        link.href = url;
        link.setAttribute('download', `profesionales_pendientes_${new Date().toISOString().split('T')[0]}.xlsx`);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        window.URL.revokeObjectURL(url);
        
        Swal.fire({
            title: "¡Éxito!",
            text: "Base de datos descargada correctamente.",
            icon: "success",
            confirmButtonColor: "var(--blue-1)"
        });
    } catch (error) {
        console.error('Error al descargar el archivo:', error);
        Swal.fire({
            title: "Error",
            text: "No se pudo descargar la base de datos.",
            icon: "error",
            confirmButtonColor: "var(--blue-1)"
        });
    }
}

// Función para aprobar perfil
const approve = async (data: userPendingsDto) => {
    const result = await Swal.fire({
        title: "¿Estás seguro?",
        text: `¿Quieres aprobar a ${data.names} ${data.lastnames}?`,
        icon: "question",
        showCancelButton: true,
        confirmButtonText: "Sí, aprobar",
        cancelButtonText: "Cancelar",
        confirmButtonColor: "var(--blue-1)",
        cancelButtonColor: "#d33"
    });

    if (result.isConfirmed) {
        try {
            const response_axios = await axios.put(`/auth-pending/approve/${data.id}`)
  
            Swal.fire({
                title: "Aprobado",
                text: "El profesional fue aprobado correctamente.",
                icon: "success",
                confirmButtonColor: "var(--blue-1)"
            });

            // Actualizar la lista
            adminProfessioanlStore.get_all_users(type.value);
        } catch (error) {
            console.error("Error al aprobar:", error);
            Swal.fire({
                title: "Error",
                text: "No se pudo aprobar al profesional.",
                icon: "error",
                confirmButtonColor: "var(--blue-1)"
            });
        }
    }
};
</script>

<template>
    <!-- Modal Ver Documentos - Usando modal_Float como en el original -->
    <modal_Float v-if="panels.viewDocuments && selectItem" @close="panels.viewDocuments = false">
        <template v-slot:modal>
            <Documents_panel :data="selectItem" />
        </template>
    </modal_Float>

    <!-- Modal Ver/Editar Perfil - Usando modal_Float como en el original -->
    <modal_Float v-if="panels.editModal && selectItem" @close="panels.editModal = false">
        <template v-slot:modal>
            <div class="w-[90%] md:w-[600px] bg-white rounded-lg p-6">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-xl font-bold">Perfil del Profesional</h2>
                    <img :src="iconClose" alt="close" @click="panels.editModal = false"
                        class="w-6 h-6 cursor-pointer" />
                </div>
                
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Nombres</label>
                        <input v-model="selectItem.names" type="text"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Apellidos</label>
                        <input v-model="selectItem.lastnames" type="text"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Correo electrónico</label>
                        <input v-model="selectItem.email" type="email"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Teléfono</label>
                        <input v-model="selectItem.phone" type="text"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Documento</label>
                        <input v-model="selectItem.document" type="text"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Estado</label>
                        <input :value="selectItem.status" type="text" disabled
                            class="w-full px-3 py-2 border border-gray-300 rounded-md bg-gray-100">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Fecha de registro</label>
                        <input :value="formatDate(selectItem.createdAt)" type="text" disabled
                            class="w-full px-3 py-2 border border-gray-300 rounded-md bg-gray-100">
                    </div>
                </div>
            </div>
        </template>
    </modal_Float>

    <!-- Contenido principal -->
    <div class="h-screen bg-gray-100">
        <div class="bg-white rounded-lg h-[100%]">
            <!-- Header con título y controles -->
            <div class="p-6 border-b">                
                <div class="flex flex-wrap gap-4 items-center justify-between">
                    <!-- Buscador -->
                    <div class="flex-1 max-w-md">
                        <input 
                            v-model="searchTerm"
                            type="text" 
                            placeholder="Buscar por nombre o correo..."
                            class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <!-- Selector de estado y botón de descarga -->
                    <div class="flex gap-3">
                        <select v-model="type"
                            class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-[var(--blue-1)]">
                            <option v-for="option in statusOptions" :key="option.value" :value="option.value">
                                {{ option.label }}
                            </option>
                        </select>
                        
                        <button @click="downloadExcel"
                            class="flex items-center gap-2 px-4 py-2 bg-[var(--blue-1)] text-white rounded-lg hover:opacity-90"
                            v-tooltip="'Descargar BD de pendientes'">
                            <img :src="excelIcon" alt="excel" class="w-5 h-5" />
                            <span>Descargar BD</span>
                        </button>
                    </div>
                </div>
            </div>

            <!-- Tabla -->
            <div class="overflow-x-auto">
                <table class="min-w-full bg-white">
                    <thead class="bg-gray-50">
                        <tr>
                            <th class="px-2 md:px-6 py-3 text-left">Nombre</th>
                            <th class="px-2 md:px-6 py-3 text-left">Correo</th>
                            <th class="px-2 md:px-6 py-3 text-left">Fecha inscripción</th>
                            <th class="px-2 md:px-6 py-3 text-left">Acciones</th>
                        </tr>
                    </thead>
                    <tbody class="w-full">
                        <tr v-for="user in filteredUsers" :key="user.id" 
                            class="border-b hover:bg-gray-100"
                            :class="{ 'bg-green-100': user.status === 'ACCEPTED' }">
                            <td class="px-2 md:px-6 py-3 max-w-[20px] truncate text-ellipsis whitespace-nowrap">
                                {{ user.names }} {{ user.lastnames }}
                            </td>
                            <td class="px-2 md:px-6 py-3 max-w-[20px] truncate text-ellipsis whitespace-nowrap">
                                {{ user.email }}
                            </td>
                            <td class="px-2 md:px-6 py-3 max-w-[20px] truncate text-ellipsis whitespace-nowrap">
                                {{ formatDate(user.createdAt) }}
                            </td>
                            <td class="px-2 md:px-6 py-3">
                                <div class="flex gap-2">
                                    <!-- Ver documentos -->
                                    <img :src="icondatas" alt="documents" 
                                        class="w-7 h-7 hover:cursor-pointer"
                                        v-tooltip="'Ver documentos'"
                                        @click="() => { 
                                            panels.editModal = false; 
                                            panels.viewDocuments = true; 
                                            selectItem = user 
                                        }" />
                                    
                                    <!-- Ver/Editar perfil -->
                                    <img :src="iconedit" alt="edit view" 
                                        class="w-7 h-7 hover:cursor-pointer"
                                        v-tooltip="'Ver/Editar perfil'"
                                        @click="() => { 
                                            panels.viewDocuments = false; 
                                            panels.editModal = true; 
                                            selectItem = user 
                                        }" />
                                    
                                    <!-- Aprobar perfil -->
                                    <img v-if="user.status !== 'ACCEPTED'"
                                        :src="icon_good" alt="aprovar" 
                                        class="w-7 h-7 hover:cursor-pointer"
                                        v-tooltip="'Aprobar perfil'" 
                                        @click="approve(user)" />
                                </div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>

            <!-- Paginación -->
            <div class="flex justify-center mt-4 pb-4">
                <paginadeComponent v-if="meta" :meta="meta" @change-page="adminProfessioanlStore.goToPage" />
            </div>
        </div>
    </div>
</template>