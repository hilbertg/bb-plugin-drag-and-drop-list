<script>
	import { getContext } from "svelte"
	//from https://github.com/jhubbardsf/svelte-sortablejs
  	import { SortableList } from '@jhubbardsf/svelte-sortablejs';

  	const { styleable, API, Provider, notificationStore } = getContext("sdk")
  	const component = getContext("component")

	//from schema
	export let orderColumn
	export let textColumn
	export let onRelease
	export let dataProvider
	export let backgroundColor
	export let borderColor
	export let borderRadius
	export let borderWidth
	export let handleIcon
	export let handleSize
	export let animationTime
	export let ghostbgColor
	export let orderLblSize
	export let orderLblColor
	export let lblSize
	export let lblColor
	export let gap
	export let gapLbl
	export let isShowingOrderLbl

	$: orderColumn, textColumn

	//from https://github.com/YuanZhang98/budibase-spotify-playlist
	$: items = dataProvider?.rows?.map((obj) => {
    return {
        id: obj.id,
        name: obj[textColumn],
        order: obj[orderColumn],
        _id: obj._id,
        tableId: obj.tableId
    };
	})?.sort((a, b) => a.order - b.order) ?? [];

	//for ghost background color change within budibase ui
	function startDrag(event) {
		const ghostcolor = ghostbgColor;

		const elements = document.querySelectorAll('[class*="ghost-class"]');

		elements.forEach(element => {
			element.style.backgroundColor = ghostcolor;
		});
	}

	//run after drag finished
	function handleEnd(event) {
		[].forEach.call(event.from.getElementsByClassName('list-group-item'), function (items,index) {
			items.setAttribute("order",index);
		});

		var elemente = document.getElementsByClassName('list-group-item');

		var storedData = [];

		Array.from(elemente).forEach(function(item) {
			var dataId = item.getAttribute('data-id');
			var order = item.getAttribute('order');

			var dataObject = {
				dataId: +dataId,
				order: +order
			};
			storedData.push(dataObject);
		});

		for (let i = 0; i < storedData.length; i++) {
			const storedItem = storedData[i];
			for (let j = 0; j < items.length; j++) {
				const item = items[j];

				//restore color, hacky way because i couldnt get {ghostbgColor} from bb into the css style .ghost-class
				let divToChange = document.querySelector(`div[data-id="${item.id}"]`);
				if (divToChange) {
					const backgroundcolor = backgroundColor;
					divToChange.style.backgroundColor = ''; 
					divToChange.style.backgroundColor = backgroundcolor;
				}

				if (storedItem.dataId === item.id) {
						saveNewOrder(storedItem.order, item._id, item.tableId);
				}
			}
		}

		//update labels
		const orderLabels = document.querySelectorAll('.order-lbl');
		orderLabels.forEach((label, index) => {
			label.textContent = index + 1;
		});

		//on change event
		if (onRelease) {onRelease();}

	}//handleEnd

		//from https://github.com/aptkingston/budibase-comment-box
		const saveNewOrder = async (newOrder, rowId, tableId) => {
			try {
				const row = await getRow(rowId, tableId)
				await API.saveRow({
				...row,
				[orderColumn]: newOrder + 1,
				})
			} catch (error) {
				notificationStore.actions.error("Failed to save new order")
				console.error(error)
			}
		}
		//also from aptkingston
		const getRow = async (rowId, tableId) => {
			if (!tableId || !rowId || !orderColumn) {
				return null
			}
			return await API.fetchRow(tableId, rowId)
		}
</script>

<div use:styleable={$component.styles}>
		<SortableList
		ghostClass="ghost-class"
		animation={animationTime} 
		handle=".cursor-grab"
		onEnd={handleEnd}
		onStart={startDrag}
		>
			{#each items as item}
				<div class="list-group-item" order={item.order} data-id={item.id} 
				style=
				"--backgroundColor:{backgroundColor};
				--borderColor:{borderColor};
				--borderWidth:{borderWidth};
				--borderRadius:{borderRadius};
				--gap: {gap};"
				>
					<div class="item" style="display: flex; align-items: center;">
						{#if isShowingOrderLbl}
							<div class="order-lbl" style=
							"display: inline-block;
							--orderLblSize: {orderLblSize};
							--orderLblColor: {orderLblColor};
							--gapLbl: {gapLbl};
							">{item.order}</div>
						{/if}
						<div class="lbl" style=
						"display: inline-block;
						--lblSize: {lblSize};
						--lblColor: {lblColor};
						">{item.name}</div>
					</div>
					<div class="cursor-grab">
						<i class="{handleIcon} {handleSize}"></i>
					</div>
				</div>
			{/each}
		</SortableList>
</div>

<style>
	.list-group-item {
		display: flex;
		align-items: center;
		position: relative;
		padding: 10px;
		width: 100%;
		margin-bottom: var(--gap, 10px);
		background-color: var(--backgroundColor);
		border-color: var(--borderColor);
		border-radius: var(--borderRadius);
		border-width: var(--borderWidth);
		border-style: solid;
	}

	.ghost-class {
		background-color: #c9ebfa;
		
	}

	.cursor-grab{
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		cursor: grab;
	}

	.item {
		flex: 1;
		padding: 10px;
	}

	.order-lbl {
		font-size: var(--orderLblSize, 24px);
		color: var(--orderLblColor);
		margin-right: var(--gapLbl, 10px);
	}

	.lbl {
		font-size: var(--lblSize, 24px);
		color: var(--lblColor);
	}
</style>