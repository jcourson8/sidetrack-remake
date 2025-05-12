<template>
  <div class="min-h-screen flex flex-col font-serif pt-10 xl:pt-20 px-10 md:px-30">
    <!-- Layout container that changes direction on larger screens -->
    <div class="flex flex-col lg:flex-row flex-grow">
      <!-- New leftmost column for very large screens -->
      <div class="hidden xl:block xl:w-[250px] mr-10">
        <h1 class="font-serif">Side Track</h1>
        <h3 class="font-serif italic">Construction Website</h3>
      </div>

      <!-- Main Content Column Wrapper (Added position relative) -->
      <div class="max-w-[350px] relative">
        <!-- Source Text Container -->
        <div id="sourceTextContainer" style="opacity: 1; visibility: visible;">
          <!-- Header (Hidden on XL, part of source) -->
          <div class="mb-8 xl:hidden">
            <h1 class="font-serif ">Side Track</h1>
            <h3 class="font-serif italic">Construction Website</h3>
          </div>
          <!-- Body Content (Part of source) -->
          <div class="space-y-6">
            <p>
              <span class="italic">Side Track</span> is a coffee shop in Opelika, Alabama.
              Simply sharing coffees and things and moments.
              Open from 8:00am to 4:00pm, Tuesday through Sunday.
              Occasionally, Side Track will host gatherings for our community such as food pop-ups, art exhibitions, convivial celebrations, and live music.
            </p>
            
            <p>
              To view photography of Side Track, click 
              <a @click="toggleGallery" class="text-cyan-500 underline cursor-pointer inline">here</a>
              Sign our library card, click
              <a @click="toggleLibraryCard" class="text-cyan-500 underline cursor-pointer inline">here</a>
              View our coffee menu, click
              <a @click="animateToCoffeeMenu" class="text-cyan-500 underline cursor-pointer inline">here</a>
            </p>

            
            <p>
              Other links:<br>
              <a href="mailto:david@sidetrack.us" class="text-cyan-500 underline block inline-block mb-0">Email</a><br>
              <a href="https://www.instagram.com/sidetrackcoffee/" target="_blank" class="text-cyan-500 underline block inline-block mb-0">Instagram</a><br>
              <a href="https://open.spotify.com/user/gnz63bn18ouuzt9moevi73vyd?si=kudTaEi0SGOQjvb0P2SwGA" target="_blank" class="text-cyan-500 underline block inline-block mb-0">Music</a><br>
              <a @click="openMap" class="text-cyan-500 underline cursor-pointer block inline-block mb-0">Address</a><br>
              <a href="https://www.youtube.com/watch?v=I25U2aagIfg" class="text-cyan-500 underline block inline-block mb-0">Inspiration</a>
            </p>
            
            <p class="italic">"The objects in the space will wither like the grass and fall like the flower. These moments, however, will last forever."</p>
            
            <div class="mt-6">
              <span>Current Date: <span id="current-date">{{ currentDate }}</span></span><br>
              <span>Current Time: <span id="current-time">{{ currentTime }}</span></span>
            </div>
            
            <div class="mt-6">
              <h4 class="font-serif">Colophon</h4><br>
              <p>
                Site: Harrison<br>
                Design: Harrison and David<br>
                Text: Times New Roman<br>
                Photography: David
              </p>
            </div>
          </div>
        </div>

        <!-- Coffee Menu View (Now absolutely positioned) -->
        <div
          id="menuContainer"
          v-show="showCoffeeMenu"
          class="absolute top-0 left-0 w-full h-full font-serif text-sm bg-white p-4 overflow-y-auto scrollbar-thin"
          style="opacity: 0; visibility: hidden;"
        >
          <!-- Simplified Menu Structure -->
          <div class="space-y-4 pt-2">
            <p class="text-lg font-bold text-center mb-4">Menu</p>
            <div v-for="category in coffeeMenuContent" :key="category.category" class="mb-4">
              <p class="font-semibold mb-1">{{ category.category }}</p>
              <div v-for="item in category.items" :key="item.name" class="flex justify-between">
                <span>{{ item.name }}</span>
                <span>{{ item.price }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Right column for both gallery and library card - they share the same space -->
      <div class="lg:flex-1 flex justify-end items-start pt-20 xl:pt-0">
        <!-- Gallery View -->
        <div 
          v-show="showGallery"
          class="w-full max-w-lg pr-0 ml-10 md:ml-30"
        >
          <div
            @click="handleImageClick"
            @mousemove="handleMouseMove"
            class="cursor-pointer flex justify-end relative w-full"
            :style="{ cursor: cursorPosition === 'left' ? 'url(https://sidetrack.us/img/left.png) 16 16, pointer' : 'url(https://sidetrack.us/img/right.png) 32 16, pointer' }"
          >
            <div v-if="!isCurrentImageLoaded" class="flex items-center justify-center h-[70vh] w-full">
              <p class="text-gray-500">Loading image...</p>
            </div>
            <div class="flex flex-col items-center">
              <img 
                v-for="(image, index) in images" 
                :key="index"
                :src="image.src" 
                :alt="image.alt"
                v-show="currentImageIndex === index && imagesLoaded[index]"
                class="max-w-full max-h-[70vh] object-contain p-0 m-0"
              >
              <div class="mt-6 text-center">
                <p 
                  v-for="(image, index) in images"
                  :key="`desc-${index}`"
                  v-show="currentImageIndex === index"
                >
                  {{ image.description }}
                </p>
                <div class="mt-2 text-sm">{{ currentImageIndex + 1 }}/{{ images.length }}</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Library Card View -->
        <div 
          v-show="showLibraryCard"
          class="w-full max-w-lg pr-0 ml-10 md:ml-30"
        >
          <div class="bg-white font-serif h-[70vh]">
            <!-- Library card table -->
            <table class="w-full text-sm border-b-3 border-t-3 border-black border-double">
              <thead>
                <tr class="border-b-3 border-black border-double">
                  <th class="font-thin font-serif italic text-center p-2 w-1/3 border-r border-black">DATE</th>
                  <th class="font-thin font-serif italic text-center p-2">ISSUED TO</th>
                </tr>
              </thead>
              <tbody class="overflow-y-auto">
                <!-- Current date and input row -->
                <tr class="border-b border-gray-300">
                  <td class="p-2 italic font-mono font-thin text-center" style="width: 33%;">{{ formattedCurrentDate }}</td>
                  <td class="p-0">
                    <input 
                      v-model="newEntry.message" 
                      class="w-full border-none outline-none font-mono font-thin italic p-2 bg-transparent"
                      type="text"
                      maxlength="50"
                      placeholder="Write your message here..."
                      @keyup.enter="submitLibraryCardSimple"
                    />
                  </td>
                </tr>
                
                <!-- Previous entries -->
                <tr v-for="(entry, index) in [...libraryCardEntries].reverse()" :key="index" class="border-b border-gray-300">
                  <td class="p-2 italic font-mono font-thin text-center" style="width: 33%;">{{ entry.date }}</td>
                  <td class="p-2 font-mono font-thin italic">{{ entry.message }}</td>
                </tr>
              </tbody>
            </table>
            
          </div>
        </div>
      </div>
    </div>

    <!-- Full screen gallery for mobile/tablet -->
    <div v-if="showGallery && !isLargeScreen" class="fixed inset-0 bg-white p-4 z-50 overflow-auto">
      <button @click="toggleGallery" class="absolute top-4 right-4 font-serif">x</button>
      <br>
      <div 
        class="mt-6 flex justify-center items-center"
        @click="handleImageClick"
        @mousemove="handleMouseMove"
        :style="{ cursor: cursorPosition === 'left' ? 'url(https://sidetrack.us/img/left.png) 16 16, pointer' : 'url(https://sidetrack.us/img/right.png) 32 16, pointer' }"
      >
        <img 
          v-for="(image, index) in images" 
          :key="`mobile-${index}`"
          :src="image.src" 
          :alt="image.alt"
          v-show="currentImageIndex === index"
          class="max-w-full max-h-[70vh] object-contain"
        >
      </div>
      <div class="mt-6 text-center">
        <p 
          v-for="(image, index) in images"
          :key="`mobile-desc-${index}`"
          v-show="currentImageIndex === index"
        >
          {{ image.description }}
        </p>
        <div class="mt-2 text-sm">{{ currentImageIndex + 1 }}/{{ images.length }}</div>
      </div>
    </div>

    <!-- Full screen library card for mobile/tablet -->
    <div v-if="showLibraryCard && !isLargeScreen" class="fixed inset-0 bg-white p-4 z-50 overflow-auto">
      <button @click="toggleLibraryCard" class="absolute top-4 right-4 font-serif">x</button>
      <br>
      <div class="mt-6 bg-white font-serif">
        <!-- Library card table -->
        <table class="w-full text-sm border-b-3 border-t-3 border-black border-double max-w-lg mx-auto">
          <thead>
            <tr class="border-b-3 border-black border-double">
              <th class="font-thin font-serif italic text-center p-2 w-1/3 border-r border-black">DATE</th>
              <th class="font-thin font-serif italic text-center p-2">ISSUED TO</th>
            </tr>
          </thead>
          <tbody class="overflow-y-auto">
            <!-- Current date and input row -->
            <tr class="border-b border-gray-300">
              <td class="p-2 italic font-mono font-thin text-center" style="width: 33%;">{{ formattedCurrentDate }}</td>
              <td class="p-0">
                <input 
                  v-model="newEntry.message" 
                  class="w-full border-none outline-none font-mono font-thin italic p-2 bg-transparent"
                  type="text"
                  maxlength="50"
                  placeholder="Write your message here..."
                  @keyup.enter="submitLibraryCardSimple"
                />
              </td>
            </tr>
            
            <!-- Previous entries -->
            <tr v-for="(entry, index) in [...libraryCardEntries].reverse()" :key="index" class="border-b border-gray-300">
              <td class="p-2 italic font-mono font-thin text-center" style="width: 33%;">{{ entry.date }}</td>
              <td class="p-2 font-mono font-thin italic">{{ entry.message }}</td>
            </tr>
          </tbody>
        </table>
        
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed, reactive, nextTick } from 'vue';
import gsap from 'gsap'; // Make sure GSAP is installed
import { SplitText } from 'gsap/SplitText'; // Make sure SplitText is installed

