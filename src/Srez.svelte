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
    let srez_ = [];
    async function reload(){
        request((response)=>{
            srez_ = response.srez;
        },JSON.stringify({
            action: 'getSrez',
            // filter: filter
        }))
    }
</script>

<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
    <h1 class="h2">Srez</h1>
</div>

<section>
    <table class="table table-striped table-hover table-sm table-bordered">
        <thead>
        <tr>
            <th>Date</th>
            <th>Type</th>
            <th>Prop</th>
            <th>Povtor 31d/1d</th>
        </tr>
        </thead>
        <tbody>
        {#each srez_ as srez}
                    <tr>

                        <td>{srez.date}</td>
                        <td>{srez.error}</td>
                        <td>{JSON.stringify(srez.prop)}</td>
                        <td>{#if srez.f1}{srez.f1}{/if}{#if srez.f2}/{srez.f2}{/if}</td>
                    </tr>
        {/each}
        </tbody>
    </table>
</section>
