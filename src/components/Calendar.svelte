<script>
  import { createEventDispatcher, afterUpdate } from 'svelte';
  const dispatch = createEventDispatcher();

  export let events = [];

  let currentDate = new Date();
  let currentWeek = getStartOfWeek(currentDate);
  let weekDays = [];

  const monthNames = [
    'Január', 'Február', 'Március', 'Április', 'Május', 'Június',
    'Július', 'Augusztus', 'Szeptember', 'Október', 'November', 'December'
  ];

  const dayNames = ['Hétfő', 'Kedd', 'Szerda', 'Csütörtök', 'Péntek', 'Szombat', 'Vasárnap'];

  afterUpdate(() => {
    console.log('Calendar updated with events:', events);
  });

  function getStartOfWeek(date) {
    const newDate = new Date(date);
    const day = newDate.getDay();
    const diff = day === 0 ? -6 : 1 - day; // Ha vasárnap (0), akkor -6, egyébként 1 - day
    newDate.setDate(newDate.getDate() + diff);
    return newDate;
  }

  function getWeekDays(startDate) {
    const days = [];
    for (let i = 0; i < 7; i++) {
      const date = new Date(startDate);
      date.setDate(startDate.getDate() + i);
      days.push(date);
    }
    return days;
  }

  function goToPrevWeek() {
    const newDate = new Date(currentWeek);
    newDate.setDate(newDate.getDate() - 7);
    currentWeek = newDate;
  }

  function goToNextWeek() {
    const newDate = new Date(currentWeek);
    newDate.setDate(newDate.getDate() + 7);
    currentWeek = newDate;
  }

  function formatDate(date) {
    return date.toISOString().split('T')[0];
  }

  function formatHeaderDate(startDate) {
    const endDate = new Date(startDate);
    endDate.setDate(startDate.getDate() + 6);

    const startMonth = monthNames[startDate.getMonth()];
    const endMonth = monthNames[endDate.getMonth()];
    
    if (startMonth === endMonth) {
      return `${startMonth} ${startDate.getDate()}-${endDate.getDate()}.`;
    } else {
      return `${startMonth} ${startDate.getDate()}. - ${endMonth} ${endDate.getDate()}.`;
    }
  }

  function getEventsForDay(date) {
    const dateStr = formatDate(date);
    return events.filter(event => event.date === dateStr);
  }

  function handleDateClick(date) {
    dispatch('dateClick', date);
  }

  function handleEventClick(event, e) {
    e.stopPropagation();
    dispatch('eventClick', event);
  }

  function handleEventDoubleClick(event, e) {
    e.stopPropagation();
    dispatch('toggleComplete', event);
  }

  function formatEventTime(time) {
    return time.slice(0, 5); // Csak az óra és perc megjelenítése
  }

  function formatTimeSlot(hour) {
    return `${hour.toString().padStart(2, '0')}:00`;
  }

  function calculateEventPosition(startTime, endTime) {
    const [startHour, startMinute] = startTime.split(':').map(Number);
    const [endHour, endMinute] = endTime.split(':').map(Number);
    
    const startMinutes = startHour * 30 + startMinute * 0.5;
    const endMinutes = endHour * 30 + endMinute * 0.5;
    
    return {
      top: `${startMinutes}px`,
      height: `${endMinutes - startMinutes}px`
    };
  }

  // Reaktív deklarációk
  $: {
    weekDays = getWeekDays(currentWeek);
    console.log('Week or events updated:', { weekDays, events });
  }
</script>

