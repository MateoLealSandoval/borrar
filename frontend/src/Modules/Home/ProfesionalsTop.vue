<script lang="ts" setup>
import { onMounted } from "vue";
const router = useRouter();
const navigate = () => {
            router.push('/specialists');
        };
import CardTopUser from "@/common/Card_Top_user.vue";
 
import { HomeTopStore } from "@/store/homeTop.store";
import { useRouter } from "vue-router";

// Store
const professionalStore = HomeTopStore();

// Cargar especialistas al montar el componente
onMounted(() => {
  professionalStore.load_specialists();
});
</script>
<template>
    <div class="w-full  bg-white py-20 z-10">
         <!-- Fondo inferior -->
    <div
    style="position: absolute; bottom: 0; left: 0; width: 100%; height: 38%; background: var( --blue-gradientPurple); z-index: 0;"
  ></div>

        <div class=" w-full m-auto">
            <h1 class="text-center px-5 w-full font-poppins text-xl md:text-4xl py-1   font-semibold text-gray-600"
                 >
               Nuestros especialistas destacados
            </h1>
            
        </div>
        <div class="w-full container m-auto flex justify-center z-10">
            <div class="container mx-auto grid gap-4 text-center mt-20 
            grid-cols-1 md:grid-cols-2 lg:grid-cols-3 justify-items-center">
                <CardTopUser v-for="specialist in professionalStore.list_professionals" :Description="specialist.description"  :image="specialist.perfilPhoto" :experiece="specialist.experience" v-bind="specialist.specialists[0] ? { rol: specialist.specialists[0].name } : {}" :name="specialist.name" :option="'Optometria'" :id="specialist.id"/>
 
            </div>

        </div>
        <div class="flex justify-center my-10 md:my-20">
            <button @click="navigate"
                class="w-[70%] md:w-auto cursor-pointer font-medium text-lg font-poppins my-1    px-10 py-2 tracking-wide text-[var(--blue-3)]   transition-colors duration-300 transform bg-gray-50  focus:outline-none focus:ring focus:ring-blue-300 focus:ring-opacity-80 rounded-3xl">
                Conoce a todos nuestros especialistas

            </button>
        </div>
    </div>

</template>
