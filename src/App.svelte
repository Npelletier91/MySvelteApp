<script>
	import { onDestroy } from 'svelte';
	import { scheduleStore } from './schedule-store';

	import Calendar from './Calendar.svelte';
	import Scheduler from './Scheduler.svelte';
	
	let schedule = {};
	const unsubscribe = scheduleStore.subscribe(currState => {
		schedule = currState;
		localStorage.setItem("storedSchedule", schedule)
	});
	
	onDestroy(() => {
		if (unsubscribe) unsubscribe();
	});
	
	let schedulerShowing = false;	
	let dateID = "";
	let dateHeading = "";
	let monthBackground = ""; // To hold month color


	$: appointments = schedule[dateID];
	
	const makeDateHeading = () => {
    let dateAsHeading = dateID.replace(/_/g, " ");
    let date = new Date(dateAsHeading);
    dateHeading = date.toLocaleString("en-US", { day: 'numeric', month: 'long', year: 'numeric' });
};

	
	const handleScheduler = async (e) => {
		schedulerShowing = true;
		dateID = e.detail.dateID;
		monthBackground = e.detail.monthBackground;
		makeDateHeading();
	}
	
	const removeEmptyDate = () => {
		if (schedule[dateID] && schedule[dateID].length === 0) {
			scheduleStore.update(currDataState => {
				delete currDataState[dateID];
				return currDataState
			});
		}
		return;
	}
	
	const closeScheduler = () => {
		schedulerShowing = false;
		removeEmptyDate();
	}
	
	const setApptToSch = (e) => {
		console.log(e.detail)
		let time = `${e.detail.hour}:${e.detail.minutes < 10 ? '0'+e.detail.minutes : e.detail.minutes}${e.detail.amOrPM}`;
		
		let newAppt = {
			id: Math.floor(Math.random() * 1000000),
			eventname: e.detail.eventName,
			time: time === ":0" ? "no time set" : time,
			completed: false
		};
		
		scheduleStore.update(currState => {
        if (!currState[dateID]) {
            // Create a new array if no appointments for the current date
            currState[dateID] = [newAppt];
        } else {
            // Append to the existing appointments array
            currState[dateID] = [...currState[dateID], newAppt];
        }
        return currState;
    });
	}
	
	$: console.log(schedule)
</script>


<main>
    <div class="container">
		<Calendar on:click={handleScheduler} {schedule} />
        {#if schedulerShowing}
            <Scheduler 
                on:modalClose={closeScheduler}
                on:addAppt={setApptToSch}
                {dateID}
                {dateHeading}
				{monthBackground} 
                {appointments}
            />
        {/if}
    </div>
</main>

<style>
    main {
        font-family: Verdana, sans-serif;
    }
    
    /* Container for max width */
    .container {
        max-width: 720px;   /* Maximum width */
        width: 100%;        /* Responsive for smaller screens */
        margin: 0 auto;     /* Center the container */
        padding: 0 16px;    /* Optional padding for small screens */
    }
    
</style>
