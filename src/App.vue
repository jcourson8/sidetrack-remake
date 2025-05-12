<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed, reactive } from 'vue';

// State
const showGallery = ref(false);
const showLibraryCard = ref(false);
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

// Methods
function toggleGallery() {
  showGallery.value = !showGallery.value;
  if (showGallery.value) {
    showLibraryCard.value = false;
  }
}

function toggleLibraryCard() {
  showLibraryCard.value = !showLibraryCard.value;
  if (showLibraryCard.value) {
    showGallery.value = false;
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

// Lifecycle
let dateTimeInterval: number;

onMounted(() => {
  // Set initial date/time and update every second
  updateDateTime();
  dateTimeInterval = setInterval(updateDateTime, 1000);
  
  // Add resize listener
  window.addEventListener('resize', updateWindowWidth);
  
  // Prefetch images
  prefetchImages();
});

onUnmounted(() => {
  clearInterval(dateTimeInterval);
  window.removeEventListener('resize', updateWindowWidth);
});
</script>

<template>
  <div class="min-h-screen flex flex-col font-serif pt-10 xl:pt-20 px-10 md:px-30">
    <!-- Layout container that changes direction on larger screens -->
    <div class="flex flex-col lg:flex-row flex-grow">
      <!-- New leftmost column for very large screens -->
      <div class="hidden xl:block xl:w-[250px] mr-10">
        <h1 class="font-serif">Side Track</h1>
        <h3 class="font-serif italic">Construction Website</h3>
      </div>

      <!-- Left column (header + main content) -->
      <div class="max-w-[350px]">
        <!-- Header -->
        <div class="mb-8 xl:hidden">
          <h1 class="font-serif ">Side Track</h1>
          <h3 class="font-serif italic">Construction Website</h3>
        </div>

        <!-- Body Content -->
        <div class="space-y-4">
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
          
          <div class="mt-4">
            <span>Current Date: <span id="current-date">{{ currentDate }}</span></span><br>
            <span>Current Time: <span id="current-time">{{ currentTime }}</span></span>
          </div>
          
          <div class="mt-4">
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
      
      <!-- Right column for both gallery and library card - they share the same space -->
      <div class="lg:flex-1 flex justify-end items-start pt-20 xl:pt-0">
        <!-- Gallery View -->
        <div 
          v-if="showGallery"
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
              <div class="mt-4 text-center">
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
          v-if="showLibraryCard"
          class="w-full max-w-md pr-0 ml-10 md:ml-30"
        >
          <div class="bg-white font-serif h-[70vh]">
            <!-- Library card table -->
            <table class="w-full text-sm border-b-3 border-t-3 border-black border-double">
              <thead>
                <tr class="border-b-3 border-black border-double">
                  <th class="font-thin font-serif italic text-center p-2 w-1/3 border-r border-black">Date</th>
                  <th class="font-thin font-serif italic text-center p-2">Issued To</th>
                </tr>
              </thead>
              <tbody class="overflow-y-auto">
                <!-- Current date and input row -->
                <tr class="border-b border-gray-300">
                  <td class="pt-2 pb-1 px-2 italic font-special-elite font-thin text-center" style="width: 33%;">{{ formattedCurrentDate }}</td>
                  <td class="p-0">
                    <input 
                      v-model="newEntry.message" 
                      class="w-full border-none outline-none font-serif font-thin italic p-2 bg-transparent"
                      type="text"
                      maxlength="50"
                      placeholder="Write your message here..."
                      @keyup.enter="submitLibraryCardSimple"
                    />
                  </td>
                </tr>
                
                <!-- Previous entries -->
                <tr v-for="(entry, index) in [...libraryCardEntries].reverse()" :key="index" class="border-b border-gray-300">
                  <td class="pt-2 pb-1 px-2 italic font-special-elite font-thin text-center" style="width: 33%;">{{ entry.date }}</td>
                  <td class="p-2 font-serif font-thin italic">{{ entry.message }}</td>
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
        class="mt-4 flex justify-center items-center"
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
      <div class="mt-4 text-center">
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
      <div class="mt-4 bg-white font-serif">
        <!-- Library card table -->
        <table class="w-full text-sm border-b-3 border-t-3 border-black border-double max-w-lg mx-auto">
          <thead>
            <tr class="border-b-3 border-black border-double">
              <th class="font-thin font-serif italic text-center p-2 w-1/3 border-r border-black">Date</th>
              <th class="font-thin font-serif italic text-center p-2">Issued To</th>
            </tr>
          </thead>
          <tbody class="overflow-y-auto">
            <!-- Current date and input row -->
            <tr class="border-b border-gray-300">
              <td class="pt-2 pb-1 px-2 italic font-special-elite font-thin text-center" style="width: 33%;">{{ formattedCurrentDate }}</td>
              <td class="p-0">
                <input 
                  v-model="newEntry.message" 
                  class="w-full border-none outline-none font-serif font-thin italic p-2 bg-transparent"
                  type="text"
                  maxlength="50"
                  placeholder="Write your message here..."
                  @keyup.enter="submitLibraryCardSimple"
                />
              </td>
            </tr>
            
            <!-- Previous entries -->
            <tr v-for="(entry, index) in [...libraryCardEntries].reverse()" :key="index" class="border-b border-gray-300">
              <td class="pt-2 pb-1 px-2 italic font-special-elite font-thin text-center" style="width: 33%;">{{ entry.date }}</td>
              <td class="p-2 font-serif font-thin italic">{{ entry.message }}</td>
            </tr>
          </tbody>
        </table>
        
      </div>
    </div>
  </div>
</template>

<style>
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
</style>

