<script>
	import { onMount } from 'svelte';
	import Modal from './Modal.svelte';

	let toastLevel = 3; // Default toast level
  let toastDecimal = 0;
	let toasting = false;
	let toastCount = 0;
	let remainingTime = 0;
	let selectedItem = 'Bread';
	let countdownInterval;
	let progress = 0; // Progress bar starts at 0%
	let elapsedTime = 0; // Track elapsed time
	let targetLevel = toastLevel;
	let reheatTimer = 0;
	let reheatActive = false;
	let reheatModeTime = 0;
	let toastRemoved = false;
	let showInfoModal = false;
	let showModal = false;
	let temperature = 0;
	let calories = 0;
	let toastLevelArray = [];
	let avgToastLevel = 0;

	// Item options for the toaster
	const items = ['Bread', 'Bagel', 'Waffle', 'Pastry'];

	// Mock user profiles with favorite and previous settings
	const profiles = {
		Alice: {
			favorite: { item: 'Bread', toastLevel: 4 },
			previous: { item: 'Bagel', toastLevel: 3 }
		},
		Bob: {
			favorite: { item: 'Waffle', toastLevel: 5 },
			previous: { item: 'Pastry', toastLevel: 2 }
		}
	};

	let selectedUser = ''; // No user selected by default

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
			1: 'waffle.png',
			2: 'waffle-slightly-toasted.png',
			3: 'waffle-moderately-toasted.png',
			4: 'waffle-well-toasted.png',
			5: 'waffle-very-well-toasted.png',
			6: 'waffle-burnt.png'
		},
		Pastry: {
			1: 'pastry.png',
			2: 'pastry-slightly-toasted.png',
			3: 'pastry-moderately-toasted.png',
			4: 'pastry-well-toasted.png',
			5: 'pastry-very-well-toasted.png',
			6: 'pastry-burnt.png'
		}
	};

  let increaseOpacityCounter = 0;
  let imageOpacity = 0;
	// Function to start toasting and begin countdown
	const startStopToasting = () => {
    document.getElementById("top-image").style.opacity = 0;
		if (!toasting) {
			toasting = !toasting;
			remainingTime = toastLevel * 30; // Simulated time based on toast level
			const totalTime = remainingTime;
			elapsedTime = 0; // Reset elapsed time
			targetLevel = toastLevel;
			toastLevel = 1;

			// Update average toast level
			toastLevelArray.push(targetLevel);
			let total = 0;
			toastLevelArray.forEach((level) => {
				total += level;
				avgToastLevel = total / toastLevelArray.length;
			});

			// Update calories based on selected item
			if (selectedItem === 'Bread') {
				calories += 80;
			} else if (selectedItem === 'Bagel') {
				calories += 245;
			} else if (selectedItem === 'Waffle') {
				calories += 218;
			} else if (selectedItem === 'Pastry') {
				calories += 300;
			}

			countdownInterval = setInterval(() => {
				if (elapsedTime < totalTime) {
					elapsedTime++;
					progress = (elapsedTime / totalTime) * 100; // Update progress
					// Update temperature based on progress
					temperature = updateTemperature(progress / 100);
				} else {
					stopToasting(); // Stop once elapsedTime reaches totalTime
					startReheatCountdown(); // Start the reheat countdown when toasting stops
				}
				if (targetLevel != toastLevel && elapsedTime % 30 === 0) {
					toastLevel++;
				}

        increaseOpacityCounter++;
        if(totalTime/increaseOpacityCounter == 10){
          imageOpacity += .066;
          document.getElementById("top-image").style.opacity = imageOpacity;
          increaseOpacityCounter = 0;
        }
			}, 1000); // Increment every second
		} else {
			stopToasting();
		}
	};

	// Function to simulate temperature
	function updateTemperature(progress) {
		// Assume temperature rises from 100 to 250 degrees as toast progresses
		let temp = 100 + 150 * progress;
		return temp > 250 ? 250 : temp; // Cap temperature at 250 degrees
	}

	// Function to stop toasting and clear countdown
	const stopToasting = () => {
		if (toasting) {
			toasting = !toasting;
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

	// Function to show info modal
	const showInfo = () => {
		showInfoModal = true; // Trigger info modal
	};

	// Close the info modal
	const closeInfoModal = () => {
		showInfoModal = false;
	};

	const closeModal = () => {
		showModal = false;
		showInfoModal = false;
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

	const startReheatCountdown = () => {
		toastRemoved = false;
		reheatTimer = 30; // 30 seconds to remove toast
		const reheatInterval = setInterval(() => {
			if (reheatTimer > 0) {
				reheatTimer--;
			} else {
				clearInterval(reheatInterval);
				if (!toastRemoved) {
					// Only activate reheat mode if toast was not removed
					activateReheatMode();
				}
			}
		}, 1000);
	};

	const removeToast = () => {
		reheatTimer = 0; // Cancel reheat countdown
		reheatActive = false; // Disable reheat mode if active
		toastRemoved = true;
		progress = 0; // Reset progress when toast is removed
    document.getElementById("top-image").src = "transparent.png";
	};

	const activateReheatMode = () => {
		reheatActive = true;
		progress = 100; // Indicate the toast is fully done in reheat mode
	};

  // Reactive block to reset remaining time, progress, etc.
  $: {
    if (!toasting) {
      remainingTime = toastLevel * 30; // Simulated time based on level
      progress = 0; // Reset progress when not toasting
      elapsedTime = 0; // Reset elapsed time
    }
  }

  const updateToastLevel = (event) =>{
    toastDecimal = event.target.value - Math.floor(event.target.value);
    toastLevel = Math.floor(event.target.value);
  }

  const colorA = [
    [255, 255, 255], //White
    [239, 228, 176], //slightly
    [242, 160, 114], //moderately
    [185, 122, 87], //well
    [79, 52, 38], //very well
    [0, 0, 0] //burnt
  ]
  let mixedColor = "#B97A57";
  $: {
    if (toastLevel == 1 && toastDecimal == 0){
      mixedColor = colorA[toastLevel-1];
    }
    else if (toastLevel == 6){
      mixedColor = colorA[toastLevel-1];
    }
    else{
      let mixedR = colorA[toastLevel-1][0] + toastDecimal*(colorA[toastLevel][0]-colorA[toastLevel-1][0]);
      let mixedG = colorA[toastLevel-1][1] + toastDecimal*(colorA[toastLevel][1]-colorA[toastLevel-1][1]);
      let mixedB = colorA[toastLevel-1][2] + toastDecimal*(colorA[toastLevel][2]-colorA[toastLevel-1][2]);

      mixedColor = "#" + (1 << 24 | mixedR << 16 | mixedG << 8 | mixedB).toString(16).slice(1);
    }
  }

  let imageUrl = '';
  const handleFileChange = (event) => {
      imageUrl = URL.createObjectURL(event.target.files[0]);
      document.getElementById("top-image").src = imageUrl;
  };

  let imageUploadButton;
  const triggerFileInput = () => {
      imageUploadButton.click();
    };
</script>

<div class="page">
	<!-- Device UI -->
	<div class="device-ui">
		<div class="header-section">
			<h2>Smart Toaster</h2>
			<h3>Austin Schoster, Skyler Simpson, Joe Schnizer, Sam Winkelmann</h3>
			<a href="https://sites.google.com/view/austin-schoster/cs5167-projects">Link to write up</a>
		</div>

		<br /><br />

		<div class="toaster-controls">
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
				<button
					class={!toasting ? 'startButton' : 'stopButton'}
					on:click={startStopToasting}
					disabled={reheatTimer > 0 || reheatActive}
				>
					<img
						src={itemImages[selectedItem][toastLevel]}
						alt="{selectedItem} at toast level {toastLevel}"
						class="item-outline"
					/><br />
					{!toasting ? 'Start Toasting' : 'Stop Toasting'}
				</button>
			</div>
		</div>

    <!-- Toast level slider -->
    <div class="toast-level">
      <label for="toast-level-slider">Toast Level: {toastLevel}</label>
      <input
        id="toast-level-slider"
        type="range"
        min="1"
        max="6"
        on:input={updateToastLevel}
        value={toastLevel}
        step=".01"
        style="--value: {(Math.floor(toastLevel) - 1) / 5};"
      />
    </div>

		<!-- Toasting controls -->
		<div>
			<p>Time Remaining: {remainingTime - elapsedTime} seconds</p>
		</div>
		<div></div>

    {#if selectedItem === "Bread"}
      <input type="file" bind:this={imageUploadButton} on:change={handleFileChange} accept="image/*" style="display: none;"/>
      <button on:click={triggerFileInput}>Upload Image</button>
    {/if}

		<!-- Progress Bar -->
		<div class="progress-bar-container">
			<div
				class="progress-bar {reheatActive ? 'reheat-mode' : ''}"
				style="width: {progress}%"
			></div>
		</div>

		<!-- Reheat Mode and Toast Removal -->
		{#if reheatTimer > 0}
			<p>Reheat mode starting in {reheatTimer} seconds</p>
			<button on:click={removeToast}>Remove Toast</button>
		{/if}

		{#if reheatActive}
			<p>Reheat Mode Active. Toast will stay warm until removed.</p>
			<button on:click={removeToast}>Remove Toast</button>
		{/if}

		<!-- Statistics -->
		<div class="stats">
			<h3>Statistics</h3>
			<p>Toasts made today: {toastCount}</p>
			<p>Temperature: {Math.round(temperature)} °F</p>
			<p>Elapsed Time: {Math.round(elapsedTime)} s</p>
			<p>Total calories: {calories}</p>
			<p>Average Toast Level: {avgToastLevel.toFixed(2)}</p>
		</div>
	</div>

	<!-- Testing UI -->
	<div class="testing-ui">
		<h2>Testing Controls</h2>
		<button on:click={startStopToasting}>Simulate Start</button>
		<button on:click={stopToasting}>Simulate Stop</button>
		<button on:click={showInfo}>Info</button>
	</div>

	<div class="toaster-diagram">
		<h2>Toaster Mockup</h2>
		<img src="toaster.png" alt="Toaster" />
	</div>
</div>

<!-- Modal for Profile Settings -->
<Modal
	bind:show={showModal}
	bind:showInfo={showInfoModal}
	title="Load Settings for {selectedUser}"
	onClose={closeModal}
>
	{#if showModal}
		<p>Select settings to load:</p>
		<div class="modal-buttons">
			<button on:click={loadFavoriteSettings}>Favorite</button>
			<button on:click={loadPreviousSettings}>Previous</button>
			<button on:click={loadBlankSettings}>Blank</button>
		</div>
	{:else if showInfoModal}
		<p>
			All the controls for simulating the smart toaster are included in the UI design rather than
			being incorporated in a separate testing interface. The toast icon acts as a button and
			initiates the toasting process. It also stops toasting after a second press. The "Simulate
			Start" and "Simulate Stop" buttons also perform the same functions outside the toaster's
			physical interface. The "Info" button provides additional information about the controls and
			their functionalities. The user can select a profile to load settings for the toaster,
			including the item to toast and the desired toast level. The toaster's progress is displayed
			through a progress bar, and the user can remove the toast or activate reheat mode when the
			toasting process is complete. The UI also includes a toaster mockup for visual reference.
		</p>
		<div class="modal-buttons">
			<button on:click={closeModal}>Close</button>
		</div>
	{/if}
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
    height: auto;
    margin-top: 10px;
    border-radius: 8px;
    position: absolute;
  }
  
  #image-stack{
    position: relative;
  }

  #bottom-image{
    width: 120px;
    z-index: 0;
    top: 0;
    left: 0;
  }

  #top-image{
    width: 60px;
    z-index: 10;
    opacity: 0;
    top: 30px;
    left: 25%;
    filter: grayscale(1);
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
    background-image: linear-gradient(
      to right,
      white, 
      #EFE4B0, 
      #F2A072, 
      #B97A57, 
      #4F3426, 
      black
    );
    border: 2px solid #007bff; 
    border-radius: 5px;
    outline: none;
  }

  input[type="range"]::-webkit-slider-thumb {
    appearance: none;
    width: 10px;
    height: 10px;
    background-color: transparent;
    padding: 4px;
    border: 4px solid #007bff;
    border-radius: 50%;
    cursor: pointer;
  }

	input[type='range']::-moz-range-thumb {
		width: 10px;
		height: 10px;
		background-color: transparent;
    padding: 4px;
    border: 4px solid #007bff;
		border-radius: 50%;
		cursor: pointer;
	}

	button:disabled {
		background-color: grey;
		cursor: not-allowed;
		opacity: 0.6;
	}

	/* Progress Bar */
	.progress-bar-container {
		width: 100%;
		max-width: 100%;
		height: 20px;
		background-color: #e0e0e0;
		border-radius: 10px;
		overflow: hidden;
		margin-top: 20px;
		position: relative;
		box-sizing: border-box;
	}

	.progress-bar {
		height: 100%;
		background-color: #28a745;
		transition: width 1s linear;
		width: 0%; /* Initialize with no progress */
		border-radius: 10px;
		min-width: 1px; /* Ensure the progress bar is always visible even if it's very small */
	}

	/* Header Section Styling */
	.header-section {
		background-color: #008080; /* Teal background */
		border-radius: 12px; /* Rounded edges */
		padding: 20px; /* Spacing inside the box */
		text-align: center; /* Center-align the text */
		margin-bottom: 20px; /* Add space below the box */
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1); /* Slight shadow for depth */
	}

	/* Headers inside the header section */
	.header-section h2,
	.header-section h3 {
		color: white; /* Set title and names color to white for contrast */
		margin-top: 7px;
	}

	/* Styling for the link inside the header section */
	.header-section a {
		color: #f0f0f5; /* Light color for the link */
		text-decoration: underline; /* Underline the link */
	}

	.header-section a:hover {
		color: #0c22c7; /* Lighter teal when hovering over the link */
	}

	/* Flashing red animation for reheat mode */
	@keyframes flashRed {
		0% {
			background-color: #ff4d4d; /* Light red */
		}
		50% {
			background-color: #ff9999; /* Lighter red */
		}
		100% {
			background-color: #ff4d4d; /* Light red */
		}
	}

	/* Apply the flashing red animation in reheat mode */
	.reheat-mode {
		animation: flashRed 1s infinite;
		min-width: 1px; /* Ensure the bar is always visible in reheat mode */
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
		transition:
			background-color 0.3s,
			box-shadow 0.3s;
	}

	button:hover {
		background-color: #0056b3;
		box-shadow: 0 4px 8px rgba(0, 123, 255, 0.2);
	}

	button:active {
		background-color: #004494;
	}

	.startButton {
		background-color: rgb(0, 181, 0);
		color: white;
		border: none;
		border-radius: 8px;
		padding: 10px 20px;
		font-size: 16px;
		cursor: pointer;
		transition:
			background-color 0.3s,
			box-shadow 0.3s;
	}

	.startButton:hover {
		background-color: rgb(31, 107, 31);
		box-shadow: 0 4px 8px rgba(0, 123, 255, 0.2);
	}

	.stopButton {
		background-color: red;
		color: white;
		border: none;
		border-radius: 8px;
		padding: 10px 20px;
		font-size: 16px;
		cursor: pointer;
		transition:
			background-color 0.3s,
			box-shadow 0.3s;
	}

	.stopButton:hover {
		background-color: rgb(183, 0, 0);
		box-shadow: 0 4px 8px rgba(0, 123, 255, 0.2);
	}

	/* New styles for toaster controls */
	.toaster-controls {
		background-color: #2450a1; /* Dark blue background */
		padding: 20px;
		border-radius: 8px;
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 25%;
		margin-bottom: 20px;
	}

	.toaster-controls label {
		color: white; /* Make labels visible on dark background */
	}

	.toaster-controls select,
	.toaster-controls button {
		margin-top: 10px;
	}

	/* Responsive Layout */
	@media (max-width: 768px) {
		.page {
			flex-direction: column;
		}

		.device-ui,
		.testing-ui {
			width: 100%;
			margin-bottom: 20px;
		}
	}
</style>
