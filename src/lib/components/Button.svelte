<script lang="ts">
    let {
        value,icon: Icon,onClick
    } = $props();
</script>

<button onclick={onClick}>
    {#if Icon}
        <Icon size={20}></Icon>
    {:else}
        {value}
    {/if}
</button>

<style>
button {
    width: 100%;
    height: 100%;
    min-height: 64px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: none;
    border-radius: 18px;
    background: #f8fafc;
    color: #1e293b;
    font-size: 1.3rem;
    cursor: pointer;
}

button:hover {
    transform: translateY(-2px);
    background: #eef2ff;
}

button:active {
    transform: scale(0.96);
}

:global(.equal-btn button) {
    background:linear-gradient( 135deg #2563eb, #1d4ed8 );
    color: white;
    font-size: 1.3rem;
    box-shadow: 0 10px 20px rgba(37, 99, 235, 0.35);
}

:global(.equal-btn button:hover) {
    background:linear-gradient( 135deg, #1d4ed8, #1e40af);
}

@media (max-width: 768px) {
    button {
        min-height: 58px;
        font-size: 1rem;
        border-radius: 16px;
    }
}
</style>