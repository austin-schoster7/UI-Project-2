<script>
  let toastLevel = 3; // Default toast level
  let toasting = false;
  let toastCount = 0;
  let remainingTime = 0;
  let selectedItem = 'Bread';

  // Item options for the toaster
  const items = ['Bread', 'Bagel', 'Waffle', 'Pastry'];

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
      1: 'bagel-1.png',
      2: 'bagel-2.png',
      3: 'bagel-3.png',
      4: 'bagel-4.png',
      5: 'bagel-5.png',
      6: 'bagel-6.png'
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

  const startToasting = () => {
    toasting = true;
    remainingTime = toastLevel * 30; // Simulated time based on level
  };

  const stopToasting = () => {
    toasting = false;
    remainingTime = 0;
    toastCount++;
  };
</script>

<div class="page">
  <!-- Device UI -->
  <div class="device-ui">
    <h2>Smart Toaster</h2>
    
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
      />
    </div>

    <!-- Toasting controls -->
    <div>
      <p>Time Remaining: {remainingTime} seconds</p>
    </div>
    <div>
      <button on:click={startToasting}>Start Toasting</button>
      <button on:click={stopToasting}>Stop Toasting</button>
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

<style>
  .page {
    display: flex;
    justify-content: space-around;
  }

  .device-ui, .testing-ui {
    border: 1px solid #ccc;
    padding: 20px;
  }

  .device-ui {
    width: 50%;
  }

  img {
    width: 150px;
  }

  .display-section {
    margin-top: 20px;
    text-align: center;
  }

  .item-outline {
    width: 100px;
    height: auto;
  }

  .toast-level {
    margin-top: 20px;
  }

  input[type="range"] {
    width: 100%;
  }
</style>
