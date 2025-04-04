<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let event = null;

  let title = event?.title || '';
  let date = event?.date || new Date().toISOString().split('T')[0];
  let startTime = event?.startTime || '09:00';
  let endTime = event?.endTime || '10:00';
  let color = event?.color || '#4CAF50';
  let completed = event?.completed || false;

  $: if (event) {
    title = event.title;
    date = event.date;
    startTime = event.startTime;
    endTime = event.endTime;
    color = event.color;
    completed = event.completed;
  }

  function handleSubmit() {
    if (!isValidTimeRange(startTime, endTime)) {
      alert('A végidőpontnak későbbinek kell lennie, mint a kezdő időpontnak!');
      return;
    }

    const eventData = {
      title,
      date,
      startTime,
      endTime,
      color,
      completed
    };

    if (event) {
      eventData.id = event.id;
    }

    dispatch('save', eventData);
  }

  function handleCancel() {
    dispatch('cancel');
  }

  function isValidTimeRange(start, end) {
    const [startHours, startMinutes] = start.split(':').map(Number);
    const [endHours, endMinutes] = end.split(':').map(Number);
    
    const startTotal = startHours * 60 + startMinutes;
    const endTotal = endHours * 60 + endMinutes;
    
    return endTotal > startTotal;
  }
</script>

<div class="form-container">
  <h2>{event ? 'Esemény szerkesztése' : 'Új esemény'}</h2>
  
  <form on:submit|preventDefault={handleSubmit}>
    <div class="form-group">
      <label for="title">Esemény neve:</label>
      <input
        type="text"
        id="title"
        bind:value={title}
        required
        placeholder="Add meg az esemény nevét..."
      />
    </div>

    <div class="form-group">
      <label for="date">Dátum:</label>
      <input
        type="date"
        id="date"
        bind:value={date}
        required
      />
    </div>

    <div class="form-group">
      <label for="startTime">Kezdeti időpont:</label>
      <input
        type="time"
        id="startTime"
        bind:value={startTime}
        required
      />
    </div>

    <div class="form-group">
      <label for="endTime">Végidőpont:</label>
      <input
        type="time"
        id="endTime"
        bind:value={endTime}
        required
      />
    </div>

    <div class="form-group">
      <label for="color">Szín:</label>
      <input
        type="color"
        id="color"
        bind:value={color}
      />
    </div>

    {#if event}
      <div class="form-group checkbox">
        <label>
          <input
            type="checkbox"
            bind:checked={completed}
          />
          Esemény befejezve
        </label>
      </div>
    {/if}

    <div class="form-actions">
      <button type="button" class="cancel" on:click={handleCancel}>Mégse</button>
      <button type="submit" class="submit">{event ? 'Mentés' : 'Hozzáadás'}</button>
    </div>
  </form>
</div>

<style>
  .form-container {
    background: white;
    padding: 20px;
    border-radius: 8px;
    width: 100%;
    max-width: 400px;
  }

  h2 {
    margin-top: 0;
    margin-bottom: 20px;
    color: #333;
    text-align: center;
  }

  .form-group {
    margin-bottom: 15px;
  }

  .form-group.checkbox {
    display: flex;
    align-items: center;
  }

  label {
    display: block;
    margin-bottom: 5px;
    color: #666;
  }

  .checkbox label {
    margin-bottom: 0;
    display: flex;
    align-items: center;
    cursor: pointer;
  }

  input {
    width: 100%;
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 16px;
  }

  input[type="color"] {
    height: 40px;
    padding: 2px;
  }

  input[type="checkbox"] {
    width: auto;
    margin-right: 8px;
  }

  .form-actions {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    margin-top: 20px;
  }

  button {
    padding: 8px 16px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.2s;
  }

  button.cancel {
    background-color: #f5f5f5;
    color: #333;
  }

  button.submit {
    background-color: #4CAF50;
    color: white;
  }

  button:hover {
    opacity: 0.9;
  }

  input:focus {
    outline: none;
    border-color: #4CAF50;
    box-shadow: 0 0 0 2px rgba(74, 175, 80, 0.2);
  }
</style> 