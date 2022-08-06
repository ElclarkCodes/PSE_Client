<script context="module">
	export async function load({ fetch, url }) {
		const params = await url.searchParams

		let search = (await params.get('search')) || ''
		let category = (await params.get('category')) || ''
		let status = (await params.get('status')) || ''
		let limit = (await params.get('limit')) || 10
		let offset = (await params.get('offset')) || 0

		let api = `https://api.elclark.my.id/v2/pse?search=${search}&category=${category}&status=${status}&limit=${limit}&offset=${offset}`
		const response = await fetch(api)

		if (!response.ok) {
			return {
				status: response.status
			}
		}

		if (response.headers.get('Content-Type') !== 'application/json') {
			return {
				status: response.status
			}
		}

		const list = await response.json()

		return {
			props: {
				list,
				search,
				category,
				status,
				limit,
				offset
			}
		}
	}
</script>

<script>
	import { fade } from 'svelte/transition'

	export let list = []

	export let search = ''
	export let category = ''
	export let status = ''
	export let limit = 10
	export let offset = 0

	let loading = false
	let loadingMore = false

	let noMoreData = false
	let lastSearch = ''

	let error = false
	let errorMessage = ''
	let errorCode = ''

	let notFound = false

	async function reloadData() {
		// empty list
		list = []

		// set loading
		loading = true

		// set offset
		offset = 0

		// set noMoreData
		noMoreData = false

		// set NotFound
		notFound = false

		let response = await fetch(
			`https://api.elclark.my.id/v2/pse?search=${search}&category=${category}&status=${status}&limit=${limit}&offset=${offset}`
		).catch((e) => {
			error = true
			errorMessage = 'Ada kesalahan saat memuat data, silahkan coba lagi!'
			errorCode = e.message

			// set loading
			loading = false
		})

		// if response is not ok
		if (!response.ok) {
			error = true
			errorMessage = 'Ada kesalahan saat memuat data, silahkan coba lagi!'
			errorCode = response.statusText + ' (' + response.status + ')'

			// set loading
			loading = false
		}

		// get data
		let data = await response.json()

		// set list
		list = [...data]

		// check if there is no more data by checking
		// if the received data is less than the limit
		if (list.length < limit) {
			noMoreData = true
		}

		if (list.length === 0) {
			notFound = true
		}

		// set loading
		loading = false
	}

	// load more data
	async function loadMoreData() {
		// set loading
		loadingMore = true

		// set offset
		offset += limit

		// get data
		let response = await fetch(
			`https://api.elclark.my.id/v2/pse?search=${search}&category=${category}&status=${status}&limit=${limit}&offset=${offset}`
		).catch((e) => {
			error = true
			errorMessage = 'Ada kesalahan saat memuat data, silahkan coba lagi!'
			errorCode = e.message

			// set loading
			loadingMore = false
		})

		// if response is not ok
		if (!response.ok) {
			error = true
			errorMessage = 'Ada kesalahan saat memuat data, silahkan coba lagi!'
			errorCode = response.statusText + ' (' + response.status + ')'

			// set loading
			loadingMore = false
		}

		let data = await response.json()

		// check if there is no more data by checking
		// if the received data is less than the limit
		if (data.length < limit) {
			noMoreData = true
		}

		// set list
		list = [...list, ...data]

		// set loading
		loadingMore = false
	}

	function validateURL(url) {
		// returns true if url is valid
		try {
			new URL(url)
			return true
		} catch (_) {
			return false
		}
	}

	function fixURL(url) {
		if (url === '' || url === '-') {
			return false
		}

		url = url.toLowerCase()

		if (validateURL(url)) {
			return url
		} else {
			if (validateURL('http://' + url)) {
				return 'http://' + url
			}
		}

		return false
	}

	function doSearch() {
		if (search === lastSearch) {
			return
		}

		lastSearch = search
		reloadData()
	}

	function closeError() {
		error = false
		errorMessage = ''
		errorCode = ''
	}

	function getColor(status) {
		if (status === 'TERDAFTAR') {
			return ''
		}

		if (status === 'DICABUT') {
			return 'dicabut'
		}

		return 'dihentikan_sementara'
	}
