<script>
  import Codes from "./pages/Codes.svelte";

  if ("serviceWorker" in navigator)
    navigator.serviceWorker.register("/serviceWorker.js")
            .then(() => console.log("Service Worker Registered"))
            .catch(err => console.error("Failed to register Cache service worker", err))

  import "./style.css"
</script>

<svelte:head>
  <title>Authenticator</title>
</svelte:head>

<main>

  <nav class="flex">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="w-6 h-6">
      <path fill-rule="evenodd" d="M2 3.75A.75.75 0 0 1 2.75 3h10.5a.75.75 0 0 1 0 1.5H2.75A.75.75 0 0 1 2 3.75ZM2 8a.75.75 0 0 1 .75-.75h10.5a.75.75 0 0 1 0 1.5H2.75A.75.75 0 0 1 2 8Zm0 4.25a.75.75 0 0 1 .75-.75h10.5a.75.75 0 0 1 0 1.5H2.75a.75.75 0 0 1-.75-.75Z" clip-rule="evenodd" />
    </svg>
    <h1 class="text-primary font-bold">2 Factor</h1>
  </nav>

  <section>
    <Codes />
  </section>

</main>

