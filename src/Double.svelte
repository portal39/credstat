<script>
    import {isAuthenticated} from "./store";
    import {onMount,onDestroy} from "svelte";
    let interval = false;

    onDestroy(async () => {
        if(interval)clearInterval(interval);
    })
    onMount(async () => {
        isAuthenticated.subscribe(load);
    });
    async function load(isAuth){

        if(isAuth){
            reload();
            if(interval)clearInterval(interval);
            interval = setInterval(reload, 20000);
        }else{
            clearInterval(interval);
        }
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
    let dd = {
        d:0,
        df:0,
        m:0,
        mf:0
    };
    async function reload(){
        request((response)=>{
            dd = Object.assign(dd,response.double);
        },JSON.stringify({
            action: 'getDouble',
            // filter: filter
        }))
    }
</script>

<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
    <h1 class="h2">Double</h1>
</div>

<section>
    <pre class="status">
За 24 часа <b>{dd.d}</b>(дубли)/<b>{dd.df}</b>(телефоны)
За 31 день <b>{dd.m}</b>(дубли)/<b>{dd.mf}</b>(телефоны)
    </pre>
</section>
