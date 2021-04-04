<script>
    import JSONTree from 'svelte-json-tree-auto';
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
    let ok='',server='',error='',all_data={},
        days=[],
        all={},
        jsonView=false,
        files={},
        valueJS='',
        offers= [],
        props= [],
        filter={};
    let jsonValue = {'test':123};
    let errorLog = {};
    let viewError = '';


    $: {
        offers.forEach(function (item, i, arr) {
            if (filter.offer && item.name_ == filter.offer) {
                offers[i]['active'] = true;
            }
        });

    }

    async function reload(){
        request((response)=>{
            if(response.data) {
                let data = response.data;
                ok = data.ok;
                server = data.server;
                error = data.error;
                all_data = data;
                days = all_data.day;
                offers = response.tags.name;
                props = response.tags.prop;
                files = response.files;
                all = response.all;
            }
        },JSON.stringify({
            action: 'getData',
            filter: filter
        }))
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

    async function ClearErrorDetail(){
        jsonView = false;
    }
    async function ClearError(){
        errorLog = {};
        ClearErrorDetail();
    }
    async function OfferSelect(){
        delete filter.pid;
        ClearError();
        if(filter.offer && filter.offer==this.textContent) {
            delete filter.offer;
        }
        else {
            filter.offer = this.textContent;
        }
        reload();
    }
    async function PropSelect(){
        let pid = this.dataset.pid;
        delete filter.offer;
        if(filter.pid && filter.pid==pid)
            delete filter.pid;
        else
            filter.pid = pid;
        reload();
    }

    async function GetError(){
        viewError = files[this.dataset.pid];
        request((response)=>{
            errorLog=response;

        },JSON.stringify({
            action: 'getError',
            q:this.dataset
        }))
    }
    async function GetErrorById(){

        request((response)=>{
            jsonValue =response.ID;
            jsonView = true;

        },JSON.stringify({
            action: 'getErrorById',
            q:this.dataset
        }))
    }
    async function DateSelect(){
        let did = this.dataset.did;
        if(filter.did && filter.did==did)
            delete filter.did;
        else
            filter.did = did;
        reload();
    }

</script>

    <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
        <h1 class="h2">Dashboard</h1>
    </div>
    <section>
			<pre class="status">Всего:
	OK - <b>{ok}</b>
	ERROR - <b>{error}</b>
	SERVER - <b>{server}</b>
                {#if all.this_day_all>0}ALL - <b>{all.this_day_all}</b>{/if}
                {#if all.this_day>0}FROM_COOKIE - <b>{all.this_day}</b>{/if}
			</pre>

        {#each offers as el}
            <button type="button" class="btn {el.active ? 'btn-dark' : 'btn-secondary' } m-3" on:click={OfferSelect}>{el.name_}</button>
        {/each}
        {#if filter.did}
            <button type="button" data-did="{filter.did}" on:click={DateSelect} class="btn btn-dark">{filter.did}</button>
        {/if}
        {#if filter.pid}
            <button type="button" data-did="{filter.pid}" on:click={PropSelect} class="btn btn-dark">{files[filter.pid]}</button>
        {/if}
        {#if jsonView}
            <div>
                <button type="button" on:click={ClearErrorDetail} class="btn btn-dark">Hide error detail</button>
                <JSONTree value={jsonValue} />
            </div>
        {/if}
        {#if errorLog.CPAHub}
            <button type="button" on:click={ClearError} class="btn btn-dark">Error log CPAHub {viewError}</button>
            <table class="table table-striped table-hover table-sm table-bordered">
                <tbody>
                {#each errorLog.CPAHub as error}
                    <tr><td on:click={GetErrorById} data-id="{error[3]}">{error[2]}</td><td>{error[1]}</td><td>
                        {#each Object.entries(error[0]) as [title, value]}
                            <code>{title}</code> {value}<br/>
                        {/each}
                    </td></tr>
                {/each}
                </tbody>
            </table>
        {/if}
        {#if errorLog.MFOGate}
            <button type="button" on:click={ClearError} class="btn btn-dark">Error log MFOGate {viewError}</button>

            <div class="alert alert-warning" role="alert">
                Удалены ошибки с дублями телефона
            </div>
            <table class="table table-striped table-hover table-sm table-bordered">
                <tbody>
                {#each errorLog.MFOGate as error}
                    <tr><td on:click={GetErrorById} data-id="{error[3]}">{error[2]}</td><td>{error[1]}</td><td>
                        {#each Object.entries(error[0]) as [title, value]}
                            <code>{title}</code> {value}<br/>
                        {/each}
                    </td></tr>
                {/each}
                </tbody>
            </table>
        {/if}
        {#if errorLog.Firano}
            <button type="button" on:click={ClearError} class="btn btn-dark">Error log Firano {viewError}</button>

            <table class="table table-striped table-hover table-sm table-bordered">
                <tbody>
                {#each errorLog.Firano as error}
                    <tr><td on:click={GetErrorById} data-id="{error[3]}">{error[2]}</td><td>{error[1]}</td><td>
                        {#each Object.entries(error[0]) as [title, value]}
                            <code>{title}</code>&nbsp;
                            {#if typeof value !== 'object'}{value}<br/>{:else}
                                {JSON.stringify(value)}
                            {/if}
                        {/each}
                    </td></tr>
                {/each}
                </tbody>
            </table>
        {/if}
        {#if errorLog.MTarget}
            <button type="button" on:click={ClearError} class="btn btn-dark">Error log MTarget {viewError}</button>

            <div class="alert alert-warning" role="alert">
                Данные агрегированы
            </div>
            <table class="table table-striped table-hover table-sm table-bordered">
                <tbody>
                {#each errorLog.MTarget as error}
                    <tr><td>{error.key}</td><td>{error.doc_count}</td></tr>
                {/each}
                </tbody>
            </table>
        {/if}




        <table class="table table-striped table-hover table-sm table-bordered">
            <thead>
            <tr>

                <th>Date</th>
                <th>Offer</th>

                <th>OK</th>
                <th>ERROR</th>
                <th>SERVER</th>
            </tr>
            </thead>
            <tbody>
            {#each days as day}
                {#each day.name as name}

                    {#each name.prop as prop}
                        <tr>

                            <td data-did="{day.name_}" on:click={DateSelect}>{day.name_}</td>
                            <td data-pid="{prop.name_}" on:click={PropSelect}>{name.name_}:{prop.name_}: <code>{files[prop.name_]}</code></td>

                            <td>{prop.ok}</td>
                            <td data-did="{day.name_}" data-pid="{prop.name_}" data-tid="{name.name_}" on:click={GetError}>{prop.error}</td>
                            <td>{prop.server}</td>
                        </tr>
                    {/each}

                {/each}
            {/each}
            </tbody>
        </table>




        <!--			<pre class="status">Total: {selectedOk}</pre>-->

    </section>