gsap.registerPlugin(SplitText);

// State
const showGallery = ref(false);
const showLibraryCard = ref(false);
const showCoffeeMenu = ref(false);
const isAnimating = ref(false);
const currentDate = ref('');
const currentTime = ref('');
const currentImageIndex = ref(0);
const windowWidth = ref(window.innerWidth);
const imagesLoaded = ref<Record<number, boolean>>({});
const isAllImagesLoaded = ref(false);
const cursorPosition = ref('right');

// Library card state
const libraryCardEntries = ref<{name?: string; message: string; date: string}[]>([
  { message: "First visitor, excited about the coffee!", date: "05/01/2023" },
  { message: "The space feels like home already", date: "05/02/2023" },
]);
const newEntry = reactive({
  message: ""
});

const coffeeMenuContent = [
  { category: "Coffee", items: [
    { name: "Espresso", price: "3.00" },
    { name: "Americano", price: "3.50" },
    { name: "Latte", price: "4.50" },
    { name: "Cappuccino", price: "4.25" },
    { name: "Mocha", price: "5.00" },
    { name: "Cortado", price: "3.75" },
    { name: "Batch Brew", price: "3.00" },
  ]},
  { category: "Not Coffee", items: [
    { name: "Chai Latte", price: "4.75" },
    { name: "Matcha Latte", price: "5.00" },
    { name: "Hot Chocolate", price: "4.00" },
    { name: "Herbal Tea", price: "3.25" },
  ]},
  { category: "Pastries", items: [
    { name: "Croissant", price: "3.50" },
    { name: "Almond Croissant", price: "4.00" },
    { name: "Scone", price: "3.75" },
  ]},
];

