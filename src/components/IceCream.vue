<script setup lang="ts">
import { ref, onMounted, } from 'vue'
import viteLogo from '../assets/vite.svg'
import heroImg from '../assets/hero.png'

import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer, LMarker, LPopup } from "@vue-leaflet/vue-leaflet"

import L from "leaflet"

// POI icon paths 
delete (L.Icon.Default.prototype as any)._getIconUrl
L.Icon.Default.mergeOptions({
  iconRetinaUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-icon-2x.png',
  iconUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-icon.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
})

import arcades from "./arcades.json"

let zoom = ref(15)
// Fix: Define center as a tuple with exactly 2 numbers
let center = ref<[number, number]>([52.511439, 13.468885])

// Modal state
const showModal = ref(false)
const selectedLocation = ref<any>(null)

// Placeholder image URL (you can replace this with your own placeholder)
const placeholderImage = "https://via.placeholder.com/400x200/4CAF50/ffffff?text=Ice+Cream+Shop"

// Open modal with selected location data
const openModal = (location: any) => {
  selectedLocation.value = location
  showModal.value = true
}

// Close modal
const closeModal = () => {
  showModal.value = false
  selectedLocation.value = null
}

// Get image URL from location properties or use placeholder
const getImageUrl = (location: any) => {
  if (location?.properties?.image_url) {
    return location.properties.image_url
  }
  return placeholderImage
}

// Helper function to convert coordinates to tuple
const getCoordinates = (coordinates: number[]): [number, number] => {
  // Coordinates from GeoJSON are [lng, lat], so we need to reverse to [lat, lng] for Leaflet. Use .reverse() behind .slice() if needed
  const reversed = coordinates.slice()
  if (reversed.length !== 2) {
    console.error('Invalid coordinates:', coordinates)
    return [0, 0]
  }
  return [reversed[0], reversed[1]] as [number, number]
}

// Debug: Check if data is loading
onMounted(() => {
  console.log('Arcades data:', arcades)
  console.log('Number of features:', arcades.features?.length)
})
</script>

<!-- Header stuff -->
<template>
  <section id="center">
    <div class="hero">
      <img :src="heroImg" class="base" width="170" height="179" alt="" />
    </div>
    <div>
      <h1>EIS EIS BABY!</h1>
      <p>A map over Rebecca's favorite Ice Creams in Berlin</p>
    </div>
    
    <!-- Map view and pins -->
    <div style="height: 600px; width: 100%; border: 3px solid #D93EDE; border-radius: 8px; overflow: hidden;">
      <l-map ref="map" v-model:zoom="zoom" v-model:center="center" :useGlobalLeaflet="false">
      <l-tile-layer url="https://tiles.stadiamaps.com/tiles/osm_bright/{z}/{x}/{y}{r}.png"
                    layer-type="base"
                    name="Stadia Maps Basemap"></l-tile-layer>
      
      <l-marker v-for="(arcade, index) in arcades.features" 
                :key="index"
                :lat-lng="getCoordinates(arcade.geometry.coordinates)">
        <l-popup>
          <div class="popup-content">
            <h3>{{ arcade.properties.name }}</h3>
            <i> {{ arcade.properties.address }}</i> <br></br><br></br>
            <span class="description">{{ arcade.properties.description }}</span><br></br>
            <b>Rating: </b> {{ arcade.properties.rating }}<br></br>
            <br />
            <button class="popup-button" @click="openModal(arcade)">
              More Info
            </button>
          </div>
        </l-popup>
      </l-marker>
    </l-map>

    </div>
  </section>

  <!-- Pop up Modal -->
  <div v-if="showModal" class="modal-overlay" @click="closeModal">
    <div class="modal-content" @click.stop>
      <div class="modal-header">
        <h2>{{ selectedLocation?.properties?.name || 'Location Details' }}</h2>
        <button class="modal-close" @click="closeModal">&times;</button>
      </div>
      
      <!-- Image -->
      <div class="modal-image-container">
        <img 
          :src="getImageUrl(selectedLocation)" 
          :alt="selectedLocation?.properties?.name || 'Ice Cream Shop'"
          class="modal-image"
          @error="(e) => (e.target as HTMLImageElement).src = placeholderImage"
          />
      </div> 
      
      <div class="modal-body">
        <!-- Add more properties based on your data structure -->
        <p><strong>Date of visit:</strong> {{ selectedLocation?.properties?.date || 'Date not found' }}</p>
        <p><strong>Address:</strong> {{ selectedLocation?.properties?.address || 'Address not specified' }}</p>
        <p><strong>Description:</strong> {{ selectedLocation?.properties?.description || 'No description available' }}</p>
        <p><strong>Rating:</strong> {{ selectedLocation?.properties?.rating || 'Information coming soon' }}</p>
        
        <!-- You can add a link to Google Maps or other external services -->
        <a 
          v-if="selectedLocation"
          :href="`https://www.google.com/maps?q=${selectedLocation.geometry.coordinates[1]},${selectedLocation.geometry.coordinates[0]}`" 
          target="_blank"
          class="modal-link"
        >
          Open in Google Maps
        </a>
      </div>
    </div>
  </div>

  <!-- Information, credits and links here -->
  <div class="ticks"></div>

  <section id="next-steps">
    <div id="docs">
      <svg class="icon" role="presentation" aria-hidden="true">
        <use href="/icons.svg#documentation-icon"></use>
      </svg>
      <h2>Who's behind this?</h2>
      <p>Rebecca and Ina. Suggest where we should go next? </p>
      <ul>
        <li>
          <a href="https://vite.dev/" target="_blank">
            <img class="logo" :src="viteLogo" alt="" />
            Let us know!
          </a>
        </li>
      </ul>
    </div>
    <div id="social">
      <svg class="icon" role="presentation" aria-hidden="true">
        <use href="/icons.svg#social-icon"></use>
      </svg>
      <h2>Support the ice creams</h2>
      <p>Ice creams are expensive. You can support our adventure to buy more ice creams</p>
      <ul>
        <li>
          <a href="https://github.com/vitejs/vite" target="_blank">
            <svg class="button-icon" role="presentation" aria-hidden="true">
              <use href="/icons.svg#github-icon"></use>
            </svg>
            Buy us an ice cream (coffee)
          </a>
        </li>

      </ul>
    </div>
  </section>

  <div class="ticks"></div>
  <section id="spacer"></section>
</template>