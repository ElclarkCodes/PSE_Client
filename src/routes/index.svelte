<script>
	import '@picocss/pico/css/pico.min.css'

	import { fade } from 'svelte/transition'

	// import moment from 'moment'

	let list = []
	let category = 'asing_terdaftar'
	let search = ''
	let limit = 10
	let offset = 0

	let loading = true
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
			`https://api.elclark.my.id/pse/${category}?search=${search}&limit=${limit}&offset=${offset}`
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

		if (search !== '' && list.length === 0) {
			notFound = true
		}

		// set loading
		loading = false
	}

	// load data
	reloadData()

	// load more data
	async function loadMoreData() {
		// set loading
		loadingMore = true

		// set offset
		offset += limit

		// get data
		let response = await fetch(
			`https://api.elclark.my.id/pse/${category}?search=${search}&limit=${limit}&offset=${offset}`
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

	// function getDate(date) {
	// 	return moment(date).locale('id').format('DD MMMM YYYY')
	// }
</script>

<div>
	<header>
		<h1>Check PSE</h1>
	</header>

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

	<form on:submit|preventDefault={doSearch}>
		<input type="text" name="search" placeholder="Cari" on:change={doSearch} bind:value={search} />
		<select name="type" bind:value={category} on:change={reloadData}>
			<option value="asing_terdaftar">Asing Terdaftar</option>
			<option value="asing_dihentikan_sementara">Asing Dihentikan Sementara</option>
			<option value="asing_dicabut">Asing Dicabut</option>
			<option value="lokal_terdaftar">Lokal Terdaftar</option>
			<option value="lokal_dihentikan_sementara">Lokal Dihentikan Sementara</option>
			<option value="lokal_dicabut">Lokal Dicabut</option>
		</select>
		<button>Cari</button>
	</form>

	<div class="list">
		<noscript>
			<div>
				<p>
					<b>Peringatan:</b> Sistem tidak mendukung JavaScript. Silahkan aktifkan JavaScript untuk menggunakan
					Aplikasi ini!
				</p>
			</div>
		</noscript>
		{#each list as item}
			<article in:fade>
				<h2>{item.nama}</h2>
				{#if fixURL(item.website)}
					<a href={fixURL(item.website)}>{fixURL(item.website)}</a>
				{/if}
				<p>{item.nama_perusahaan}</p>
				<!-- <p>
						{getDate(item.attributes.tanggal_daftar)}
					</p> -->
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
				aria-busy={loadingMore}
				on:click={loadMoreData}
				class="secondary"
				on:scroll={loadMoreData}>Muat Lainnya</button
			>
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

		margin-top: 1rem;
		margin-bottom: 2rem;
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
</style>
