<template>
    <header class="header">
        <h1 class="title">Gestión de Sorteos de Twitch</h1>
        <div class="user">
            <svg
                xmlns="http://www.w3.org/2000/svg"
                width="24"
                height="24"
                viewBox="0 0 24 24"
                fill="none"
                stroke="var(--color-primary)"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
                class="lucide lucide-user h-4 w-4"
            >
                <path d="M19 21v-2a4 4 0 0 0-4-4H9a4 4 0 0 0-4 4v2"></path>
                <circle cx="12" cy="7" r="4"></circle>
            </svg>
            <span class="username">@Jadez</span>
        </div>
    </header>
</template>

<script setup></script>

<style scoped>
.header {
    background-color: var(--color-background); /* Color de fondo */
    box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1); /* Sombra ligera */
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 20px;
    font-family: var(--font-primary);
}

.title {
    color: var(--color-primary);
    font-size: 20px;
    font-weight: bold;
}

.user {
    display: flex;
    align-items: center;
    gap: 8px;
    color: var(--color-text);
}

.lucide {
    background: var(--color-secondary);
    padding: 8px;
    border-radius: 50%;
}

.username {
    font-weight: 500;
}
</style>
