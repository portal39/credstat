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
		// token.subscribe(checkAuth);

		// auth0Client = await auth.createClient();
		//
		// isAuthenticated.set(await auth0Client.isAuthenticated());
		// user.set(await auth0Client.getUser());
	});
	function setAuth(data){
		token.set(data);
		if(data.token==''){
			LogOut();
		}
		isAuthenticated.set(!!$token.user);
	}

	async function checkAuth() {
		await request(setAuth);

	}
	function request(callback){
		fetch('https://api.swcred.ru/stat/', {
			method: 'POST',
			// mode: 'no-cors',
			cache: 'no-cache',
			credentials: 'include',
			body: JSON.stringify($token)
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
	import Checkbox from '@smui/checkbox';
	$: selectedPrice = selected.reduce((total, option) => option.price + total, 0);
	let options = [
		{
			name: 'Broom',
			description: 'A wooden handled broom.',
			price: 15
		},
		{
			name: 'Dust Pan',
			description: 'A plastic dust pan.',
			price: 8
		},
		{
			name: 'Mop',
			description: 'A strong, durable mop.',
			price: 18
		},
		{
			name: 'Bucket',
			description: 'A metal bucket.',
			price: 13
		}
	];
	let selected = [options[2]];

</script>

<main>
	<!-- App Bar -->

		{#if !$isAuthenticated}
		<div id="signinButton" class="g-signin2" on:click={gAuth}></div>
		{:else }
		<div class="btn" on:click={LogOut}>LogOut</div>
		{/if}


	<section>
		<h2>Data Table</h2>

		<div>
			<DataTable table$aria-label="People list">
				<Head>
					<Row>
						<Cell>Name</Cell>
						<Cell>Favorite Color</Cell>
						<Cell>Favorite Number</Cell>
					</Row>
				</Head>
				<Body>
				<Row>
					<Cell>Steve</Cell>
					<Cell>Red</Cell>
					<Cell numeric>45</Cell>
				</Row>
				<Row>
					<Cell>Sharon</Cell>
					<Cell>Purple</Cell>
					<Cell numeric>5</Cell>
				</Row>
				<Row>
					<Cell>Rodney</Cell>
					<Cell>Orange</Cell>
					<Cell numeric>32</Cell>
				</Row>
				<Row>
					<Cell>Mack</Cell>
					<Cell>Blue</Cell>
					<Cell numeric>12</Cell>
				</Row>
				</Body>
			</DataTable>
		</div>

		<div>
			With Row Selection:

			<div>
				<DataTable>
					<Head>
						<Row>
							<Cell checkbox>
								<Checkbox />
							</Cell>
							<Cell>Name</Cell>
							<Cell>Description</Cell>
							<Cell>Price</Cell>
						</Row>
					</Head>
					<Body>
					{#each options as option (option.name)}
						<Row>
							<Cell checkbox>
								<Checkbox bind:group={selected} value={option} valueKey={option.name} />
							</Cell>
							<Cell>{option.name}</Cell>
							<Cell>{option.description}</Cell>
							<Cell numeric>{option.price}</Cell>
						</Row>
					{/each}
					</Body>
				</DataTable>
			</div>
		</div>

		<pre class="status">Selected: {selected.map(option => option.name).join(', ')}</pre>
		<pre class="status">Total: {selectedPrice}</pre>
	</section>
</main>



