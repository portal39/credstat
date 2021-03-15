<script>
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
	days=[],files={};
	async function load(isAuth){
		if(isAuth){
			request((resonse)=>{
				let data =  resonse.data;
				ok=data.ok;
				server=data.server;
				error=data.error;
				all_data = data;
				days = all_data.day;
				files = resonse.files;
				// options
			},{
				action:'init'
			})
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

	import DataTable, {Head, Body, Row, Cell} from '@smui/data-table';
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
<!--				<ul class="nav flex-column">-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link active" aria-current="page" href="#">-->
<!--							<span data-feather="home"></span>-->
<!--							Dashboard-->
<!--						</a>-->
<!--					</li>-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="file"></span>-->
<!--							Orders-->
<!--						</a>-->
<!--					</li>-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="shopping-cart"></span>-->
<!--							Products-->
<!--						</a>-->
<!--					</li>-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="users"></span>-->
<!--							Customers-->
<!--						</a>-->
<!--					</li>-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="bar-chart-2"></span>-->
<!--							Reports-->
<!--						</a>-->
<!--					</li>-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="layers"></span>-->
<!--							Integrations-->
<!--						</a>-->
<!--					</li>-->
<!--				</ul>-->

<!--				<h6 class="sidebar-heading d-flex justify-content-between align-items-center px-3 mt-4 mb-1 text-muted">-->
<!--					<span>Saved reports</span>-->
<!--					<a class="link-secondary" href="#" aria-label="Add a new report">-->
<!--						<span data-feather="plus-circle"></span>-->
<!--					</a>-->
<!--				</h6>-->
<!--				<ul class="nav flex-column mb-2">-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="file-text"></span>-->
<!--							Current month-->
<!--						</a>-->
<!--					</li>-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="file-text"></span>-->
<!--							Last quarter-->
<!--						</a>-->
<!--					</li>-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="file-text"></span>-->
<!--							Social engagement-->
<!--						</a>-->
<!--					</li>-->
<!--					<li class="nav-item">-->
<!--						<a class="nav-link" href="#">-->
<!--							<span data-feather="file-text"></span>-->
<!--							Year-end sale-->
<!--						</a>-->
<!--					</li>-->
<!--				</ul>-->
			</div>
		</nav>
		<main class="col-md-9 ms-sm-auto col-lg-10 px-md-4">
			<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
				<h1 class="h2">Dashboard</h1>
<!--				<div class="btn-toolbar mb-2 mb-md-0">-->
<!--					<div class="btn-group me-2">-->
<!--						<button type="button" class="btn btn-sm btn-outline-secondary">Share</button>-->
<!--						<button type="button" class="btn btn-sm btn-outline-secondary">Export</button>-->
<!--					</div>-->
<!--					<button type="button" class="btn btn-sm btn-outline-secondary dropdown-toggle">-->
<!--						<span data-feather="calendar"></span>-->
<!--						This week-->
<!--					</button>-->
<!--				</div>-->
			</div>


		<section>
			<pre class="status">Всего:
	OK - <b>{ok}</b>
	ERROR - <b>{error}</b>
	SERVER - <b>{server}</b>
			</pre>



				<h4></h4>

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

								<td>{day.name_}</td>
								<td>{name.name_}:{prop.name_}:{files[prop.name_]}</td>

								<td>{prop.ok}</td>
								<td>{prop.error}</td>
								<td>{prop.server}</td>
							</tr>
						{/each}

					{/each}
				{/each}
				</tbody>
			</table>



<!--			<pre class="status">Selected: {selected.map(option => option.name_).join(', ')}</pre>-->
<!--			<pre class="status">Total: {selectedOk}</pre>-->

		</section>



		</main>
	</div>
</div>
{/if}


