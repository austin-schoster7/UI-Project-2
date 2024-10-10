<script>
  import { onMount } from "svelte";
  import Modal from './Modal.svelte';

  let toastLevel = 3; // Default toast level
  let toasting = false;
  let toastCount = 0;
  let remainingTime = 0;
  let selectedItem = 'Bread';
  let countdownInterval;
  let progress = 0; // Progress bar starts at 0%
  let elapsedTime = 0; // Track elapsed time
  let targetLevel = toastLevel;

  // Item options for the toaster
  const items = ['Bread', 'Bagel', 'Waffle', 'Pastry'];

  // Mock user profiles with favorite and previous settings
  const profiles = {
    'Alice': {
      favorite: { item: 'Bread', toastLevel: 4 },
      previous: { item: 'Bagel', toastLevel: 3 }
    },
    'Bob': {
      favorite: { item: 'Waffle', toastLevel: 5 },
      previous: { item: 'Pastry', toastLevel: 2 }
    }
  };

  let selectedUser = ''; // No user selected by default

  // Modal control variables
  let showModal = false;

  // Mapping item names to their images for each toast level (1-6)
  const itemImages = {
    Bread: {
      1: 'bread.jpg',
      2: 'bread-slightly-toasted.png',
      3: 'bread-moderately-toasted.png',
      4: 'bread-well-toasted.png',
      5: 'bread-very-well-toasted.png',
      6: 'bread-burnt.png'
    },
    Bagel: {
      1: 'bagel.jpg',
      2: 'bagel-slightly-toasted.png',
      3: 'bagel-moderately-toasted.png',
      4: 'bagel-well-toasted.png',
      5: 'bagel-very-well-toasted.png',
      6: 'bagel-burnt.png'
    },
    Waffle: {
      1: 'waffle-1.png',
      2: 'waffle-2.png',
      3: 'waffle-3.png',
      4: 'waffle-4.png',
      5: 'waffle-5.png',
      6: 'waffle-6.png'
    },
    Pastry: {
      1: 'pastry-1.png',
      2: 'pastry-2.png',
      3: 'pastry-3.png',
      4: 'pastry-4.png',
      5: 'pastry-5.png',
      6: 'pastry-6.png'
    }
  };

  // Function to start toasting and begin countdown
  const startToasting = () => {
    if (!toasting) {
      toasting = true;
      remainingTime = toastLevel * 30; // Simulated time based on toast level
      const totalTime = remainingTime;
      elapsedTime = 0; // Reset elapsed time
      targetLevel = toastLevel;
      toastLevel = 1;

      countdownInterval = setInterval(() => {
        if (elapsedTime < totalTime) {
          elapsedTime++;
          progress = (elapsedTime / totalTime) * 100; // Update progress
        } else {
          stopToasting(); // Stop once elapsedTime reaches totalTime
        }
        if (targetLevel != toastLevel && elapsedTime % 30 === 0) {
          toastLevel++;
        }
      }, 1000); // Increment every second
    }
  };

  // Function to stop toasting and clear countdown
  const stopToasting = () => {
    if (toasting) {
      toasting = false;
      clearInterval(countdownInterval);
      progress = 0; // Reset progress bar
      remainingTime = 0;
      elapsedTime = 0;
      toastCount++;
    }
  };

  // Load profile settings based on user selection
  const loadProfile = () => {
    if (selectedUser) {
      showModal = true; // Show the modal
    }
  };

  const closeModal = () => {
    showModal = false;
  };

  const loadFavoriteSettings = () => {
    const favorite = profiles[selectedUser].favorite;
    selectedItem = favorite.item;
    toastLevel = favorite.toastLevel;
    showModal = false;
  };

  const loadPreviousSettings = () => {
    const previous = profiles[selectedUser].previous;
    selectedItem = previous.item;
    toastLevel = previous.toastLevel;
    showModal = false;
  };

  const loadBlankSettings = () => {
    selectedItem = 'Bread';
    toastLevel = 3;
    showModal = false;
  };

  // Reactive block to reset remaining time, progress, etc.
  $: {
    if (!toasting) {
      remainingTime = toastLevel * 30; // Simulated time based on level
      progress = 0; // Reset progress when not toasting
      elapsedTime = 0; // Reset elapsed time
    }
  }
</script>

