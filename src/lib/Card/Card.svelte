<script>
	export let content;
	export let date;
	export let section;
	$: notneek = section;
</script>

<div class="shadow rounded-lg min-w-full bg-gray-200 border-l-8 border-{section}-500 my-2">
	<div class="flex items-center space-x-4 p-4">
		<div class="flex-1">
			<div class="flex flex-col items-baseline">
				<p class="text-sm font-semibold">
					{content}
				</p>
				<p class="text-gray-500 text-xs">{date.toDateString()}</p>
			</div>
			<div class="flex flex-row space-x-4">
				<p>hi</p>
				<p>hi</p>
			</div>
		</div>
	</div>
</div>
