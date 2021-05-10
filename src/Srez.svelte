<script>
    import {isAuthenticated} from "./store";
    import {onMount,onDestroy} from "svelte";
    let interval = false;
    import Litepicker from 'litepicker';

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
            new Litepicker({
                element: document.getElementById('datepicker'),
                singleMode: false,
                maxDate: new Date(),
                tooltipText: {
                    one: 'night',
                    other: 'nights'
                },
                lang: "ru-RU",
                numberOfMonths: 2,
                numberOfColumns:2,
                format: "YYYY/MM/DD",
                setup: (picker) => {
                    picker.on('selected', (date1, date2) => {
                        dateMin = date1.format('YYYY/MM/DD');
                        dateMax = date2.format('YYYY/MM/DD');
                        reload();
                        clearInterval(interval);
                    });
                },
                tooltipNumber: (totalDays) => {
                    return totalDays - 1;
                }
            })
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
    let dateMin=false, dateMax=false;

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
    let all = [];
    let label = [];
    let phone = [];

    async function reload(){
        request((response)=>{

            srez_ = response.srez;
            all = response.all;
            phone = response.phone;
            console.log(phone)
            label = response.dataset.label;
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

                    }else {
                        myChart.data.labels = labels;
                        myChart.data.datasets = dataset;
                        myChart.update();
                    }
                dataset = [];
            }
        },JSON.stringify({
            action: 'getSrez',
            dateMin: dateMin,
            dateMax: dateMax
            // filter: filter
        }))
    }
    let hide = true;


</script>

<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
    <h1 class="h2">Srez</h1>
</div>

<section>
    <div>
        <canvas id="myChart"></canvas>
    </div>
    <div>
        <button type="button" class="btn btn-secondary" id="datepicker">DatePicker</button>
        {#if all.length>0}

            <p class="mt-5">C {labels[labels.length-1]} по {labels[0]}</p>
        <table class="table table-striped table-hover table-sm table-bordered mb-5">
            <tbody>
        {#each all as v,k}
            <tr><td><b>{label[k]}</b></td><td>{v}</td></tr>
        {/each}
            </tbody>
        </table>

        {/if}
    </div>
    {#if srez_.length>0}
        <div class="form-check form-switch">
            <input class="form-check-input" bind:checked={hide} type="checkbox" id="flexSwitchCheckDefault">
            <label class="form-check-label" for="flexSwitchCheckDefault">Hide/Show Ads</label>
        </div>
    <table class="table table-striped table-hover table-sm table-bordered">
        <thead>
        <tr>
            <th>дата</th>
            <th>повтор 31d/1d</th>
            <th>статут среза</th>
            <th>фио</th>
            <th>дата рождения</th>
            <th>номер телефона</th>
            <th>регион</th>
            <th>город</th>
            <th>сумма</th>
            <th>срок</th>
            <th class:d-none={hide}>обьявление</th>

        </tr>
        </thead>
        <tbody>
        {#each srez_ as srez}
                    <tr>
                        <td>{srez.date}</td>
                        <td>{#if srez.f1}{srez.f1}{/if}/{#if srez.f2}{srez.f2}{/if}<br>{#if phone[srez.prop.TELE.toString()]}[<b>{phone[srez.prop.TELE].join(', ')}</b>]{/if}</td>
                        <td>{srez.error}</td>
                        <td>{srez.prop.EMAIL}{#if srez.prop.NAME}<br>{srez.prop.NAME}{/if}</td>
                        <td>{srez.prop.DAY}-{srez.prop.MONTH}-{srez.prop.YEAR}</td>
                        <td>{srez.prop.TELE}</td>
                        <td>{srez.prop.REGION}</td>
                        <td>{srez.prop.CITY}</td>
                        <td>{srez.prop.SUM}</td>
                        <td>{srez.prop.PERIOD}</td>
                        <td class:d-none={hide}>{srez.prop.ADS}</td>


                    </tr>
        {/each}
        </tbody>
    </table>
    {/if}

</section>
