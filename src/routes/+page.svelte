<script lang="ts">
	import { onMount } from "svelte";
  
	interface Item {
	  displayname: string;
	  name: string;
	  price: number;
	}
  
	let availableItems: Item[] = [];
	let socket: WebSocket;
  
	async function handleBuyItem(itemName: string) {
	  try {
		const response = await fetch('/api/buy/', {
		  method: 'POST',
		  headers: {
			'Content-Type': 'application/json',
		  },
		  body: JSON.stringify({ item: itemName }),
		});
  
		const data = await response.json();
		console.log(data);
	  } catch (err: any) {
		console.error(err);
	  }
	}
  
	function setupWebSocket() {
	  const protocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
	  const wsUrl = `${protocol}//${window.location.host}/api/ws/`;
  
	  socket = new WebSocket(wsUrl);
  
	  socket.onmessage = (event) => {
		try {
		  availableItems = JSON.parse(event.data);
		} catch (error) {
		  console.error('Error parsing WebSocket data:', error, event.data);
		}
	  };
  
	  socket.onclose = (event) => {
		console.log('WebSocket closed:', event);
		setTimeout(setupWebSocket, 5000);
	  };
  
	  socket.onerror = (error) => {
		console.error('WebSocket error:', error);
	  };
	}
  
	onMount(() => {
	  handleBuyItem('tipp1');
	  setupWebSocket();
	});
  </script>
  
  <div class="flex gap-6 items-center justify-center w-full h-screen">
	{#each availableItems as item}
	  <div class="flex flex-col items-center justify-center text-white text-2xl bg-[#1a1a1a] rounded-lg shadow-lg p-8">
		<h1 class="text-4xl font-semibold">{item.displayname}</h1>
		<p class="mt-2">{item.price} zed</p>
		<button class="mt-8 bg-green-600 hover:bg-green-800 text-white font-bold py-2 px-4 rounded" on:click={() => handleBuyItem(item.name)}>Vásárlás</button>
	  </div>
	{/each}
  </div>
  
  <style>
	:global(html) {
	  background-color: #0a0a0a;
	}
  </style>