</script>

<div>
	<article>
		<a href="/">
			<h1>Check PSE</h1>
		</a>

		<form on:submit|preventDefault={doSearch}>
			<input
				type="text"
				name="search"
				placeholder="Cari"
				on:change={doSearch}
				bind:value={search}
			/>
			<select name="category" bind:value={category} on:change={reloadData}>
				<option value="">Semua (Asing dan Domestik)</option>
				<option value="asing">Asing</option>
				<option value="lokal">Domestik</option>
			</select>
			<select name="status" bind:value={status} on:change={reloadData}>
				<option value="">Semua (Terdaftar, Dihentikan Sementara dan dicabut)</option>
				<option value="terdaftar">Terdaftar</option>
				<option value="dihentikan_sementara">Dihentikan Sementara</option>
				<option value="dicabut">Dicabut</option>
			</select>
			<button type="submit">Cari</button>
		</form>
	</article>

	<div class="list">
		{#each list as item}
			<article in:fade class={getColor(item.status_id)}>
				<h2>{item.nama}</h2>
				{#if fixURL(item.website)}
					<a href={fixURL(item.website)}>{fixURL(item.website)}</a>
				{/if}
				<p>{item.nama_perusahaan}</p>
				<p>{item.sektor}</p>
			</article>
		{/each}

		{#if notFound}
			<article>
				<center>
					<p>Data Tidak Ditemukan!</p>
				</center>
			</article>
		{/if}

		{#if loading}
			<div aria-busy="true" />
		{/if}

		{#if !loading && list.length > 0 && !noMoreData}
			<button
				type="button"
				class="secondary"
				aria-busy={loadingMore}
				on:click={loadMoreData}
				on:scroll={loadMoreData}>Muat Lainnya</button
			>

			<noscript>
				<article>
					<p>
						Mohon maaf! fitur ini tidak didukung di browser ini. silahkan gunakan browser yang
						mendukung javascript.
					</p>
				</article>
			</noscript>
		{/if}
	</div>

	{#if error}
		<dialog open>
			<article>
				<header>
					<span aria-label="Close" class="close" on:click={closeError} />
					Terjadi Kesalahan!
				</header>
				<p>
					{errorMessage}
				</p>

				<code>
					{errorCode}
				</code>
			</article>
		</dialog>
	{/if}

	<article>
		<header><h4>PERINGATAN!</h4></header>
		<p>
			Ini <strong>bukan</strong> website resmi <strong>Kominfo</strong>. dan kami
			<strong>tidak</strong>
			memiliki hubungan apapun dengan <strong>Kominfo</strong>. untuk informasi lebih lanjut tentang
			<strong>Penyelenggara Sistem Elektronik</strong> silahkan kunjungi website resmi
			<a href="https://pse.kominfo.go.id"><strong>Kominfo</strong></a>.
		</p>
	</article>

	<footer class="made">
		<center
			><strong
				>Made With Love By <a href="https://elclark.my.id/profiles/elclark">Elclark</a></strong
			>
		</center>
	</footer>
</div>

<style>
	h1 {
		text-align: center;

		margin-top: 0;
		margin-bottom: 1.5rem;
	}

	article h4 {
		margin: 0;
	}

	article h2 {
		font-size: 1.5rem;
		font-weight: bold;
		margin-bottom: 0.1rem;
	}

	article p {
		font-size: 1rem;
		margin-bottom: 0;
	}

	article header {
		margin-bottom: 1rem;
	}

	.made {
		padding: 1rem;
	}

	:global(.dihentikan_sementara) {
		border: 1px solid #fb8c00;
	}

	:global(.dicabut) {
		border: 1px solid #e53935;
	}
</style>
