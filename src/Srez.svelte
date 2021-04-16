<script>
    import {isAuthenticated} from "./store";
    import {onMount,onDestroy} from "svelte";
    let interval = false;

    onDestroy(async () => {
        if(interval)clearInterval(interval);
    })
    let myChart = false;
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
    const colors = [
        'rgba(255, 99, 132, 0.5)',
        'rgba(54, 162, 235, 0.5)',
        'rgba(255, 206, 86, 0.5)',
        'rgba(75, 192, 192, 0.5)',
        'rgba(153, 102, 255, 0.5)'
    ];
    let labels = [];
    let dataset = [];

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

            if(response.dataset) {

                    labels = response.dataset.labels;
                    Object.values(response.dataset.set).forEach(function (item, i, arr) {

                        dataset.push({
                            label: response.dataset.label[i],
                            backgroundColor: colors[i],
                            borderColor: colors[i],
                            data: item,
                        })

                    });

                    if (!myChart) {
                        myChart = new Chart(
                            document.getElementById('myChart'),
                            {
                                type: 'bar',
                                data: {
                                    labels: labels,
                                    datasets: dataset
                                },
                                options: {
                                    responsive: true,
                                    plugins: {
                                        legend: {
                                            position: 'top',
                                        },
                                        title: {
                                            display: true,
                                            text: 'SREZZ'
                                        }
                                    }
                                },
                            }
                        );
                        dataset = [];
                    }else {
                        myChart.data.labels = labels;
                        myChart.data.datasets = dataset;
                        myChart.update();
                    }

            }
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
    <div>
        <canvas id="myChart"></canvas>
    </div>
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
