<script>
    import {isAuthenticated, token} from "./store";
    import {onMount} from "svelte";
    onMount(async () => {
        checkAuth();
        setInterval(checkAuth,60000)
        // isAuthenticated.subscribe(load);

    });




    let tries = 0;
    function setAuth(data){
        if(tries<4 && data.token==''){
            tries++;
            setTimeout(checkAuth,2000);

        }else {
            token.set(data);
            if (data.token == '') {
                LogOut();
            }
            isAuthenticated.set(!!$token.user);
        }
    }

    async function checkAuth() {
        await request(setAuth,JSON.stringify($token));
    }
    function request(callback,data){
        fetch('https://api.swcred.ru/stat/', {
            method: 'POST',
            // mode: 'no-cors',
            cache: 'no-cache',
            credentials: 'include',
            body: data,
        }).then((response) => {
            return response.json();
        }).then(callback);
    }

    async function gAuth() {
        let ga = gapi.auth2.getAuthInstance();
        Promise.resolve(ga.grantOfflineAccess())
            .then((authResult) => {token.set({code:authResult['code']});checkAuth()})
            .catch(console.error);
    }
    async function LogOut(){

        token.set({
            code:'logout',
            token:'',
            user:false
        });
        isAuthenticated.set(false);

        await gapi.auth2.getAuthInstance().disconnect();
        checkAuth();
        await gapi.signin2.render('signinButton')

    }
</script>
<header class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0 shadow">
    <span class="navbar-brand col-md-3 col-lg-2 me-0 px-3">CredStat v.1.0.6</span>
    <button class="navbar-toggler position-absolute d-md-none collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#sidebarMenu" aria-controls="sidebarMenu" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <!--	<input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">-->

    {#if $isAuthenticated}
        <ul class="navbar-nav px-3">
            <li class="nav-item text-nowrap">
                <a class="nav-link" on:click={LogOut}>Sign out</a>
            </li>
        </ul>
    {:else}
        <div id="signinButton" class="g-signin2 navbar-nav px-3" on:click={gAuth}></div>
    {/if}
</header>
