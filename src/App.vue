<script setup>
import { ref } from 'vue';

// Initial data
const suspects = ref(['Miss Scarlett', 'Colonel Mustard', 'Mrs. White', 'Mr. Green', 'Mrs. Peacock', 'Professor Plum']);
const weapons = ref(['Wrench', 'Candlestick', 'Lead Pipe', 'Knife', 'Rope']);
const rooms = ref(['Ballroom', 'Billiard Room', 'Conservatory', 'Dining Room', 'Hall', 'Kitchen', 'Library', 'Lounge', 'Secret Passage', 'Study']);
const removedRooms = ref([]); // Store removed rooms
const culprit = ref({ suspect: '', weapon: '', room: '' }); // Store the culprit details
const showCulpritPopup = ref(false); // Control popup visibility

// Function to shuffle an array
const shuffle = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
};

// Remove 2 random rooms and save them to removedRooms
const removeRooms = () => {
  // Generate two unique random indices
  const index1 = Math.floor(Math.random() * rooms.value.length);
  let index2;
  do {
    index2 = Math.floor(Math.random() * rooms.value.length);
  } while (index2 === index1); // Ensure the second index is different from the first

  // Remove the rooms at the random indices and save them to removedRooms
  removedRooms.value = [rooms.value[index1], rooms.value[index2]];

  // Remove the rooms from the rooms array
  rooms.value.splice(Math.max(index1, index2), 1); // Remove the higher index first to avoid shifting issues
  rooms.value.splice(Math.min(index1, index2), 1);
};

// Remove a random suspect, weapon, and room and store them in culprit
const setCulprit = () => {
  const randomSuspectIndex = Math.floor(Math.random() * suspects.value.length);
  const randomWeaponIndex = Math.floor(Math.random() * weapons.value.length);
  const randomRoomIndex = Math.floor(Math.random() * rooms.value.length);

  culprit.value = {
    suspect: suspects.value[randomSuspectIndex],
    weapon: weapons.value[randomWeaponIndex],
    room: rooms.value[randomRoomIndex],
  };

  // Remove the selected items from their respective arrays
  suspects.value.splice(randomSuspectIndex, 1);
  weapons.value.splice(randomWeaponIndex, 1);
  rooms.value.splice(randomRoomIndex, 1);
};

// Shuffle all items and distribute into 4 piles
const distributeItems = () => {
  const allItems = [...suspects.value, ...weapons.value, ...rooms.value];
  const shuffledItems = shuffle(allItems);
  const pileSize = Math.ceil(shuffledItems.length / 4);
  const piles = [];

  for (let i = 0; i < 4; i++) {
    piles.push(shuffledItems.slice(i * pileSize, (i + 1) * pileSize));
  }

  return piles;
};

// Initialize the piles
const piles = ref(distributeItems());

// Track visibility of each pile
const pileVisibility = ref(piles.value.map(() => false)); // Initially all piles are hidden

// Toggle visibility for a pile
const togglePileVisibility = (index) => {
  pileVisibility.value[index] = !pileVisibility.value[index];
};

// Remove rooms, set culprit, and redistribute items
removeRooms();
setCulprit();
piles.value = distributeItems();

// Toggle culprit popup visibility
const toggleCulpritPopup = () => {
  showCulpritPopup.value = !showCulpritPopup.value;
};
</script>

<template>
  <header>
    <h1>Clue Game</h1>
  </header>

  <main>
    <div class="grid-container">


      <section>
        <h2>Removed Rooms</h2>
        <ul>
          <li v-for="room in removedRooms" :key="room">{{ room }}</li>
        </ul>
      </section>

      <section class="piles-section">
        <h2>Piles</h2>
        <div class="piles-grid">
          <div v-for="(pile, index) in piles" :key="index" class="pile">
            <h3>Pile {{ index + 1 }}</h3>
            <button @click="togglePileVisibility(index)">
              {{ pileVisibility[index] ? 'Hide' : 'Show' }}
            </button>
            <ul v-if="pileVisibility[index]">
              <li v-for="item in pile" :key="item">{{ item }}</li>
            </ul>
          </div>
        </div>
      </section>
    </div>


    <!-- Culprit Section -->
    <section>
      <h2>Culprit</h2>
      <div v-if="showCulpritPopup" class="popup-overlay">
        <ul class="popup-content">
          <li><strong>Suspect:</strong> {{ culprit.suspect }}</li>
          <li><strong>Weapon:</strong> {{ culprit.weapon }}</li>
          <li><strong>Room:</strong> {{ culprit.room }}</li>
          <button @click="toggleCulpritPopup">Close</button>
        </ul>
      </div>

      <button v-if="!showCulpritPopup" @click="toggleCulpritPopup">Show Culprit</button>
    </section>

    <!-- Culprit Popup -->

  </main>
</template>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f4f4f4;
  color: #333; /* Darker text color for better visibility */
}

header {
  background-color: #333;
  color: #fff;
  padding: 1rem;
  text-align: center;
}

main {
  padding: 1rem;
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

section {
  background-color: #fff;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h2 {
  color: #333;
  margin-bottom: 1rem;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

li {
  background-color: #f9f9f9;
  margin: 0.5rem 0;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.piles-section {
  grid-column: 1 / -1; /* Make the piles section span all columns */
}

.piles-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.pile {
  background-color: #f9f9f9;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

@media (max-width: 768px) {
  .grid-container {
    grid-template-columns: 1fr;
  }

  .piles-grid {
    grid-template-columns: 1fr;
  }
}
</style>
