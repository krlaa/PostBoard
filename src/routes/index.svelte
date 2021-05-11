<script>
	// Inspired by https://svelte.dev/repl/810b0f1e16ac4bbd8af8ba25d5e0deff?version=3.4.2.
	import Card from '$lib/Card/card.svelte';
	import MediaQuery from '$lib/Global/MediaQuery.svelte';
	import { flip } from 'svelte/animate';
	import Delete from 'svelte-material-icons/Delete.svelte';
	import Redo from 'svelte-material-icons/Redo.svelte';

	import Up from 'svelte-material-icons/ArrowUp.svelte';
	import Down from 'svelte-material-icons/ArrowDown.svelte';

	import Right from 'svelte-material-icons/ArrowRight.svelte';
	import Left from 'svelte-material-icons/ArrowLeft.svelte';

	import Edit from 'svelte-material-icons/Pen.svelte';

	import { fade } from 'svelte/transition';
	import Post from '../post.js';
	let spin = false;
	let open = false;
	let trashBin = [];
	let curerentIndex;
	let baskets = [
		{
			name: 'Todo',
			items: [new Post('Fix the new item on the main server')],
			section: 'red'
		},
		{
			name: 'In Progress',
			items: [new Post('This is a test post see if it works')],
			section: 'yellow'
		},
		{
			name: 'Review',
			items: [new Post('Try deleting me then press the redo button to bring me back')],
			section: 'green'
		}
	];
	let content = '';
	let hoveringOverBasket;
	function addPost() {
		if (content != '') {
			baskets[basketIndex].items[itemIndex].content = content;
		} else {
			baskets[0].items.push(new Post(content));
		}
		baskets = baskets;
		content = '';
		hidden = true;
	}
	function editPost(basketIndex, itemIndex) {
		open = !open;
		hidden = !hidden;
		content = baskets[basketIndex].items[itemIndex].content;
		// hidden = true;
	}
	function toggle_add() {
		open = !open;
		spin = true;
		setTimeout(() => {
			spin = false;
		}, 500);
		// hidden = !hidden;
	}
	function redo() {
		let item = trashBin.pop();

		baskets[item.basketIndex].items.splice(item.itemIndex, 0, item.conent);
		baskets = baskets;

		console.log('put back');
	}

	function dragStart(event, basketIndex, itemIndex) {
		// The data we want to make available when the element is dropped
		// is the index of the item being dragged and
		// the index of the basket from which it is leaving.
		curerentIndex = basketIndex;
		const data = { basketIndex, itemIndex };
		event.dataTransfer.setData('text/plain', JSON.stringify(data));
	}

	function drop(event, basketIndex) {
		event.preventDefault();
		console.log('well');
		const json = event.dataTransfer.getData('text/plain');
		const data = JSON.parse(json);

		// Remove the item from one basket.
		// Splice returns an array of the deleted elements, just one in this case.
		const [item] = baskets[data.basketIndex].items.splice(data.itemIndex, 1);

		// Add the item to the drop target basket.
		baskets[basketIndex].items.push(item);
		baskets = baskets;

		hoveringOverBasket = null;
	}
	function switchPost(basketIndex, itemIndex, changeFactor) {
		console.log('I trt');

		// Remove the item from one basket.
		// Splice returns an array of the deleted elements, just one in this case.
		const [item] = baskets[basketIndex].items.splice(itemIndex, 1);
		console.log(item);
		// Add the item to the drop target basket.
		baskets[basketIndex + changeFactor].items.push(item);
		baskets = baskets;

		hoveringOverBasket = null;
	}
	let hidden = true;
	function trash(event) {
		event.preventDefault();
		const json = event.dataTransfer.getData('text/plain');
		const data = JSON.parse(json);

		// Remove the item from one basket.
		// Splice returns an array of the deleted elements, just one in this case.
		const [item] = baskets[data.basketIndex].items.splice(data.itemIndex, 1);

		console.log(data);
		// Add the item to the drop target basket.
		trashBin.push({ ...data, conent: item });
		console.log(trashBin);
		baskets = baskets;

		hoveringOverBasket = null;
	}
	function deleteIcon(basketIndex, itemIndex) {
		// Remove the item from one basket.
		// Splice returns an array of the deleted elements, just one in this case.
		const [item] = baskets[basketIndex].items.splice(itemIndex, 1);

		// Add the item to the drop target basket.
		trashBin.push({ basketIndex: basketIndex, itemIndex: itemIndex, conent: item });

		console.log(trashBin);
		baskets = baskets;

		hoveringOverBasket = null;
	}
	let size = '3em';
	let width = size;
	let height = size;
	let color = 'black';
	// export let viewBox = '0 0 24 24';
</script>

