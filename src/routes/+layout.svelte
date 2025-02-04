<script>
	import "../style.css"
    import "swiper/css"
    import Navbar from "$lib/components/Navbar.svelte";

    let {children} = $props();
</script>

<div>
    <header>
        <Navbar/>
    </header>
    <main>
        {@render children()}
    </main>
</div>