// Computed
const isLargeScreen = computed(() => windowWidth.value >= 1024);
const isCurrentImageLoaded = computed(() => imagesLoaded.value[currentImageIndex.value] === true);
const formattedCurrentDate = computed(() => {
  const now = new Date();
  return `${String(now.getMonth() + 1).padStart(2, '0')}/${String(now.getDate()).padStart(2, '0')}/${now.getFullYear()}`;
});

// Images data
const images = [
  { src: 'https://sidetrack.us/img/image-1.jpg', alt: 'flowers from ragan', description: 'Flowers from Ragan' },
  { src: 'https://sidetrack.us/img/image-2.jpg', alt: 'stone sitting by flowers from renee', description: 'Stone Sitting by Flowers from Renee' },
  { src: 'https://sidetrack.us/img/image-3.jpg', alt: 'scene of coffee, pastries, and friends', description: 'Scene of coffee, pastries, and friends' },
  { src: 'https://sidetrack.us/img/image-4.jpg', alt: 'taylor, morgon, flowers, and nat geo', description: 'Taylor, Morgon, Flowers, and Nat Geo' },
  { src: 'https://sidetrack.us/img/image-5.jpg', alt: 'our forever friend, marcos', description: 'Our Forever Friend, Marcos' },
  { src: 'https://sidetrack.us/img/image-6.jpg', alt: 'cappuccino in solitude', description: 'Cappuccino In Solitude' },
  { src: 'https://sidetrack.us/img/image-7.jpg', alt: 'françoise spinning usually', description: 'Françoise spinning usually' },
  { src: 'https://sidetrack.us/img/image-8.jpg', alt: 'cupping with mark, hannah, brielle, and paulina', description: 'Cupping with Mark, Hannah, Brielle, and Paulina' },
  { src: 'https://sidetrack.us/img/image-9.jpg', alt: 'brielle cupping coffees', description: 'Brielle cupping coffees' },
  { src: 'https://sidetrack.us/img/image-10.jpg', alt: 'riley looking at socks', description: 'Riley looking at socks' }
];