<div class="page">
  <!-- Device UI -->
  <div class="device-ui">
    <h2>Smart Toaster</h2>
    <h3>Austin Schoster, Skyler Simpson, Joe Schnizer, Sam Winkelmann</h3>
    <a href="https://sites.google.com/view/austin-schoster/cs5167-projects">Link to write up</a>
    
    <br><br>

    <!-- User profile selection -->
    <div class="profile-section">
      <label>Select User: </label>
      <select bind:value={selectedUser} on:change={loadProfile}>
        <option value="">Select a user</option>
        {#each Object.keys(profiles) as user}
          <option value={user}>{user}</option>
        {/each}
      </select>
    </div>

    <!-- Item selection dropdown -->
    <div>
      <label>Select Item to Toast: </label>
      <select bind:value={selectedItem}>
        {#each items as item}
          <option value={item}>{item}</option>
        {/each}
      </select>
    </div>
    
    <!-- Display area for selected item outline -->
    <div class="display-section">
      <h3>Toasting: {selectedItem}</h3>
      <img src={itemImages[selectedItem][toastLevel]} alt="{selectedItem} at toast level {toastLevel}" class="item-outline" />
    </div>

    <!-- Toast level slider -->
    <div class="toast-level">
      <label for="toast-level-slider">Toast Level: {toastLevel}</label>
      <input
        id="toast-level-slider"
        type="range"
        min="1"
        max="6"
        bind:value={toastLevel}
        step="1"
        style="--value: {(toastLevel - 1) / 5}"
      />
    </div>

    <!-- Toasting controls -->
    <div>
      <p>Time Remaining: {remainingTime - elapsedTime} seconds</p>
    </div>
    <div>
      <button on:click={startToasting}>Start Toasting</button>
      <button on:click={stopToasting}>Stop Toasting</button>
    </div>

    <!-- Progress Bar -->
    <div class="progress-bar-container">
      <div class="progress-bar" style="width: {progress}%"></div>
    </div>

    <p>Toasts made today: {toastCount}</p>
  </div>
  
  <!-- Testing UI -->
  <div class="testing-ui">
    <h2>Testing Controls</h2>
    <button on:click={startToasting}>Simulate Start</button>
    <button on:click={stopToasting}>Simulate Stop</button>
    <button on:click={() => alert('Info about controls...')}>Info</button>
  </div>

  <div class="toaster-diagram">
    <h2>Toaster Mockup</h2>
    <img src="toaster.png" alt="Toaster" />
  </div>
</div>

<!-- Modal for Profile Settings -->
<Modal bind:show={showModal} title="Load Settings for {selectedUser}" onClose={closeModal}>
  <p>Select settings to load:</p>
  <div class="modal-buttons">
    <button on:click={loadFavoriteSettings}>Favorite</button>
    <button on:click={loadPreviousSettings}>Previous</button>
    <button on:click={loadBlankSettings}>Blank</button>
  </div>
</Modal>

<style>
  /* General Page Layout */
  .page {
    display: flex;
    justify-content: space-around;
    background-color: #f0f0f5; /* Light background color for a modern look */
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    padding: 20px;
    height: 100vh;
  }

  /* Device UI Container */
  .device-ui {
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    padding: 20px;
    width: 50%;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  /* Testing UI Container */
  .testing-ui {
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    padding: 20px;
    width: 25%;
    text-align: center;
  }

  /* Headers */
  h2 {
    color: #333;
    font-size: 24px;
    font-weight: 600;
    margin-bottom: 20px;
  }

  h3 {
    color: #444;
    font-size: 20px;
    font-weight: 500;
  }

  /* Item Selection Dropdown */
  select {
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #f9f9f9;
    font-size: 16px;
    width: 100%;
    margin-bottom: 20px;
    transition: border 0.3s;
  }

  select:hover {
    border-color: #007bff;
  }

  /* Toasting Item Image */
  .display-section {
    margin-top: 20px;
    text-align: center;
    background-color: #f9f9f9;
    padding: 15px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  }

  .item-outline {
    width: 120px;
    height: auto;
    margin-top: 10px;
    border-radius: 8px;
  }

  /* Toast Level Slider */
  .toast-level {
    margin-top: 20px;
    width: 100%;
    text-align: left;
  }

  input[type="range"] {
    width: 100%;
    margin-top: 5px;
    appearance: none;
    height: 8px;
    background: linear-gradient(
      to right,
      #007bff 0%,
      #007bff calc(var(--value) * 100%),
      #ddd calc(var(--value) * 100%),
      #ddd 100%
    );
    border-radius: 5px;
    outline: none;
  }

  input[type="range"]::-webkit-slider-thumb {
    appearance: none;
    width: 20px;
    height: 20px;
    background-color: #007bff;
    border-radius: 50%;
    cursor: pointer;
  }

  input[type="range"]::-moz-range-thumb {
    width: 20px;
    height: 20px;
    background-color: #007bff;
    border-radius: 50%;
    cursor: pointer;
  }

  /* Progress Bar */
  .progress-bar-container {
    width: 100%;
    height: 20px;
    background-color: #e0e0e0;
    border-radius: 10px;
    overflow: hidden;
    margin-top: 20px;
    position: relative;
  }

  .progress-bar {
    height: 100%;
    background-color: #28a745;
    transition: width 1s linear;
    border-radius: 10px;
  }

  /* Buttons */
  button {
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 8px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s, box-shadow 0.3s;
  }

  button:hover {
    background-color: #0056b3;
    box-shadow: 0 4px 8px rgba(0, 123, 255, 0.2);
  }

  button:active {
    background-color: #004494;
  }

  /* Responsive Layout */
  @media (max-width: 768px) {
    .page {
      flex-direction: column;
    }

    .device-ui, .testing-ui {
      width: 100%;
      margin-bottom: 20px;
    }
  }
</style>

