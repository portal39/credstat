<script>

	import {onMount} from "svelte";
	import {isAuthenticated} from "./store";

	import Main from './Main.svelte';
	import Auth from './Auth.svelte';
	import Double from './Double.svelte';
	import Srez from './Srez.svelte';

	let active = "main";



</script>
<Auth/>

{#if $isAuthenticated}
<div class="container-fluid">
	<div class="row">
		<nav id="sidebarMenu" class="col-md-3 col-lg-2 d-md-block bg-light sidebar collapse">
			<div class="position-sticky pt-3">
				<ul class="nav flex-column">
					<li class="nav-item">
						<a class="nav-link active" href="#main" aria-current="page" on:click="{() => active = 'main'}">
							<span data-feather="home"></span>Dashboard</a>
					</li>
					<li class="nav-item">
						<a class="nav-link" href="#double" on:click="{() => active = 'double'}">
							<span data-feather="file"></span>Дубли</a>
					</li>
					<li class="nav-item">
						<a class="nav-link" href="#srez" on:click="{() => active = 'srez'}">
							<span data-feather="file"></span>Srez</a>
					</li>
				</ul>
			</div>
		</nav>
		<main class="col-md-9 ms-sm-auto col-lg-10 px-md-4">

			{#if active==='main'}
				<Main/>
			{/if}
			{#if active==='double'}
				<Double/>
			{/if}
			{#if active==='srez'}
				<Srez/>
			{/if}
		</main>
	</div>
</div>
{/if}


