<script lang="ts">
import Card_Text from "@/common/Card_Text.vue";

export default {
    name: "Figures",
    components: {
        Card_Text
    },
}
</script>

<template>
    <div class="w-full my-20">
        <h1 class="w-full text-center font-poppins font-medium text-2xl md:text-4xl mt-20"
            style="color: var(--blue-3);">Cifras que nos enorgullecen</h1>
        <div class="flex flex-wrap justify-between my-16">
            <Card_Text :text="'Citas realizadas'" item="+" :data="50000"
                class="w-[47%] md:w-[23%] mb-6 flex flex-col justify-between" />
            <Card_Text :text="'Personas atendidas'" item="+" :data="100000"
                class="w-[47%] md:w-[23%] mb-6 flex flex-col justify-between" />
            <Card_Text :text="'Satisfacción'" item="%" :data="98"
                class="w-[47%] md:w-[23%] mb-6 flex flex-col justify-between" />
            <Card_Text :text="'Comunidad Instagram'" item="+" :data="500"
                class="w-[47%] md:w-[23%] mb-6 flex flex-col justify-between" />
        </div>

        <h1 class="w-full font-poppins font-medium text-center text-2xl md:text-4xl mb-10"
            style="color: var(--blue-3);">
            Nuestros aliados</h1>
        
        <!-- Contenedor de logos mejorado con mejor control de tamaños -->
        <div class="w-full flex flex-wrap justify-center items-center gap-8 px-4">
            <!-- Cada logo ahora tiene un contenedor con altura fija para uniformidad -->
            <div class="partner-logo-container">
                <img class="partner-logo" src="@/assets/images/partner_logos/logoiconica.jpg" alt="Icónica" />
            </div>
            <div class="partner-logo-container">
                <img class="partner-logo" src="@/assets/images/partner_logos/LogoSantotomasjpg.jpeg" alt="Santo Tomás" />
            </div>
            <div class="partner-logo-container">
                <img class="partner-logo" src="@/assets/images/partner_logos/franjalogo.png" alt="Grupo Franja" />
            </div>
            <div class="partner-logo-container">
                <img class="partner-logo" src="@/assets/images/partner_logos/logoSivit.jpg" alt="Sivit Connect" />
            </div>
        </div>
    </div>
</template>

<style scoped>
/* Estilos para los contenedores de logos */
.partner-logo-container {
    /* Tamaño base para móviles */
    width: 120px;
    height: 80px;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 8px;
}

/* Estilos para las imágenes de logos */
.partner-logo {
    max-width: 100%;
    max-height: 100%;
    width: auto;
    height: auto;
    object-fit: contain;
    /* Aplica un filtro suave para mejorar la visualización */
    filter: contrast(1.05);
}

/* Responsive: tablets */
@media (min-width: 768px) {
    .partner-logo-container {
        width: 160px;
        height: 100px;
        padding: 12px;
    }
}

/* Responsive: desktop */
@media (min-width: 1024px) {
    .partner-logo-container {
        width: 200px;
        height: 120px;
        padding: 16px;
    }
}

/* Hover effect opcional para interactividad */
.partner-logo-container:hover .partner-logo {
    transform: scale(1.05);
    transition: transform 0.3s ease;
}
</style>