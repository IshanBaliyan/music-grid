<script>
	import { createEventDispatcher, onMount } from 'svelte';
	import ClipboardJS from 'clipboard';
	import { scale_keys } from './Music.svelte';

	const dispatch = createEventDispatcher();

	export let grid;
	export let config;

	let urlUpdatedRecently = false;
	let scrollY = 0;

	const encodeGridToUrl = (grid, speed, scale) => {
		let res = ''
		for (var i = grid.length - 1; i >= 0; i--) {
			let temp = 0, k = 1;
			for (var j = grid[i].length - 1; j >= 0; j--) {
				temp = temp + k * grid[i][j];
				k = k * 2;
			}
			res += (temp + '-');
		}
		history.replaceState({}, '', '#' + res + '&' + speed + '&' + scale);
	}

	const updateUrl = (grid, speed, scale) => {
		if(!urlUpdatedRecently) {
			encodeGridToUrl(grid, speed, scale);
			urlUpdatedRecently = true;
			setTimeout(() => {urlUpdatedRecently = false}, 1000);
		}
	}

	$: updateUrl(grid, config.speed, config.scale_key);


	let clipboard = new ClipboardJS('.share', {
		text: function() {
			encodeGridToUrl(grid, config.speed, config.scale_key);
			return window.location.href;
		}
	});


	let header;
	let headerOffset;
	let primaryClass = "primary";

	onMount(() => {
		headerOffset = header.offsetTop;
	});

	const showSticky = (ignored) => {
		if(headerOffset) {
			if (window.pageYOffset > headerOffset) {
				primaryClass = "primary sticky";
			} else {
				primaryClass = "primary";
			}
		}
	}

	$: showSticky(scrollY);

</script>

<style>

	.fa {
		transition: 0.06s ease-in;
	}

	.fa:hover {
		transform: scale(1.1);
	}

	.fa:active {
		transform: scale(0.8);
	}

	a {
		margin-right: 4px;
		margin-left: 4px;
		padding: 12px;
		color: white;
	}

	.primary, .settings {
		padding: 10px;
		background: black;
	}

	.sticky {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
	}

	select {
		background: black;
		color: white;
		border: none;
		outline: none;
	}
</style>
<svelte:window bind:scrollY={scrollY}/>

<div class="container">
	<div class="settings">
		<br/>
		<label>
			Scale :
			<select bind:value={config.scale_key} on:change={() => dispatch('scalechange')}>
				{#each scale_keys as scale}
				<option value={scale}>
					{scale}
				</option>
				{/each}
			</select>
		</label>
	</div>
	<div class={primaryClass} bind:this={header}>
		<a on:click={() => dispatch('playpause')}>
			{#if config.playing}
				<i class="fa fa-lg fa-inverse fa-pause"/>
			{:else}
				<i class="fa fa-lg fa-inverse fa-play"/>
			{/if}
		</a>
		<a on:click={() => dispatch('stop')}><i class="fa fa-inverse fa-lg fa-stop"/></a>
		<a on:click={() => dispatch('clear')}><i class="fa fa-lg fa-trash"/></a>
		<a on:click={() => alert('Link copied to clipboard. Paste it to share')} class="share"><i class="fa fa-lg fa-share-alt"/></a>
		<a on:click={() => dispatch('download')}><i class="fa fa-lg fa-download"/></a>
	</div>
</div>