<div class="calendar">
  <div class="header">
    <button on:click={goToPrevWeek}>&lt;</button>
    <h2>{formatHeaderDate(currentWeek)}</h2>
    <button on:click={goToNextWeek}>&gt;</button>
  </div>

  <div class="week-view">
    <div class="time-column">
      <div class="time-header"></div>
      {#each Array(24) as _, i}
        <div class="time-slot">{formatTimeSlot(i)}</div>
      {/each}
    </div>

    {#each weekDays as day}
      <div class="day-column">
        <div class="day-header" class:today={formatDate(day) === formatDate(new Date())}>
          <div class="day-name">{dayNames[day.getDay() === 0 ? 6 : day.getDay() - 1]}</div>
          <div class="day-date">{day.getDate()}</div>
        </div>
        <div class="day-content" on:click={() => handleDateClick(day)}>
          {#each getEventsForDay(day) as event}
            {@const position = calculateEventPosition(event.startTime, event.endTime)}
            <div 
              class="event"
              class:completed={event.completed}
              style="
                background-color: {event.color};
                top: {position.top};
                height: {position.height};
              "
              on:click|stopPropagation={(e) => handleEventClick(event, e)}
              on:dblclick|stopPropagation={(e) => handleEventDoubleClick(event, e)}
            >
              <div class="event-time">
                {formatEventTime(event.startTime)} - {formatEventTime(event.endTime)}
              </div>
              <div class="event-title">{event.title}</div>
            </div>
          {/each}
          {#each Array(24) as _, hour}
            <div class="hour-line"></div>
          {/each}
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  .calendar {
    width: 100%;
    background: white;
    border-radius: 8px;
    overflow: hidden;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px;
    background-color: #4CAF50;
    color: white;
  }

  .header button {
    background: none;
    border: none;
    color: white;
    font-size: 20px;
    cursor: pointer;
    padding: 5px 10px;
    border-radius: 4px;
  }

  .header button:hover {
    background-color: rgba(255, 255, 255, 0.1);
  }

  .header h2 {
    margin: 0;
    font-size: 1.5em;
  }

  .week-view {
    display: flex;
    height: 720px; /* 24 óra * 30px magasság */
    border-top: 1px solid #ddd;
    overflow-y: auto; /* Görgethetővé tesszük */
  }

  .time-column {
    width: 60px;
    border-right: 1px solid #ddd;
    background: #f5f5f5;
    position: sticky; /* Rögzítjük az időoszlopot */
    left: 0;
    z-index: 2;
  }

  .time-header {
    height: 30px;
    border-bottom: 1px solid #ddd;
    background: #f5f5f5;
    position: sticky;
    top: 0;
    z-index: 3;
  }

  .time-slot {
    height: 30px;
    padding: 0 5px;
    font-size: 11px;
    color: #666;
    text-align: right;
    border-bottom: 1px solid #eee;
    display: flex;
    align-items: center;
    justify-content: flex-end;
  }

  .day-column {
    flex: 1;
    border-right: 1px solid #ddd;
    min-width: 120px;
    position: relative;
  }

  .day-header {
    height: 30px;
    padding: 2px;
    text-align: center;
    border-bottom: 1px solid #ddd;
    background: #f5f5f5;
    position: sticky;
    top: 0;
    z-index: 1;
  }

  .day-header.today {
    background-color: #e8f5e9;
  }

  .day-name {
    font-weight: bold;
    color: #666;
    font-size: 0.9em;
  }

  .day-date {
    font-size: 1em;
    color: #333;
  }

  .day-content {
    position: relative;
    height: calc(100% - 30px);
  }

  .hour-line {
    height: 30px;
    border-bottom: 1px solid #eee;
  }

  .event {
    position: absolute;
    left: 2px;
    right: 2px;
    min-height: 20px;
    padding: 2px;
    border-radius: 4px;
    color: white;
    font-size: 11px;
    overflow: hidden;
    cursor: pointer;
    z-index: 1;
  }

  .event:hover {
    opacity: 0.9;
    z-index: 2;
  }

  .event.completed {
    background-image: repeating-linear-gradient(
      45deg,
      transparent,
      transparent 10px,
      rgba(255, 255, 255, 0.2) 10px,
      rgba(255, 255, 255, 0.2) 20px
    );
    opacity: 0.8;
  }

  .event-time {
    font-size: 0.8em;
    margin-bottom: 1px;
  }

  .event-title {
    font-weight: bold;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  @media (max-width: 768px) {
    .week-view {
      overflow-x: auto;
    }
    
    .day-column {
      min-width: 100px;
    }

    .event {
      font-size: 10px;
    }
  }
</style> 