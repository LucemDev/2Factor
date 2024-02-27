<script>
    import Code from "../components/Code.svelte";
    import {Html5QrcodeScanner} from "html5-qrcode";

    let codes = [];
    const saved = localStorage.getItem("codes");
    if (saved) codes = JSON.parse(saved);

    let code = {};

    const scan = () => {
        const scanner = new Html5QrcodeScanner("reader", { fps: 10, qrbox: {width: 250, height: 250} }, false);
        scanner.render((scanned, result) => {
            code.issuer = scanned.split("totp/")[1]?.split("?")[0];
            code.secret = scanned.split("secret=")[1]?.split("&")[0];
            code.name = scanned.split("issuer=")[1]?.split("&")[0] || code.issuer;
            scanner.clear();
        }, console.error)
    }

    const save = () => {
        if (!code.issuer || !code.secret)
            return alert("Details required");
        if (code.secret.length < 16)
            return alert("Secret must be 16 characters long");
        codes = [...codes, code];
        localStorage.setItem("codes", JSON.stringify(codes));
        code = null;
    }

    const remove = (i) => {
        codes = codes.filter((_, j) => i !== j);
        localStorage.setItem("codes", JSON.stringify(codes));
    }


    const backup = () => {
        const codes = JSON.parse(localStorage.getItem("codes"));
        const links = [];
        codes.forEach(code => links.push(`otpauth://totp/${code.issuer}?secret=${code.secret}&issuer=${code.issuer}`));
        const blob = new Blob([links.join("\n")], { type: "text/plain" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url; a.download = "backup.txt";
        a.click(); URL.revokeObjectURL(url);
    }

    const restore = () => {
        const input = document.createElement("input");
        input.type = "file";
        input.accept = "text/plain";
        input.onchange = () => {
            const file = input.files[0];
            const reader = new FileReader();
            reader.onload = () => {
                const links = reader.result.split("\n");
                links.forEach(link => {
                    const url = new URL(link);
                    const issuer = url.searchParams.get("issuer");
                    const secret = url.searchParams.get("secret");
                    codes = [...codes, {issuer, secret}];
                });
                localStorage.setItem("codes", JSON.stringify(codes));
            }
            reader.readAsText(file);
        }
        input.click();
    }
</script>


<div class="flex-1 flex justify-evenly bg-primary">

    <button on:click={restore}>
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
        </svg>
    </button>

    <button on:click={backup}>
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5m-13.5-9L12 3m0 0 4.5 4.5M12 3v13.5" />
        </svg>
    </button>


</div>

<div>
    <ul>
        {#each codes as otp, i}
            <Code {otp} on:remove={() => remove(i)}/>
        {/each}
    </ul>

    <button on:click={() => code = {}} class="absolute bottom-0 right-0 m-16 rounded-full bg-primary text-fade p-2">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="w-8 h-8">
            <path d="M8.75 3.75a.75.75 0 0 0-1.5 0v3.5h-3.5a.75.75 0 0 0 0 1.5h3.5v3.5a.75.75 0 0 0 1.5 0v-3.5h3.5a.75.75 0 0 0 0-1.5h-3.5v-3.5Z" />
        </svg>
    </button>
</div>

{#if code}
    <form on:submit|preventDefault={save} on:reset|preventDefault={() => code=null} class="relative w-64 flex flex-col border border-primary p-4">
        <button type="reset" class="absolute right-0">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="w-4 h-4">
                <path d="M5.28 4.22a.75.75 0 0 0-1.06 1.06L6.94 8l-2.72 2.72a.75.75 0 1 0 1.06 1.06L8 9.06l2.72 2.72a.75.75 0 1 0 1.06-1.06L9.06 8l2.72-2.72a.75.75 0 0 0-1.06-1.06L8 6.94 5.28 4.22Z" />
            </svg>
        </button>
        <input bind:value={code.issuer} type="text" placeholder="Name">
        <input bind:value={code.secret} type="text" placeholder="Secret">
        <button on:click={scan} type="button">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="w-8 h-8">
                <path d="M4.75 4.25a.5.5 0 1 0 0 1 .5.5 0 0 0 0-1Z" />
                <path fill-rule="evenodd" d="M2 3.5A1.5 1.5 0 0 1 3.5 2H6a1.5 1.5 0 0 1 1.5 1.5V6A1.5 1.5 0 0 1 6 7.5H3.5A1.5 1.5 0 0 1 2 6V3.5Zm1.5 0H6V6H3.5V3.5Z" clip-rule="evenodd" />
                <path d="M4.25 11.25a.5.5 0 1 1 1 0 .5.5 0 0 1-1 0Z" />
                <path fill-rule="evenodd" d="M2 10a1.5 1.5 0 0 1 1.5-1.5H6A1.5 1.5 0 0 1 7.5 10v2.5A1.5 1.5 0 0 1 6 14H3.5A1.5 1.5 0 0 1 2 12.5V10Zm1.5 2.5V10H6v2.5H3.5Z" clip-rule="evenodd" />
                <path d="M11.25 4.25a.5.5 0 1 0 0 1 .5.5 0 0 0 0-1Z" />
                <path fill-rule="evenodd" d="M10 2a1.5 1.5 0 0 0-1.5 1.5V6A1.5 1.5 0 0 0 10 7.5h2.5A1.5 1.5 0 0 0 14 6V3.5A1.5 1.5 0 0 0 12.5 2H10Zm2.5 1.5H10V6h2.5V3.5Z" clip-rule="evenodd" />
                <path d="M8.5 9.417a.917.917 0 1 1 1.833 0 .917.917 0 0 1-1.833 0ZM8.5 13.083a.917.917 0 1 1 1.833 0 .917.917 0 0 1-1.833 0ZM13.083 8.5a.917.917 0 1 0 0 1.833.917.917 0 0 0 0-1.833ZM12.166 13.084a.917.917 0 1 1 1.833 0 .917.917 0 0 1-1.833 0ZM11.25 10.333a.917.917 0 1 0 0 1.833.917.917 0 0 0 0-1.833Z" />
            </svg>
        </button>
        <div id="reader" width="600px"></div>
        <button type="submit">Save</button>
    </form>
{/if}