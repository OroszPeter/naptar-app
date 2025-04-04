<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import Calendar from './components/Calendar.svelte';
  import EventForm from './components/EventForm.svelte';

  // Létrehozunk egy store-t az események kezeléséhez
  const eventsStore = writable([]);
  let events = [];

  // Feliratkozunk a store változásaira
  eventsStore.subscribe(value => {
    events = value;
    localStorage.setItem('events', JSON.stringify(value));
  });

  let showEventForm = false;
  let selectedEvent = null;
  let selectedDate = null;

  onMount(() => {
    const savedEvents = localStorage.getItem('events');
    if (savedEvents) {
      eventsStore.set(JSON.parse(savedEvents));
    }
  });

  function handleDateClick(event) {
    selectedDate = event.detail;
    showEventForm = true;
    selectedEvent = null;
  }

  function handleEventClick(event) {
    selectedEvent = event.detail;
    showEventForm = true;
  }

  function handleSaveEvent(event) {
    const eventData = event.detail;
    let newEvents;
    
    if (selectedEvent) {
      // Esemény szerkesztése
      newEvents = events.map(e => 
        e.id === selectedEvent.id ? { ...eventData, id: selectedEvent.id } : e
      );
    } else {
      // Új esemény hozzáadása
      newEvents = [...events, { ...eventData, id: Date.now() }];
    }
    
    eventsStore.set(newEvents);
    showEventForm = false;
    selectedEvent = null;
  }

  function handleDeleteEvent() {
    if (selectedEvent) {
      const newEvents = events.filter(e => e.id !== selectedEvent.id);
      eventsStore.set(newEvents);
    }
    showEventForm = false;
    selectedEvent = null;
  }

  function handleToggleComplete(event) {
    const eventToToggle = event.detail;
    const newEvents = events.map(e => 
      e.id === eventToToggle.id ? { ...e, completed: !e.completed } : e
    );
    eventsStore.set(newEvents);
  }

  function handleCancelEvent() {
    showEventForm = false;
    selectedEvent = null;
  }
</script>

<main>
  <div class="calendar-container">
    <Calendar 
      {events}
      on:dateClick={handleDateClick}
      on:eventClick={handleEventClick}
      on:toggleComplete={handleToggleComplete}
    />
  </div>

  {#if showEventForm}
    <div class="modal">
      <div class="modal-content">
        <EventForm
          event={selectedEvent}
          on:save={handleSaveEvent}
          on:cancel={handleCancelEvent}
        />
        {#if selectedEvent}
          <button class="delete-button" on:click={handleDeleteEvent}>
            Esemény törlése
          </button>
        {/if}
      </div>
    </div>
  {/if}
</main>

<style>
  main {
    width: 100%;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: flex-start;
    padding: 20px;
    background-color: #f5f5f5;
  }

  .calendar-container {
    width: 100%;
    max-width: 1200px;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 20px;
  }

  .modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .modal-content {
    background: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    position: relative;
    width: 90%;
    max-width: 500px;
  }

  .delete-button {
    width: 100%;
    padding: 8px;
    margin-top: 10px;
    background-color: #ff4444;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.2s;
  }

  .delete-button:hover {
    background-color: #cc0000;
  }

  @media (max-width: 600px) {
    .modal-content {
      width: 95%;
      margin: 10px;
    }
  }
</style> 