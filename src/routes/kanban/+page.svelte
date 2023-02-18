<script lang="ts">
	import { flip } from 'svelte/animate';
import { loop_guard } from 'svelte/internal';
	import { send, receive } from './transition.ts';
	let Board = [
		{
			name: 'Random',
			items: ['React', 'Angular', 'Solid', 'Rails', 'Express', 'Flask', 'VPS']
		},
		{
			name: 'Frontend Stack',
			items: ['Svelte', 'SvelteKit']
		},
		{
			name: 'Backend Stack',
			items: ['Django']
		},
		{
			name: 'Server Stack',
			items: ['Vercel']
		}
	];

	
	let draggedRect: DOMRect | undefined;
	
	$: draggedHeight = draggedRect?.height?? 0
	
	let cardHover: number | null = null;
	let laneHover: number | null = null;

	let cardDragged: { laneIndex: number; cardIndex: number };

	/* function dragOver(ev: DragEvent, laneIndex: number, cardIndex: number) {
		ev.preventDefault();
		console.log(`${laneIndex}-${cardIndex}`)
		if (laneIndex !== cardDragged.laneIndex || cardIndex !== cardDragged.cardIndex) {
			const el: Element = document.getElementById(`card-${laneIndex}-${cardIndex}`)
			el.style.transform = `translateY(${draggedRect.height}px)`;
		}
		if (laneIndex !== cardDragged.laneIndex) {
			console.log('Different line');
		} else {
			console.log('Same line');
		}
		if (cardIndex !== cardDragged.cardIndex) {
			console.log('Different card');
		} else {
			console.log('Same card');
		}
	} */


	function dragStart(event: DragEvent, laneIndex: number, cardIndex: number) {
		const draggedElement = document.getElementById(`card-${laneIndex}-${cardIndex}`);
		if (draggedElement) {
			draggedRect = draggedElement?.getBoundingClientRect();
			cardDragged = { laneIndex, cardIndex };
			event.dataTransfer?.setData('text/plain', JSON.stringify(cardDragged));
		}
	}

	function drop(event: DragEvent, laneIndex: number) {
		/* const laneDropIndex = laneIndex
		const cardDropIndex = cardIndex */
		const json = event.dataTransfer?.getData('text/plain');
		if (json) {
			const data = JSON.parse(json);
			const [item] = Board[data.laneIndex].items.splice(data.cardIndex, 1);
			console.log(cardHover)
			if (typeof cardHover === "number") {
				Board[laneIndex].items.splice(cardHover, 0, item);
			} else {
				Board[laneIndex].items.push(item);
			}
		}
		
		Board = Board;
		laneHover = null;
		cardHover = null;
	}
</script>

<!-- Component Start -->
<div
	class="flex flex-col w-screen h-screen overflow-auto text-gray-700 bg-gradient-to-tr from-blue-200 via-indigo-200 to-pink-200"