let sourceSplit: SplitText | null = null;
let targetSplit: SplitText | null = null;
let animationTimeline: gsap.core.Timeline | null = null;

// Methods
function toggleGallery() {
  if (isAnimating.value) return;
  showGallery.value = !showGallery.value;
  if (showGallery.value) {
    showLibraryCard.value = false;
    showCoffeeMenu.value = false;
    gsap.set('#menuContainer', { opacity: 0, visibility: 'hidden', display: 'none' });
    gsap.set('#sourceTextContainer', { opacity: 1, visibility: 'visible' });
  }
}

function toggleLibraryCard() {
  if (isAnimating.value) return;
  showLibraryCard.value = !showLibraryCard.value;
  if (showLibraryCard.value) {
    showGallery.value = false;
    showCoffeeMenu.value = false;
    gsap.set('#menuContainer', { opacity: 0, visibility: 'hidden', display: 'none' });
    gsap.set('#sourceTextContainer', { opacity: 1, visibility: 'visible' });
  }
}

function submitLibraryCardSimple() {
  if (newEntry.message.trim()) {
    const now = new Date();
    const formattedDate = `${String(now.getMonth() + 1).padStart(2, '0')}/${String(now.getDate()).padStart(2, '0')}/${now.getFullYear()}`;
    
    libraryCardEntries.value.push({
      message: newEntry.message,
      date: formattedDate
    });
    
    // Reset form
    newEntry.message = "";
  }
}

function nextImage() {
  currentImageIndex.value = (currentImageIndex.value + 1) % images.length;
}

function previousImage() {
  currentImageIndex.value = (currentImageIndex.value - 1 + images.length) % images.length;
}

function handleImageClick(event: MouseEvent) {
  const target = event.currentTarget as HTMLElement;
  const rect = target.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const thirdWidth = rect.width / 3;
  
  if (x < thirdWidth) {
    // Left third of the image
    previousImage();
  } else {
    // Right two-thirds of the image
    nextImage();
  }
}

function handleMouseMove(event: MouseEvent) {
  const target = event.currentTarget as HTMLElement;
  const rect = target.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const thirdWidth = rect.width / 3;
  
  cursorPosition.value = x < thirdWidth ? 'left' : 'right';
}

function openMap() {
  window.open('https://maps.google.com/?q=Side+Track+Coffee+Opelika+Alabama', '_blank');
}

