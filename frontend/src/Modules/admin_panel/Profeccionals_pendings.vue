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
const searchQuery = ref('')

onMounted(() => {
    adminProfessioanlStore.get_all_users(type.value);
});

onUnmounted(() => {
    adminProfessioanlStore.reset();
});

const all_users = computed(() => {
    const users = adminProfessioanlStore.users || [];
    
    if (!searchQuery.value) return users;
    
    const search = searchQuery.value.toLowerCase();
    return users.filter(user => 
        user.names?.toLowerCase().includes(search) ||
        user.lastnames?.toLowerCase().includes(search) ||
        user.email?.toLowerCase().includes(search) ||
        user.document?.toLowerCase().includes(search)
    );
});

const meta = computed(() => adminProfessioanlStore.meta || null);

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

// Función para descargar Excel
async function downloadExcel() {
    try {
        const response = await axios.post('/auth-pending/excel', { status: type.value }, {
            responseType: 'blob',
        });
        const blob = new Blob([response.data], { 
            type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' 
        });
        const url = window.URL.createObjectURL(blob);

        const link = document.createElement('a');
        link.href = url;
        link.setAttribute('download', `profesionales_pendientes.xlsx`);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        window.URL.revokeObjectURL(url);
    } catch (error) {
        console.error('Error al descargar el archivo:', error);
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
            await axios.put(`/auth-pending/approve/${data.id}`)
  
            Swal.fire({
                title: "Aprobado",
                text: "El profesional fue aprobado correctamente.",
                icon: "success",
                confirmButtonColor: "var(--blue-1)"
            });

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

// Función para actualizar el item seleccionado
const update_item = async () => {
    try {
        if (selectItem.value) {
            await axios.put(`/auth-pending/${selectItem.value.id}`, {
                names: selectItem.value.names,
                lastnames: selectItem.value.lastnames,
                email: selectItem.value.email,
                phone: selectItem.value.phone,
                document: selectItem.value.document
            });

            Swal.fire({
                title: "¡Éxito!",
                text: "El profesional ha sido actualizado correctamente.",
                icon: "success",
                confirmButtonColor: "var(--blue-1)"
            });

            selectItem.value = null;
            panels.value.editModal = false;
            adminProfessioanlStore.get_all_users(type.value);
        }
    } catch (error) {
        console.error("Error al actualizar:", error);
        Swal.fire({
            title: "Error",
            text: "No se pudo actualizar el profesional.",
            icon: "error",
            confirmButtonColor: "var(--blue-1)"
        });
    }
};
</script>

<template>
    <!-- Modal Ver Documentos -->
    <modal_Float :model-value="panels.viewDocuments && selectItem != null" 
        :width-percent="90" 
        :height-percent="90"
        @click-outside="() => { panels.viewDocuments = false; selectItem = null }" 
        v-if="panels.viewDocuments && selectItem">
        <Documents_panel :data="selectItem" />
    </modal_Float>

    <!-- Modal Ver/Editar Perfil -->
    <modal_Float :model-value="panels.editModal && selectItem != null" 
        :width-percent="80" 
        :height-percent="80"
        @click-outside="() => { panels.editModal = false; selectItem = null }" 
        v-if="panels.editModal && selectItem">
        <div class="w-[98%] h-full flex flex-col bg-white rounded-lg p-6">
            <div class="w-full flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold">Perfil del Profesional</h2>
                <img :src="iconClose" alt="close" 
                    class="w-6 h-6 cursor-pointer" 
                    @click="() => { panels.editModal = false; selectItem = null }" />
            </div>
            
            <div class="w-[90%] m-auto my-2 flex-grow overflow-auto space-y-4">
                <div>
                    <h1 class="font-semibold text-black text-lg mb-1">Nombres</h1>
                    <input v-model="selectItem.names" type="text" 
                        placeholder="Nombres"
                        class="w-full p-2 rounded-md border border-gray-300 focus:border-[var(--blue-1)] focus:outline-none" />
                </div>
                
                <div>
                    <h1 class="font-semibold text-black text-lg mb-1">Apellidos</h1>
                    <input v-model="selectItem.lastnames" type="text" 
                        placeholder="Apellidos"
                        class="w-full p-2 rounded-md border border-gray-300 focus:border-[var(--blue-1)] focus:outline-none" />
                </div>
                
                <div>
                    <h1 class="font-semibold text-black text-lg mb-1">Correo</h1>
                    <input v-model="selectItem.email" type="email" 
                        placeholder="Correo electrónico"
                        class="w-full p-2 rounded-md border border-gray-300 focus:border-[var(--blue-1)] focus:outline-none" />
                </div>
                
                <div>
                    <h1 class="font-semibold text-black text-lg mb-1">Documento</h1>
                    <input v-model="selectItem.document" type="text" 
                        placeholder="Documento"
                        class="w-full p-2 rounded-md border border-gray-300 focus:border-[var(--blue-1)] focus:outline-none" />
                </div>
                
                <div>
                    <h1 class="font-semibold text-black text-lg mb-1">Teléfono</h1>
                    <input v-model="selectItem.phone" type="text" 
                        placeholder="Teléfono"
                        class="w-full p-2 rounded-md border border-gray-300 focus:border-[var(--blue-1)] focus:outline-none" />
                </div>
                
                <div>
                    <h1 class="font-semibold text-black text-lg mb-1">Estado</h1>
                    <input :value="selectItem.status" type="text" 
                        disabled
                        class="w-full p-2 rounded-md border border-gray-300 bg-gray-100" />
                </div>
                
                <div>
                    <h1 class="font-semibold text-black text-lg mb-1">Fecha de registro</h1>
                    <input :value="formatDate(selectItem.createdAt)" type="text" 
                        disabled
                        class="w-full p-2 rounded-md border border-gray-300 bg-gray-100" />
                </div>
            </div>
            
            <div @click="update_item"
                class="w-[90%] mx-auto mb-4 bg-[var(--blue-1)] rounded-2xl mt-auto cursor-pointer hover:opacity-90">
                <h1 class="py-3 text-center text-white font-semibold">Actualizar</h1>
            </div>
        </div>
    </modal_Float>

    <!-- Contenido principal -->
    <div class="w-full h-full bg-gray-100">
        <!-- Header con título y controles -->
        <div class="py-10 w-[90%] mx-auto flex justify-between items-center">
            <h1>Bienvenido a Doc Visual Administrador</h1>
            
            <div class="flex items-center gap-4">
                <!-- Buscador -->
                <div class="flex items-center gap-2">
                    <input 
                        v-model="searchQuery"
                        type="text" 
                        placeholder="Buscar..."
                        class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-[var(--blue-1)]">
                    
                    <!-- Selector de estado -->
                    <select v-model="type"
                        class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-[var(--blue-1)]">
                        <option v-for="option in statusOptions" :key="option.value" :value="option.value">
                            {{ option.label }}
                        </option>
                    </select>
                </div>
                
                <!-- Botón descargar Excel -->
                <div @click="downloadExcel" class="cursor-pointer">
                    <img :src="excelIcon" alt="excel" class="mx-auto w-10 h-auto" />
                    <h1 class="md:text-base text-[10px] text-center">Descargar datos</h1>
                </div>
            </div>
        </div>

        <!-- Tabla -->
        <div class="w-[90%] mx-auto bg-white rounded-2xl shadow-gray-400 shadow-2xl" v-if="all_users">
            <div class="min-h-[400px] w-full">
                <table class="min-w-full bg-white shadow-md rounded-xl overflow-hidden">
                    <thead class="bg-gray-200 text-gray-500 text-left">
                        <tr>
                            <th class="px-6 py-3">Nombre</th>
                            <th class="px-6 py-3">Correo</th>
                            <th class="px-6 py-3">Fecha inscripción</th>
                            <th class="px-6 py-3">Acciones</th>
                        </tr>
                    </thead>
                    <tbody class="w-full">
                        <tr v-for="user in all_users" :key="user.id" 
                            class="border-b hover:bg-gray-50"
                            :class="{ 'bg-green-100': user.status === 'ACCEPTED' }">
                            <td class="px-6 py-3">
                                {{ user.names }} {{ user.lastnames }}
                            </td>
                            <td class="px-6 py-3">
                                {{ user.email }}
                            </td>
                            <td class="px-6 py-3">
                                {{ formatDate(user.createdAt) }}
                            </td>
                            <td class="px-6 py-3">
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