>
	<div class="px-10 mt-6">
		<h1 class="text-2xl font-bold">Team Project Board</h1>
	</div>
	<!-- BOARD -->
	<div class="flex flex-grow px-10 mt-4 space-x-6 overflow-auto">
		<!-- COLUMN -->
		{#each Board as lane, laneIndex (lane)}
			<!-- in:receive={{ key: laneIndex }} out:send={{ key: laneIndex }} -->
			<div id={`lane-${laneIndex}`} class="flex flex-col shrink-0 w-72" animate:flip>
				<!-- COLUMN TITLE -->
				<div class="flex items-center flex-shrink-0 h-10 px-2">
					<span class="block text-sm font-semibold">{lane.name}</span>
					<span
						class="flex items-center justify-center w-5 h-5 ml-2 text-sm font-semibold text-indigo-500 bg-white rounded bg-opacity-30"
						>6</span
					>
					<button
						class="flex items-center justify-center w-6 h-6 ml-auto text-indigo-500 rounded hover:bg-indigo-500 hover:text-indigo-100"
					>
						<svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M12 6v6m0 0v6m0-6h6m-6 0H6"
							/>
						</svg>
					</button>
				</div>
				<!-- COLUMN CONTAINER -->
				<div
					
					class="flex flex-col grow pb-2 overflow-auto"
					on:drop={(event) => drop(event, laneIndex)}
					on:dragover|preventDefault={() => false}
					on:dragenter|preventDefault={() => (laneHover = laneIndex)}
					on:dragend={() => {laneHover = null}}
				> <!-- overflow-auto -->
					<!-- CARD -->
					{#each lane.items as card, cardIndex (card)}
						<div 
						id={`card-${laneIndex}-${cardIndex}`} 
						class="card"
						style={(cardHover !== null && (laneIndex === laneHover && cardIndex >= cardHover))? `transform: translateY(${draggedHeight}px)` : `transform: translateY(0px)`}
						in:receive={{ key: cardIndex }}
						out:send={{ key: cardIndex }}
						animate:flip={{ duration: 500 }}>
						{laneIndex}
						{laneHover}
						{cardIndex}
						{cardHover}
							<div
								class="relative flex flex-col items-start p-4 mt-3 bg-white rounded-lg cursor-pointer bg-opacity-90 group hover:bg-opacity-100"
								draggable={true}
								on:dragstart={(event) => dragStart(event, laneIndex, cardIndex)}
								on:dragover|preventDefault={() => false}
								on:dragend={() => {cardHover = null}}
							>
							<!-- on:dragenter|preventDefault={() => {cardHover = cardIndex; console.log("enter")}}
								on:dragleave={() => {cardHover = null; console.log("leave")}} -->
								<div class="flex w-full" on:dragenter|preventDefault={() => {cardHover = cardIndex;}}>
									<button
									class="absolute top-0 right-0 flex items-center justify-center hidden w-5 h-5 mt-3 mr-2 text-gray-500 rounded hover:bg-gray-200 hover:text-gray-700 group-hover:flex"
									>
									<svg
										class="w-4 h-4 fill-current"
										xmlns="http://www.w3.org/2000/svg"
										viewBox="0 0 20 20"
										fill="currentColor"
									>
										<path
											d="M10 6a2 2 0 110-4 2 2 0 010 4zM10 12a2 2 0 110-4 2 2 0 010 4zM10 18a2 2 0 110-4 2 2 0 010 4z"
										/>
									</svg>
									</button>
									<span
										class="flex items-center h-6 px-3 text-xs font-semibold text-pink-500 bg-pink-100 rounded-full"
										>Design</span
									>
								</div>
								
								<h4 class="mt-3 text-sm font-medium">{card}</h4>
								<div on:dragleave={() => {cardHover = null;}} class="flex items-center w-full mt-3 text-xs font-medium text-gray-400">
									<div class="flex items-center">
										<svg
											class="w-4 h-4 text-gray-300 fill-current"
											xmlns="http://www.w3.org/2000/svg"
											viewBox="0 0 20 20"
											fill="currentColor"
										>
											<path
												fill-rule="evenodd"
												d="M6 2a1 1 0 00-1 1v1H4a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V6a2 2 0 00-2-2h-1V3a1 1 0 10-2 0v1H7V3a1 1 0 00-1-1zm0 5a1 1 0 000 2h8a1 1 0 100-2H6z"
												clip-rule="evenodd"
											/>
										</svg>
										<span class="ml-1 leading-none">Dec 12</span>
									</div>
									<div class="relative flex items-center ml-4">
										<svg
											class="relative w-4 h-4 text-gray-300 fill-current"
											xmlns="http://www.w3.org/2000/svg"
											viewBox="0 0 20 20"
											fill="currentColor"
										>
											<path
												fill-rule="evenodd"
												d="M18 5v8a2 2 0 01-2 2h-5l-5 4v-4H4a2 2 0 01-2-2V5a2 2 0 012-2h12a2 2 0 012 2zM7 8H5v2h2V8zm2 0h2v2H9V8zm6 0h-2v2h2V8z"
												clip-rule="evenodd"
											/>
										</svg>
										<span class="ml-1 leading-none">4</span>
									</div>
									<div class="flex items-center ml-4">
										<svg
											class="w-4 h-4 text-gray-300 fill-current"
											xmlns="http://www.w3.org/2000/svg"
											viewBox="0 0 20 20"
											fill="currentColor"
										>
											<path
												fill-rule="evenodd"
												d="M8 4a3 3 0 00-3 3v4a5 5 0 0010 0V7a1 1 0 112 0v4a7 7 0 11-14 0V7a5 5 0 0110 0v4a3 3 0 11-6 0V7a1 1 0 012 0v4a1 1 0 102 0V7a3 3 0 00-3-3z"
												clip-rule="evenodd"
											/>
										</svg>
										<span class="ml-1 leading-none">1</span>
									</div>
									<img
										class="w-6 h-6 ml-auto rounded-full"
										src="https://randomuser.me/api/portraits/women/26.jpg"
									/>
								</div>
							</div>
						</div>
					{/each}
				</div>
			</div>
		{/each}
		<div class="flex-shrink-0 w-6" />
	</div>
</div>
<!-- Component End -->
