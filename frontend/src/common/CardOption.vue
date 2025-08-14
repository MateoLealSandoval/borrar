<script setup lang="ts">
import { useRouter } from 'vue-router';
const props = defineProps<{
    image: string;
    option: string;
    link: string;
}>();
const router = useRouter();

const navigate = () => {
    router.push(props.link);
};
</script>

<template>
    <div class="card-container w-[23%] overflow-hidden rounded-3xl flex flex-col cursor-pointer">
        <!-- Ícono circular con imagen -->
        <div class="icon-wrapper mt-10 flex items-center justify-center p-5 md:p-6 lg:p-8 m-auto bg-white rounded-full w-[50%] aspect-square overflow-hidden z-10">
            <img class="object-contain object-center w-full h-full" :src="image" :alt="option" />
        </div>

        <!-- Texto de la especialidad -->
        <div class="py-6 text-center min-h-[50px] md:min-h-[150px] z-10">
            <a href="#" class="break-words text text-[10px] md:text-2xl font-poppins font-bold text-gray-800 leading-[1.2]">
                <div class="option-text text-[10px] md:text-2xl font-poppins font-bold leading-[1.3]" v-html="option"></div>
            </a>
        </div>

        <!-- Botón de acción -->
        <button class="action-button rounded-3xl w-[80%] m-auto mb-4 md:mb-8 cursor-pointer px-1 bg-white font-poppins font-semibold text-[9px] md:text-xl py-1 md:py-2 tracking-wide text-[var(--blue-3)] transition-all duration-300 transform hover:bg-white hover:text-[var(--blue-3)] focus:outline-none"
            @click="navigate">
            Agenda ahora
        </button>
    </div>
</template>

<style scoped>
/* Contenedor principal de la tarjeta */
.card-container {
    position: relative;
    background: var(--blue-purple-2);
    transition: all 0.3s ease;
}

/* Estado normal: texto oscuro */
.card-container .option-text {
    color: #374151; /* text-gray-800 */
    transition: color 0.3s ease;
}

/* Estado hover: aplicar estilos destacados */
.card-container:hover {
    background: var(--blue-purple-2);
    box-shadow: 
        0 0 0 3px white,
        0 10px 25px rgba(100, 121, 253, 0.3),
        0 20px 40px rgba(100, 121, 253, 0.15);
    transform: translateY(-2px);
}

/* Texto blanco en hover */
.card-container:hover .option-text {
    color: white;
}

/* Outline blanco para el círculo del ícono en hover */
.card-container:hover .icon-wrapper {
    box-shadow: 
        0 0 0 3px rgba(255, 255, 255, 0.5),
        0 4px 12px rgba(0, 0, 0, 0.1);
}

/* Botón mantiene sus estilos en hover de la tarjeta */
.card-container:hover .action-button {
    background-color: white;
    color: var(--blue-3);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* Efecto adicional cuando se hace hover directo sobre el botón */
.action-button:hover {
    transform: scale(1.05);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

/* Para mantener consistencia con el diseño responsive */
@media (max-width: 768px) {
    .card-container:hover {
        box-shadow: 
            0 0 0 2px white,
            0 6px 15px rgba(100, 121, 253, 0.25);
    }
}
</style>