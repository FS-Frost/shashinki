<script lang="ts">
    import { onMount } from "svelte";
    import Swal from "sweetalert2";
    import { Color } from "./color";
    import { GitHub } from "./github";

    const KEY_TOKEN: string = "shashinki.token";

    let token: string = "";
    let github: GitHub | null;
    let authenticated: boolean = false;
    let owner: string = "";
    let repo: string = "shashinki-db";
    let dataPath: string = "shashin.ki";

    async function login(): Promise<void> {
        try {
            owner = "";

            Swal.fire({
                title: "Iniciando sesión",
                didOpen: () => {
                    Swal.showLoading();
                },
            });

            github = GitHub.new(token);
            const user = await github.getUser();
            const responseRepos = await github.listRepos(user.login);
            const repos = responseRepos.map((repo) => repo.name);
            owner =
                responseRepos.length > 0
                    ? responseRepos[0].owner.login ?? ""
                    : "";

            await Swal.fire({
                icon: "success",
                title: `Bienvenido, ${owner}`,
                confirmButtonText: "Aceptar",
                confirmButtonColor: Color.Primary,
            });

            authenticated = true;
            localStorage.setItem(KEY_TOKEN, token);
        } catch (error) {
            authenticated = false;

            if (Swal.isVisible()) {
                Swal.close();
            }

            await Swal.fire({
                icon: "error",
                title: "Error",
                text: `Error: ${error}`,
                confirmButtonText: "Aceptar",
                confirmButtonColor: Color.Primary,
            });
        }
    }

    async function sendData(): Promise<void> {
        try {
            if (github == null) {
                alert("No github connection");
                return;
            }

            Swal.fire({
                title: "Enviando datos",
                didOpen: () => {
                    Swal.showLoading();
                },
            });

            const user = await github.getUser();
            const data = "some-data" + new Date();

            const responseCreateCommit = await github.createCommit(
                user.login,
                repo,
                dataPath,
                data,
            );

            if (!responseCreateCommit.status) {
                throw new Error("Error al guardar en GitHub");
            }

            await Swal.fire({
                icon: "success",
                title: "Guardado",
                confirmButtonText: "Aceptar",
                confirmButtonColor: Color.Primary,
            });
        } catch (error) {
            if (Swal.isVisible()) {
                Swal.close();
            }

            await Swal.fire({
                icon: "error",
                title: "Error",
                text: `Error: ${error}`,
                confirmButtonText: "Aceptar",
                confirmButtonColor: Color.Primary,
            });
        }
    }

    onMount(() => {
        const savedToken = localStorage.getItem(KEY_TOKEN) ?? "";
        if (savedToken.length > 0) {
            token = savedToken;
        }
    });
</script>

<main>
    <div class="field">
        <label class="label" for="">Token</label>
        <div class="control">
            <input class="input" type="password" bind:value={token} />
        </div>
    </div>

    <button
        class="button is-link is-fullwidth"
        on:click={() => login()}
        disabled={token.length === 0}
    >
        Iniciar sesión
    </button>

    {#if authenticated}
        <button
            class="button is-success is-fullwidth"
            on:click={() => sendData()}
        >
            Enviar datos
        </button>

        <a
            href="https://github.com/{owner}/{repo}/blob/main/{dataPath}"
            target="_blank"
        >
            Ver en GitHub
        </a>
    {/if}
</main>

<style>
    main {
        padding: 1em;
        margin: 0 auto;
    }
</style>