{#if !hidden}
	<div
		class="z-30 absolute h-screen w-full bg-transparent cursor-pointer"
		on:click={() => {
			console.log('hello');
			hidden = true;
		}}
	/>
	<div
		in:fade={{ duration: 300 }}
		out:fade={{ duration: 300 }}
		class="z-50 fixed left-0 right-0 bottom-1/4 h-1/2 editor mx-auto w-10/12 flex flex-col text-gray-800 p-4 shadow-lg max-w-2xl bg-gray-600 rounded-xl justify-evenly"
	>
		<textarea
			bind:value={content}
			class="description bg-gray-100 sec p-3 h-60 outline-none rounded-md"
			spellcheck="false"
			placeholder="Describe everything about this post here"
		/>
		<!-- icons -->
		<div class="icons flex text-gray-500 m-2">
			<div class="count ml-auto text-gray-400 text-xs font-semibold">0/300</div>
		</div>
		<!-- buttons -->
		<div class="buttons flex">
			<div
				on:click={() => (hidden = true)}
				class="btn p-1 px-4 font-semibold rounded-md cursor-pointer text-red-100 bg-red-600 ml-auto"
			>
				Cancel
			</div>
			<div
				on:click={addPost}
				class="btn border border-indigo-500 p-1 px-4 font-semibold rounded-md cursor-pointer text-gray-200 ml-2 bg-indigo-500"
			>
				Post
			</div>
		</div>
	</div>
{/if}
<div class="bg-gray-800 flex flex-col z-0">
	<div class="justify-self-center self-end p-8" on:click={redo}>
		<Redo color={'white'} {size} {width} {height} />
	</div>
	<div class="grid lg:grid-cols-{baskets.length} gap-2 p-10 md:grid-cols-2 grid-cols-1">
		{#each baskets as basket, basketIndex (basket)}
			<div
				class="bg-gray-700 lg:h-screen md:h-screen h-auto p-5 rounded-md flex flex-col space-y-6"
				class:bg-yellow-900={hoveringOverBasket === basket.name}
			>
				<div class="text-{basket.section}-400 flex justify-between">
					{basket.name}
					<div
						class="text-center bg-green-300 px-3 rounded-full flex items-center justify-center text-gray-900"
					>
						{basket.items.length}
					</div>
				</div>
				<ul
					class="flex flex-col h-full pb-44 w-full"
					on:dragenter={() => (hoveringOverBasket = basket.name)}
					on:drop={(event) => drop(event, basketIndex)}
					ondragover="return false"
				>
					{#each basket.items as item, itemIndex (item)}
						<div class="inline" animate:flip in:fade={{ duration: 300 }}>
							<li
								class="my-2 inline-block cursor-move w-full"
								draggable={true}
								on:dragstart={(event) => dragStart(event, basketIndex, itemIndex)}
							>
								{#key baskets}
									<div
										class="shadow rounded-lg min-w-full bg-gray-200 border-l-8 border-{basket.section}-500 my-2"
									>
										<div class="flex flex-col p-4">
											<div class="flex-1">
												<div class="flex flex-col items-baseline">
													<p class="text-sm font-semibold">
														{item.content}
													</p>
													<p class="text-gray-500 text-xs">{item.date.toDateString()}</p>
												</div>
											</div>
											<div class="flex flex-row space-x-4 pt-5 justify-between overflow-x-auto ">
												<div class="flex flex-row space-x-5 cursor-pointer">
													<MediaQuery query="(max-width: 768px)" let:matches>
														{#if matches}
															{#if !(basketIndex == 0)}
																<div on:click={() => switchPost(basketIndex, itemIndex, -1)}>
																	<Up {color} {size} width={20} height={20} />
																</div>
															{/if}
															{#if !(basketIndex == baskets.length - 1)}
																<div on:click={() => switchPost(basketIndex, itemIndex, 1)}>
																	<Down {color} {size} width={20} height={20} />
																</div>
															{/if}
														{:else}
															{#if !(basketIndex == 0)}
																<div on:click={() => switchPost(basketIndex, itemIndex, -1)}>
																	<Left {color} {size} width={20} height={20} />
																</div>
															{/if}
															{#if !(basketIndex == baskets.length - 1)}
																<div on:click={() => switchPost(basketIndex, itemIndex, 1)}>
																	<Right {color} {size} width={20} height={20} />
																</div>
															{/if}
														{/if}
													</MediaQuery>
												</div>

												<div class="flex flex-row space-x-5 cursor-pointer">
													<div on:click={() => deleteIcon(basketIndex, itemIndex)}>
														<Delete {color} {size} width={20} height={20} />
													</div>
													<div on:click={() => editPost(basketIndex, itemIndex)}>
														<Edit {color} {size} width={20} height={20} />
													</div>
												</div>
											</div>
										</div>
									</div>
								{/key}
							</li>
						</div>
					{/each}
				</ul>
			</div>
		{/each}
		<button
			class:animate-spin={spin === true}
			on:click={toggle_add}
			class="fixed bottom-10 right-16 p-0 w-16 h-16 bg-red-600 rounded-full hover:bg-red-700 active:shadow-lg mouse shadow transition ease-in duration-200 focus:outline-none flex flex-row items-center justify-center"
		>
			<svg
				class:rotate-45={open === true}
				viewBox="0 0 20 20"
				enable-background="new 0 0 20 20"
				class="w-6 h-6 inline-block transform"
			>
				<path
					fill="#FFFFFF"
					d="M16,10c0,0.553-0.048,1-0.601,1H11v4.399C11,15.951,10.553,16,10,16c-0.553,0-1-0.049-1-0.601V11H4.601
                                    C4.049,11,4,10.553,4,10c0-0.553,0.049-1,0.601-1H9V4.601C9,4.048,9.447,4,10,4c0.553,0,1,0.048,1,0.601V9h4.399
                                    C15.952,9,16,9.447,16,10z"
				/>
			</svg>
		</button>
	</div>
</div>
