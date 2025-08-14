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

onMounted(() => {
    adminProfessioanlStore.get_all_users(type.value);
});

onUnmounted(() => {
    adminProfessioanlStore.reset();
});

const all_users = computed(() => adminProfessioanlStore.users || null);
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

// Función para descargar Excel - IGUAL que en Bulleting
async function dowloadExcel() {
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
        link.setAttribute('download', 'profesionales_pendientes.xlsx');
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

// Función para actualizar - IGUAL que en Bulleting
const update_item = async () => {
    try {
        if (selectItem) {
            await axios.put(`/auth-pending/${selectItem.value?.id}`, {
                names: selectItem.value?.names,
                lastnames: selectItem.value?.lastnames,
                email: selectItem.value?.email,
                phone: selectItem.value?.phone,
                document: selectItem.value?.document
            })
            
            Swal.fire({
                title: "¡Éxito!",
                text: "El profesional ha sido actualizado correctamente.",
                icon: "success",
                confirmButtonColor: "var(--blue-1)"
            });
            
            selectItem.value = null;
            adminProfessioanlStore.get_all_users(type.value);
        }
    } catch (error) {
        let message = "Ocurrió un error inesperado.";

        if (axios.isAxiosError(error)) {
            message = error.response?.data?.message || error.message;
        } else if (error instanceof Error) {
            message = error.message;
        }

        Swal.fire({
            title: "Error",
            text: message,
            icon: "error",
            confirmButtonColor: "var(--blue-1)"
        });
    }
}
</script>

<template>
    <!-- Modal Ver Documentos - IGUAL estructura que Bulleting -->
    <modal_Float :model-value="panels.viewDocuments && selectItem != null" 
        :width-percent="90" 
        :height-percent="90"
        @click-outside="() => { panels.viewDocuments = false; selectItem = null }" 
        v-if="panels.viewDocuments && selectItem">
        <div class="w-[98%] h-full flex flex-col">
            <div class="w-full">
                <img :src="iconClose" alt="close" class="ml-auto cursor-pointer" 
                    @click="() => { panels.viewDocuments = false; selectItem = null }" />
            </div>
            <div class="w-[95%] m-auto my-2 flex-grow overflow-auto">
                <Documents_panel :data="selectItem" />
            </div>
        </div>
    </modal_Float>

    <!-- Modal Ver/Editar - EXACTAMENTE como Bulleting -->
    <modal_Float :model-value="selectItem != null && panels.editModal" 
        :width-percent="80" 
        :height-percent="80"
        @click-outside="() => { selectItem = null; panels.editModal = false }" 
        v-if="selectItem && panels.editModal">
        <div class="w-[98%] h-full flex flex-col">
            <div class="w-full">
                <img :src="iconClose" alt="close" class="ml-auto cursor-pointer" 
                    @click="() => { selectItem = null; panels.editModal = false }" />
            </div>
            <div class="w-[90%] m-auto my-2 flex-grow overflow-auto">
                <!-- Nombres -->
                <h1 class="font-semibold text-black text-2xl">Nombres</h1>
                <input v-model="selectItem.names" type="text" placeholder="* Nombres" :class="[
                    'w-full p-2 rounded-md border mb-6',
                    selectItem.names?.trim() === '' ? 'border-red-400' : 'border-gray-300'
                ]" />
                
                <!-- Apellidos -->
                <h1 class="font-semibold text-black text-2xl">Apellidos</h1>
                <input v-model="selectItem.lastnames" type="text" placeholder="* Apellidos" :class="[
                    'w-full p-2 rounded-md border mb-6',
                    selectItem.lastnames?.trim() === '' ? 'border-red-400' : 'border-gray-300'
                ]" />
                
                <!-- Correo -->
                <h1 class="font-semibold text-black text-2xl">Correo</h1>
                <input v-model="selectItem.email" type="text" placeholder="* Correo" :class="[
                    'w-full p-2 rounded-md border mb-6',
                    selectItem.email?.trim() === '' ? 'border-red-400' : 'border-gray-300'
                ]" />
                
                <!-- Documento -->
                <h1 class="font-semibold text-black text-2xl">Documento</h1>
                <input v-model="selectItem.document" type="text" placeholder="* Documento" :class="[
                    'w-full p-2 rounded-md border mb-6',
                    selectItem.document?.trim() === '' ? 'border-red-400' : 'border-gray-300'
                ]" />
                
                <!-- Teléfono -->
                <h1 class="font-semibold text-black text-2xl">Teléfono</h1>
                <input v-model="selectItem.phone" type="text" placeholder="* Teléfono" 
                    class="w-full p-2 rounded-md border mb-6 border-gray-300" />
            </div>
            <div @click="update_item"
                class="w-[90%] mx-auto mb-4 bg-[var(--blue-1)] rounded-2xl mt-auto cursor-pointer">
                <h1 class="py-3 text-center text-white font-semibold">Actualizar</h1>
            </div>
        </div>
    </modal_Float>

    <!-- Contenido principal - EXACTAMENTE como users_registers -->
    <div class="w-full bg-gray-100">
        <h1 class="py-10 w-[90%] mx-auto">Bienvenido a Doc Visual Administrador</h1>
        
        <!-- Controles superiores -->
        <div class="w-[90%] mx-auto mb-4 flex justify-between items-center">
            <div class="flex gap-4">
                <!-- Selector de estado -->
                <select v-model="type"
                    class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-[var(--blue-1)]">
                    <option v-for="option in statusOptions" :key="option.value" :value="option.value">
                        {{ option.label }}
                    </option>
                </select>
            </div>
            
            <!-- Botón descargar Excel - IGUAL que Bulleting -->
            <div @click="dowloadExcel" class="mr-5 cursor-pointer">
                <img :src="excelIcon" alt="excel" class="mx-auto w-10 h-auto" />
                <h1 class="md:text-base text-[10px] text-center">Descargar datos</h1>
            </div>
        </div>
        
        <!-- Tabla - IGUAL estructura que users_registers -->
        <div class="w-[90%] mx-auto bg-white rounded-2xl shadow-gray-300 shadow-2xl min-h-[500px]"
            v-if="all_users && all_users.length">
            <table class="min-w-full bg-white shadow-md rounded-xl overflow-hidden">
                <thead class="bg-gray-100 text-gray-700 text-left">
                    <tr>
                        <th class="px-6 py-3">Nombre</th>
                        <th class="px-6 py-3">Correo</th>
                        <th class="px-6 py-3">Fecha inscripción</th>
                        <th class="px-6 py-3">Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="user in all_users" :key="user.id" 
                        class="border-b hover:bg-gray-50"
                        :class="{ 'bg-green-100': user.status === 'ACCEPTED' }">
                        <td class="px-6 py-3">{{ user.names + " " + user.lastnames }}</td>
                        <td class="px-6 py-3">{{ user.email }}</td>
                        <td class="px-6 py-3">{{ formatDate(user.createdAt) }}</td>
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
        
        <!-- Mensaje cuando no hay datos -->
        <div v-else class="w-[90%] mx-auto bg-white rounded-2xl shadow-gray-300 shadow-2xl min-h-[500px] flex items-center justify-center">
            <p class="text-gray-500">No se encontraron profesionales pendientes</p>
        </div>
    </div>
</template>