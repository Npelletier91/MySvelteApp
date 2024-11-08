<script>
    import { createEventDispatcher } from 'svelte';
	const dispatch = createEventDispatcher();

    export let monthBackground; // Make monthBackground available as a prop
	export let schedule;


	const handleDayClick = (event, dateID, monthBackground) => {
        dispatch('click', { dateID, monthBackground });
    };
	
	const date = new Date();
	
	const today = {
		dayNumber: date.getDate(),
		month: date.getMonth(),
		year: date.getFullYear(),
	}
	
	const monthNames = [ "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
	let monthIndex = date.getMonth();
	const currentMonth = date.toLocaleString('en-US', { month: 'long' })
	$: month = monthNames[monthIndex];
	
	let year = date.getFullYear();
	
	$: firstDayIndex = new Date(year, monthIndex, 1).getDay();
	const currentDay = date.getDate();
	$: numberOfDays = new Date(year, monthIndex+1, 0).getDate();
	
	// Calculates how many days are in the previous month by using Date( , ,0)
	$: previousMonthDays = new Date(year, monthIndex, 0).getDate();

	let calendarCellsQty; 
	$: {
		if (firstDayIndex + numberOfDays <= 28) {
			calendarCellsQty = 28;  // 4 rows (only for February in specific cases)
		} else if (firstDayIndex + numberOfDays <= 35) {
			calendarCellsQty = 35;  // 5 rows
		} else {
			calendarCellsQty = 42;  // 6 rows
		}
	}
	
	const goToNextMonth = () => {
		if (monthIndex >= 11) {
			year += 1;
			return monthIndex = 0;
		}
		return monthIndex += 1;
	}
	
	const goToPrevMonth = () => {
		if (monthIndex <= 0) {
			year -= 1;
			return monthIndex = 11;
		}
		return monthIndex -= 1;
	}

	const monthColors = [
    '#1E90FF', // January - Wintery blue
    '#00BFFF', // February - Light blue
    '#FFD700', // March - Gold (early spring)
    '#32CD32', // April - Green
    '#FF69B4', // May - Pink (spring blossom)
    '#FF4500', // June - Red-orange (start of summer)
    '#FF6347', // July - Tomato red
    '#FFDAB9', // August - Peach
    '#FFA07A', // September - Light salmon
    '#FF8C00', // October - Pumpkin orange
    '#4682B4', // November - Steel blue
    '#708090'  // December - Slate gray
	];

	// Reactive change to background depending on monthIndex
$: monthBackground = monthColors[monthIndex];


$: console.log(`${month}, ${today.dayNumber}, ${year}, FIRST DAY index is ${firstDayIndex}, MONTH index is ${monthIndex}, No. of days: ${numberOfDays}`)
</script>


<div class="month" style="background-color: {monthBackground}">
	<ul>
		<li>
					<!-- Reacts to on:click because monthIndex changes -->				
			<button class="prev" on:click={goToPrevMonth}>&#10094;</button>
			<button class="next" on:click={goToNextMonth}>&#10095;</button>
			</li>
			
		<li>{month}<br>
			<span style="font-size:18px">{year}</span>
		</li>
	</ul>
</div>

<ul class="weekdays">
	<li>Su</li>
	<li>Mo</li>
	<li>Tu</li>
	<li>We</li>
	<li>Th</li>
	<li>Fr</li>
	<li>Sa</li>
</ul>

<ul class="days" style="--month-color: {monthBackground}; --month-color-transparent: {monthBackground}40;">
	{#each Array(calendarCellsQty) as _, i}
		{#if i < firstDayIndex}
			<!-- Display days from the previous month -->
			<li class="other-month">
				{previousMonthDays - (firstDayIndex - i - 1)}
			</li>
		{:else if i >= firstDayIndex + numberOfDays}
			<!-- Display days from the next month -->
			<li class="other-month">{(i - firstDayIndex - numberOfDays) + 1}</li>
		{:else}
			<!-- Display days from the current month -->
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<li
				class:active={i === today.dayNumber + (firstDayIndex - 1) &&
					monthIndex === today.month &&
					year === today.year}
				data-dateID={`${month}_${(i - firstDayIndex) + 1}_${year}`}
				class:has-appts={`${month}_${(i - firstDayIndex) + 1}_${year}` in schedule}
                on:click={(event) => handleDayClick(event, `${month}_${(i - firstDayIndex) + 1}_${year}`, monthBackground)}
				>
				{(i - firstDayIndex) + 1}
			</li>
		{/if}
	{/each}
</ul>

				
<style>

	ul {list-style-type: none;}

	/* Month header */
	.month {
		padding: 70px 25px;
		border-top-left-radius: 10px;
		border-top-right-radius: 10px;
		width: auto;
		text-align: center;
	}

	/* Month list */
	.month ul {
		margin: 0;
		padding: 0;
	}

	.month ul li {
		color: white;
		font-size: 20px;
		text-transform: uppercase;
		letter-spacing: 3px;
	}

	.days li.other-month {
        color: #ccc; /* Lighter grey for days from the previous and next month */
    }

	/* Previous button inside month header */
	.month .prev {
		float: left;
		padding-top: 10px;
		cursor: pointer;
		border-radius: 10px;
	}

	/* Next button */
	.month .next {
		float: right;
		padding-top: 10px;
		cursor: pointer;
		border-radius: 10px	;
	}


	/* Weekdays (Mon-Sun) */
	.weekdays {
		margin: 0;
		padding: 10px 0;
		background-color:#ddd;
	}

	.weekdays li {
		display: inline-block;
		width: 13.4%;
		color: #666;
		text-align: center;
	}

	/* Days (1-31) */
	.days {
		padding: 10px 0;
		border-bottom-left-radius: 10px;
		border-bottom-right-radius: 10px;
		background: #eee;
		margin: 0;
	}

	.days li {
		list-style-type: none;
		display: inline-block;
		border: 1px solid black;
		border-radius: 50px;
		border: transparent;
		margin: 5px;
		padding: 4px;
		width: 11.6%;
		text-align: center;
		margin-bottom: 1px;
		font-size: 1.2rem;
		color: #777;
		cursor: pointer;
	}

	    /* Hover effect to use the month color */
	.days li:hover {
        background-color: var(--month-color);
        color: white;
    }

	/* Highlight the "current" day */
	.active {
		padding: 5px;
		background: #ffffff;
		color: white;
	}
	
	.days li.has-appts {
		background: var(--month-color-transparent);
	}
	.days li.has-appts:hover {
        background-color: var(--month-color);
	}
</style>