function prefetchImages() {
  let loadedCount = 0;
  images.forEach((image, index) => {
    const img = new Image();
    img.onload = () => {
      imagesLoaded.value[index] = true;
      loadedCount++;
      if (loadedCount === images.length) {
        isAllImagesLoaded.value = true;
      }
    };
    img.onerror = () => {
      imagesLoaded.value[index] = false;
    };
    img.src = image.src;
  });
}

function updateDateTime() {
  const now = new Date();
  
  // Format time as HH:MM:SS
  const hours = String(now.getHours()).padStart(2, '0');
  const minutes = String(now.getMinutes()).padStart(2, '0');
  const seconds = String(now.getSeconds()).padStart(2, '0');
  currentTime.value = `${hours}:${minutes}:${seconds}`;
  
  // Format date
  const options: Intl.DateTimeFormatOptions = { 
    month: 'numeric', 
    day: 'numeric',
    year: 'numeric'
  };
  currentDate.value = now.toLocaleDateString('en-US', options);
}

function updateWindowWidth() {
  windowWidth.value = window.innerWidth;
}

async function animateToCoffeeMenu() {
  if (isAnimating.value) return;
  isAnimating.value = true;

  const sourceContainer = document.getElementById('sourceTextContainer');
  const menuContainer = document.getElementById('menuContainer');

  if (!sourceContainer || !menuContainer) {
    console.error("Source or target container not found");
    isAnimating.value = false;
    return;
  }

  if (!showCoffeeMenu.value) { // Animate TO menu
    showGallery.value = false; // Hide other panels if they were open
    showLibraryCard.value = false;
    showCoffeeMenu.value = true; // Make menu part of DOM via v-show

    await nextTick(); // Wait for DOM update

    // Temporarily make target measurable but visually hidden
    // Set display block temporarily for measurement, will be absolute positioned
    gsap.set(menuContainer, { display: 'block', opacity: 0, visibility: 'visible' });
    gsap.set(sourceContainer, { opacity: 1, visibility: 'visible' }); // Ensure source is visible

    sourceSplit?.revert();
    targetSplit?.revert();

    sourceSplit = new SplitText(sourceContainer, { type: "chars" });
    targetSplit = new SplitText(menuContainer, { type: "chars", charsClass: "targetChar" });

    if (!sourceSplit.chars || !targetSplit.chars) {
        console.error("Failed to split text.");
        gsap.set(menuContainer, { display: 'none', opacity: 0, visibility: 'hidden' }); // Reset style
        showCoffeeMenu.value = false;
        isAnimating.value = false;
        return;
    }

    if (targetSplit.chars) {
      gsap.set(targetSplit.chars, { opacity: 0 }); // Hide individual target chars initially
    }

    animationTimeline?.kill();
    animationTimeline = gsap.timeline({
      onComplete: () => {
        gsap.set(sourceContainer, { opacity: 0, visibility: 'hidden' });
        gsap.set(menuContainer, { opacity: 1, visibility: 'visible', display: 'block' }); // Ensure it's block for final display
        sourceSplit?.revert();
        targetSplit?.revert();
        isAnimating.value = false;
      },
      onStart: () => {
         gsap.set(menuContainer, { opacity: 0, visibility: 'hidden' }); // Keep container hidden during anim
      }
    });

    const sourceChars = sourceSplit.chars;
    const targetChars = targetSplit.chars;
    const numSource = sourceChars.length;
    const numTarget = targetChars.length;
    const duration = 1.8;

    for (let i = 0; i < numSource; i++) {
      const sourceChar = sourceChars[i];

      if (i < numTarget) {
        const targetChar = targetChars[i];
        const targetContent = targetChar.textContent;
        const targetBounds = targetChar.getBoundingClientRect();
        const sourceBounds = sourceChar.getBoundingClientRect();

        // Simpler calculation as they share the same positioned parent wrapper
        const targetX = targetBounds.left - sourceBounds.left;
        const targetY = targetBounds.top - sourceBounds.top;

        gsap.set(sourceChar, { willChange: 'transform, opacity' });

        const charStartTime = gsap.utils.random(0, 0.5);
        const charMoveDuration = duration * gsap.utils.random(0.7, 1.1);
        const charMorphDelay = charMoveDuration * 0.5;

        animationTimeline.to(sourceChar, {
            x: targetX,
            y: targetY,
            rotation: gsap.utils.random(-90, 90),
            scale: 1,
            duration: charMoveDuration,
            ease: "power1.inOut",
        }, charStartTime);

        animationTimeline.to(sourceChar, {
            opacity: 0.3,
            duration: charMorphDelay * 0.4,
            ease: "power1.out"
        }, charStartTime + charMoveDuration * 0.3);

        animationTimeline.call(() => {
            sourceChar.textContent = targetContent;
        }, [], charStartTime + charMorphDelay);

        animationTimeline.to(sourceChar, {
            opacity: 1,
            rotation: 0,
            duration: charMoveDuration * 0.5,
            ease: "power1.in"
        }, `>`);

      } else {
         animationTimeline.to(sourceChar, {
             x: gsap.utils.random(-250, 250),
             y: gsap.utils.random(-200, 200),
             opacity: 0,
             scale: 0.2,
             rotation: gsap.utils.random(-360, 360),
             duration: duration * gsap.utils.random(0.5, 0.9),
             ease: "power1.in",
             delay: gsap.utils.random(0, 0.3),
         }, 0);
      }
    }
     // Timing for final reveal/hide
     animationTimeline.set(menuContainer, { opacity: 1, visibility: 'visible' }, `+=${duration * 0.1}`); // Reveal menu near the end
     animationTimeline.set(sourceContainer, { opacity: 0, visibility: 'hidden' }, `<`); // Hide source at same time


  } else { // Animate FROM menu (simple fade)
      animationTimeline?.kill();
      sourceSplit?.revert();
      targetSplit?.revert();

      gsap.to(menuContainer, {
          opacity: 0,
          duration: 0.3,
          onComplete: () => {
              gsap.set(menuContainer, { opacity: 0, visibility: 'hidden', display: 'none' });
              showCoffeeMenu.value = false;
              gsap.set(sourceContainer, { opacity: 1, visibility: 'visible' });
              isAnimating.value = false;
          }
      });
  }
}

