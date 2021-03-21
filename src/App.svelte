<script>
	import JSONTree from 'svelte-json-tree-auto';
	import {onMount} from "svelte";
	import {isAuthenticated, token} from "./store";
	var auth2;
	window.start = (googleUser) => {
		gapi.load('auth2', function() {
			auth2 = gapi.auth2.init({
				client_id: '895503250306-0svh9lp60r3hu1ca4p93dbhkd183vcm4.apps.googleusercontent.com',
				//scope: 'additional_scope'
			});
		});
	}
	onMount(async () => {
		checkAuth();
		setInterval(checkAuth,60000)
		isAuthenticated.subscribe(load);

	});
	let ok='',server='',error='',all_data={},
	days=[],
			files={},
			valueJS='',
			offers= [],
			props= [],
			filter={};
	let jsonValue;
	let error2 = null;
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
			}
		},JSON.stringify({
			action: 'getData',
			filter: filter
		}))
	}
	let interval = false;
	async function load(isAuth){
		if(isAuth){
			reload();
			if(interval)clearInterval(interval);
			interval = setInterval(reload, 20000);
		}else{
			clearInterval(interval);
		}
	}
	function setAuth(data){
		token.set(data);
		if(data.token==''){
			LogOut();
		}
		isAuthenticated.set(!!$token.user);
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
		await auth2.grantOfflineAccess().then((authResult) => {token.set({code:authResult['code']});checkAuth()});
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
	async function ClearError(){
		errorLog = {};
	}
	async function OfferSelect(){
		delete filter.pid;
		errorLog = {};
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
	let errorLog = {};
	let viewError = '';
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

			jsonValue = new Function(`return ${response}`)();

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


	// $: selectOffer =
	// import DataTable, {Head, Body, Row, Cell} from '@smui/data-table';
	// import Checkbox from '@smui/checkbox';
	// $: selectedOk = selected.reduce((total, option) => {
	// 	console.log(option);
	// 	option.ok + total}, 0);

	// let selected = [];

</script>
<header class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0 shadow">
	<a class="navbar-brand col-md-3 col-lg-2 me-0 px-3" href="#">CredStat</a>
	<button class="navbar-toggler position-absolute d-md-none collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#sidebarMenu" aria-controls="sidebarMenu" aria-expanded="false" aria-label="Toggle navigation">
		<span class="navbar-toggler-icon"></span>
	</button>
<!--	<input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">-->

	{#if $isAuthenticated}
	<ul class="navbar-nav px-3">
		<li class="nav-item text-nowrap">
			<a class="nav-link" href="#" on:click={LogOut}>Sign out</a>
		</li>
	</ul>
	{:else}
		<div id="signinButton" class="g-signin2 navbar-nav px-3" on:click={gAuth}></div>
	{/if}
</header>
{#if $isAuthenticated}
<div class="container-fluid">
	<div class="row">
		<nav id="sidebarMenu" class="col-md-3 col-lg-2 d-md-block bg-light sidebar collapse">
			<div class="position-sticky pt-3">

			</div>
		</nav>
		<main class="col-md-9 ms-sm-auto col-lg-10 px-md-4">
			<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
				<h1 class="h2">Dashboard</h1>
			</div>


		<section>
			<pre class="status">Всего:
	OK - <b>{ok}</b>
	ERROR - <b>{error}</b>
	SERVER - <b>{server}</b>
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


			<JSONTree value={jsonValue} />

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



		</main>
	</div>
</div>
{/if}


