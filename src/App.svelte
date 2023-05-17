<script lang="ts">
	let drop_zone: HTMLDivElement;
	let objects = [
		{ el: null, id: 1 },
		{ el: null, id: 2 },
		{ el: null, id: 3 }
	];

	let dropped = [];
	let status = "";

	let dropped_in = false;
	let activeEvent = "";
	let originalX = "";
	let originalY = "";

	function handleDragEnter(e: DragEvent) {
		status = "You are dragging over the " + (e.target as HTMLElement).getAttribute("id");
	}

	function handleDragLeave(e: DragEvent) {
		status = "You left the " + (e.target as HTMLElement).getAttribute("id");
	}

	function handleDragDrop(e: DragEvent) {
		e.preventDefault();
		var element_id = e.dataTransfer.getData("text");
		dropped = dropped.concat(element_id);
		dropped_in = true;
		status = "You dropped " + element_id + " into drop zone";
	}

	function handleDragStart(e: DragEvent) {
		status = "Dragging the element " + (e.target as HTMLElement).getAttribute("id");
		e.dataTransfer.dropEffect = "move";
		e.dataTransfer.setData("text", (e.target as HTMLElement).getAttribute("id"));
	}

	function handleDragEnd(e: DragEvent) {
		if (dropped_in == false) {
			status = "You let the " + (e.target as HTMLElement).getAttribute("id") + " go.";
		}
		dropped_in = false;
	}

	function handleTouchStart(e: TouchEvent) {
		status = "Touch start with element " + (e.target as HTMLElement).getAttribute("id");
		originalX = (e.target as HTMLElement).offsetLeft - 10 + "px";
		originalY = (e.target as HTMLElement).offsetTop - 10 + "px";
		activeEvent = "start";
	}

	function handleTouchMove(e: TouchEvent) {
		let touchLocation = e.targetTouches[0];
		let pageX = Math.floor(touchLocation.pageX - 50) + "px";
		let pageY = Math.floor(touchLocation.pageY - 50) + "px";
		status = "Touch x " + pageX + " Touch y " + pageY;
		(e.target as HTMLElement).style.position = "absolute";
		(e.target as HTMLDivElement)["left"] = pageX;
		(e.target as HTMLDivElement)["top"] = pageY;
		activeEvent = "move";
	}

	function handleTouchEnd(e: TouchEvent) {
		e.preventDefault();
		if (activeEvent === "move") {
			let pageX = parseInt((e.target as HTMLElement).style.left) - 50;
			let pageY = parseInt((e.target as HTMLElement).style.top) - 50;

			if (
				detectTouchEnd(
					drop_zone.offsetLeft,
					drop_zone.offsetTop,
					pageX,
					pageY,
					drop_zone.offsetWidth,
					drop_zone.offsetHeight
				)
			) {
				dropped = dropped.concat((e.target as HTMLElement).id);
				(e.target as HTMLElement).style.position = "initial";
				dropped_in = true;
				status = "You dropped " + (e.target as HTMLElement).getAttribute("id") + " into drop zone";
			} else {
				(e.target as HTMLElement).style.left = originalX;
				(e.target as HTMLElement).style.top = originalY;
				status = "You let the " + (e.target as HTMLElement).getAttribute("id") + " go.";
			}
		}
	}

	function detectTouchEnd(x1, y1, x2, y2, w, h) {
		//Very simple detection here
		if (x2 - x1 > w) return false;
		return y2 - y1 <= h;
	}
</script>

<h2 id="app_status">Drag status: {status}</h2>
<h1>Drop Zone</h1>

<div
	on:dragenter={handleDragEnter}
	on:dragleave={handleDragLeave}
	on:drop={handleDragDrop}
	bind:this={drop_zone}
	id="drop_zone"
	ondragover="return false"
>
	{#each objects.filter((v) => dropped.includes(`${v.id}`)) as { id }}
		<div class="objects" {id} style="cursor: auto">
			Object {id}
		</div>
	{/each}
</div>

{#each objects.filter((v) => !dropped.includes(`${v.id}`)) as { id }, i}
	<div
		{id}
		class="objects"
		draggable="true"
		bind:this={objects[i].el}
		on:dragstart={handleDragStart}
		on:dragend={handleDragEnd}
		on:touchstart={handleTouchStart}
		on:touchmove={handleTouchMove}
		on:touchend={handleTouchEnd}
	>
		Object {id}
	</div>
{/each}

<style>
	:global(html),
	:global(body) {
		margin: 0;
		height: 100%;
		overflow: hidden;
		user-select: none;
		-webkit-user-select: none;
	}

	#drop_zone {
		background-color: #eee;
		border: #999 1px solid;
		width: 280px;
		height: 200px;
		padding: 8px;
		font-size: 19px;
	}

	.objects {
		display: inline-block;
		background-color: #fff3cc;
		border: #dfbc6a 1px solid;
		width: 50px;
		height: 50px;
		margin: 10px;
		padding: 8px;
		font-size: 18px;
		text-align: center;
		box-shadow: 2px 2px 2px #999;
		cursor: move;
	}
</style>
