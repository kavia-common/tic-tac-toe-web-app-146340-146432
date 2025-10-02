<script lang="ts">
    import '../app.css';
    let { children } = $props();
</script>

<main class="shell">
    {@render children()}
</main>

<style>
    .shell {
        min-height: 100svh;
        display: grid;
        place-items: center;
        padding: 2rem 1rem;
    }
</style>
