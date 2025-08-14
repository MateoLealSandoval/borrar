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
    viewProfile: false,
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

// Función para actualizar datos del profesional
const updateProfessional = async () => {
    if (!selectItem.value) return;

    try {
        const response = await axios.put(`/auth-pending/${selectItem.value.id}`, {
            names: selectItem.value.names,
            lastnames: selectItem.value.lastnames,
            email: selectItem.value.email,
            phone: selectItem.value.phone,
            document: selectItem.value.document
        });

        Swal.fire({
            title: "¡Éxito!",
            text: "Los datos se actualizaron correctamente.",
            icon: "success",
            confirmButtonColor: "var(--blue-1)"
        });

        panels.value.viewProfile = false;
        adminProfessioanlStore.get_all_users(type.value);
    } catch (error) {
        console.error("Error al actualizar:", error);
        Swal.fire({
            title: "Error",
            text: "No se pudieron actualizar los datos.",
            icon: "error",
            confirmButtonColor: "var(--blue-1)"
        });
    }
};
</script>

<template>
    <!-- Modal Ver Documentos -->
    <modal_Float v-if="panels.viewDocuments && selectItem" @close="panels.viewDocuments = false">
        <template v-slot:modal>
            <div class="bg-white p-6 rounded-lg shadow-lg max-h-[80vh] overflow-auto">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-xl font-bold">Documentos del Profesional</h2>
                    <img :src="iconClose" alt="close" @click="panels.viewDocuments = false"
                        class="w-6 h-6 cursor-pointer" />
                </div>
                <Documents_panel :data="selectItem" />
            </div>
        </template>
    </modal_Float>

    <!-- Modal Ver/Editar Perfil -->
    <modal_Float v-if="panels.viewProfile && selectItem" @close="panels.viewProfile = false">
        <template v-slot:modal>
            <div class="bg-white p-6 rounded-lg shadow-lg w-[600px] max-h-[80vh] overflow-auto">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-xl font-bold">Perfil del Profesional</h2>
                    <img :src="iconClose" alt="close" @click="panels.viewProfile = false"
                        class="w-6 h-6 cursor-pointer" />
                </div>
                
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-medium mb-1">Nombres</label>
                        <input v-model="selectItem.names" type="text"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium mb-1">Apellidos</label>
                        <input v-model="selectItem.lastnames" type="text"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium mb-1">Correo electrónico</label>
                        <input v-model="selectItem.email" type="email"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium mb-1">Teléfono</label>
                        <input v-model="selectItem.phone" type="text"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium mb-1">Documento</label>
                        <input v-model="selectItem.document" type="text"
                            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:border-[var(--blue-1)]">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium mb-1">Estado</label>
                        <input :value="selectItem.status" type="text" disabled
                            class="w-full px-3 py-2 border border-gray-300 rounded-md bg-gray-100">
                    </div>
                    
                    <div>
                        <label class="block text-sm font-medium mb-1">Fecha de registro</label>
                        <input :value="formatDate(selectItem.createdAt)" type="text" disabled
                            class="w-full px-3 py-2 border border-gray-300 rounded-md bg-gray-100">
                    </div>
                    
                    <div class="flex justify-end gap-3 pt-4">
                        <button @click="panels.viewProfile = false"
                            class="px-4 py-2 bg-gray-300 text-gray-700 rounded-md hover:bg-gray-400">
                            Cancelar
                        </button>
                        <button @click="updateProfessional"
                            class="px-4 py-2 bg-[var(--blue-1)] text-white rounded-md hover:bg-opacity-90">
                            Guardar cambios
                        </button>
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
                <h1 class="text-2xl font-bold mb-4">Profesionales Pendientes</h1>
                
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
                            class="flex items-center gap-2 px-4 py-2 bg-[var(--blue-1)] text-white rounded-lg hover:bg-opacity-90"
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
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                Nombre
                            </th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                Correo
                            </th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                Fecha inscripción
                            </th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                Acciones
                            </th>
                        </tr>
                    </thead>
                    <tbody class="bg-white divide-y divide-gray-200">
                        <tr v-for="user in filteredUsers" :key="user.id" 
                            class="hover:bg-gray-50"
                            :class="{ 'bg-green-50': user.status === 'ACCEPTED' }">
                            <td class="px-6 py-4 whitespace-nowrap">
                                <div class="text-sm font-medium text-gray-900">
                                    {{ user.names }} {{ user.lastnames }}
                                </div>
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap">
                                <div class="text-sm text-gray-500">{{ user.email }}</div>
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap">
                                <div class="text-sm text-gray-500">{{ formatDate(user.createdAt) }}</div>
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap">
                                <div class="flex gap-2">
                                    <!-- Ver documentos -->
                                    <button @click="() => { 
                                        panels.viewProfile = false; 
                                        panels.viewDocuments = true; 
                                        selectItem = user 
                                    }"
                                        class="p-1.5 hover:bg-gray-100 rounded-lg transition-colors"
                                        v-tooltip="'Ver documentos'">
                                        <img :src="icondatas" alt="documents" class="w-6 h-6" />
                                    </button>
                                    
                                    <!-- Ver/Editar perfil -->
                                    <button @click="() => { 
                                        panels.viewDocuments = false; 
                                        panels.viewProfile = true; 
                                        selectItem = user 
                                    }"
                                        class="p-1.5 hover:bg-gray-100 rounded-lg transition-colors"
                                        v-tooltip="'Ver/Editar perfil'">
                                        <img :src="iconedit" alt="edit" class="w-6 h-6" />
                                    </button>
                                    
                                    <!-- Aprobar perfil -->
                                    <button v-if="user.status !== 'ACCEPTED'"
                                        @click="approve(user)"
                                        class="p-1.5 hover:bg-gray-100 rounded-lg transition-colors"
                                        v-tooltip="'Aprobar perfil'">
                                        <img :src="icon_good" alt="approve" class="w-6 h-6" />
                                    </button>
                                </div>
                            </td>
                        </tr>
                        
                        <!-- Mensaje cuando no hay datos -->
                        <tr v-if="!filteredUsers || filteredUsers.length === 0">
                            <td colspan="4" class="px-6 py-8 text-center text-gray-500">
                                No se encontraron profesionales
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>

            <!-- Paginación -->
            <div class="flex justify-center py-4 border-t">
                <paginadeComponent v-if="meta" :meta="meta" @change-page="adminProfessioanlStore.goToPage" />
            </div>
        </div>
    </div>
</template>