// Lifecycle
let dateTimeInterval: number;

onMounted(() => {
  updateDateTime();
  dateTimeInterval = setInterval(updateDateTime, 1000);
  window.addEventListener('resize', updateWindowWidth);
  prefetchImages();

  gsap.set('#menuContainer', { opacity: 0, visibility: 'hidden', display: 'none' }); // Ensure hidden initially
  gsap.set('#sourceTextContainer', { opacity: 1, visibility: 'visible' });
});

onUnmounted(() => {
  clearInterval(dateTimeInterval);
  window.removeEventListener('resize', updateWindowWidth);
  animationTimeline?.kill();
  sourceSplit?.revert();
  targetSplit?.revert();
});
</script>

<style>
@font-face {
  font-family: 'Times New Roman';
  src: local('Times New Roman');
}

body {
  font-family: 'Times New Roman', serif;
}

/* Table styling for scrollable tbody */
table {
  table-layout: fixed;
  width: 100%;
}

tbody {
  display: block;
  max-height: 60vh;
  overflow-y: auto;
}

thead, tbody tr {
  display: table;
  width: 100%;
  table-layout: fixed;
}

thead {
  width: calc(100%); /* Adjust for scrollbar width */
}

/* Minimal scrollbar styles */
tbody::-webkit-scrollbar {
  width: 4px;
  border-radius: 10px;
  margin-top: 5px;
  margin-bottom: 5px;
}

tbody::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
  margin-top: 5px;
  margin-bottom: 5px;
}

tbody::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
  margin-top: 5px;
  margin-bottom: 5px;
}

tbody::-webkit-scrollbar-thumb:hover {
  background: #555;
  border-radius: 10px;
  margin-top: 5px;
  margin-bottom: 5px;
}

/* Add styles for animation & overlay */
#sourceTextContainer {
  /* Ensure it maintains layout space */
  position: relative; /* Or static if parent handles positioning */
  z-index: 1; /* Below menu when menu is shown */
}

#menuContainer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%; /* Or max-height if content might overflow */
  max-height: 100%; /* Example max height */
  /* Add background if needed, e.g., bg-white */
  z-index: 2; /* Above source text when shown */
  overflow-y: auto; /* Allow scrolling within the overlay */
}

.targetChar {
}
</style>

