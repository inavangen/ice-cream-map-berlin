<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import heroImg from '../assets/hero.png'

import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer, LMarker, LPopup } from "@vue-leaflet/vue-leaflet"

import L from "leaflet"

import customMarker from '../assets/marker2x.png'

// Create custom icon 
const customIcon = L.icon({
  iconUrl: customMarker,
  iconSize: [38, 62],
  iconAnchor: [19, 62],
  popupAnchor: [0, -53],
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
  shadowSize: [31, 31],
  shadowAnchor: [9, 31]
})

import arcades from "./arcades.json"

let zoom = ref(13)
let center = ref<[number, number]>([52.511439, 13.468885])

// Modal state
const showModal = ref(false)
const selectedLocation = ref<any>(null)

// Placeholder image URL
const placeholderImage = "https://via.placeholder.com/400x200/4CAF50/ffffff?text=Ice+Cream+Shop"

// Open modal with selected location data
const openModal = (location: any) => {
  selectedLocation.value = location
  showModal.value = true
  // Prevent body scroll on mobile
  document.body.classList.add('modal-open')
}

// Close modal
const closeModal = () => {
  showModal.value = false
  selectedLocation.value = null
  // Re-enable body scroll
  document.body.classList.remove('modal-open')
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
  const reversed = coordinates.slice()
  if (reversed.length !== 2) {
    console.error('Invalid coordinates:', coordinates)
    return [0, 0]
  }
  return [reversed[0], reversed[1]] as [number, number]
}

// Handle escape key to close modal
const handleEscapeKey = (event: KeyboardEvent) => {
  if (event.key === 'Escape' && showModal.value) {
    closeModal()
  }
}

// Watch for modal visibility to add/remove event listeners
watch(showModal, (newVal) => {
  if (newVal) {
    document.addEventListener('keydown', handleEscapeKey)
  } else {
    document.removeEventListener('keydown', handleEscapeKey)
  }
})

// Clean up event listeners on component unmount
onMounted(() => {
  console.log('Arcades data:', arcades)
  console.log('Number of ice cream reviews:', arcades.reviews?.length)
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
      <p>A map over Rebecca's favorite Ice Creams</p>
    </div>
    
    <!-- Map view and pins -->
    <div style="height: 600px; width: 100%; border: 4px solid #A34390; border-radius: 12px; overflow: hidden;">
      <l-map ref="map" v-model:zoom="zoom" v-model:center="center" :useGlobalLeaflet="false">
        <l-tile-layer 
          url="https://tile.openstreetmap.org/{z}/{x}/{y}.png"
          layer-type="base"
          name="Open Free Map"
        ></l-tile-layer>
        
        <l-marker 
          v-for="(arcade, index) in arcades.reviews" 
          :key="index"
          :lat-lng="getCoordinates(arcade.geometry.coordinates)"
          :icon="customIcon"
        >
          <l-popup>
            <div class="popup-content">
              <!-- Circular thumbnail -->
              <div class="popup-thumbnail-wrapper">
                <img 
                  :src="getImageUrl(arcade)"
                  alt="Arcade thumbnail"
                  class="popup-circular-thumb"
                />
              </div>
              
              <b>{{ arcade.properties.name }}</b><br>
              <i>{{ arcade.properties.address }}</i> <br><br>
              <span class="description">{{ arcade.properties.description }}</span><br>
              <b>Rating: </b> {{ arcade.properties.rating }}
              
              <!-- Pink "Read full review" button -->
              <button class="popup-button" @click="openModal(arcade)">
                Read full review
              </button>
            </div>
          </l-popup>
        </l-marker>
      </l-map>
    </div>
  </section>

  <!-- Pop up Modal  -->
<div v-if="showModal" class="modal-overlay" @click.self="closeModal">
  <div class="modal-container">
    <div class="modal-content">
      <button class="modal-close" @click="closeModal">&times;</button>
      
      <div class="modal-header">
        <h2>{{ selectedLocation?.properties?.name || 'Location Details' }}</h2>
      </div>
      
      <!-- Image in larger popup -->
      <div class="modal-image-wrapper">
        <img 
          :src="getImageUrl(selectedLocation)"
          alt="Location image"
          class="modal-large-image"
        />
      </div>
      
      <div class="modal-body">
        <p><strong>Date of visit:</strong> {{ selectedLocation?.properties?.date || 'Date not found' }}</p>
        <p><strong>Address:</strong> {{ selectedLocation?.properties?.address || 'Address not specified' }}</p>
        <p><strong>Description:</strong> {{ selectedLocation?.properties?.description || 'No description available' }}</p>
        <p><strong>Rating:</strong> {{ selectedLocation?.properties?.rating || 'Information coming soon' }}</p>
        
        <!-- Google Maps link -->
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
</div>

  <!-- Information, credits and links here -->
  <div class="ticks"></div>

  <section id="next-steps">
    <div id="docs">
      <h2>Who's behind this?</h2>
      <p>This webpage is developed and designed by Ina and reviews are written by Rebecca. Got some feedback, or a suggest to where we should go next?</p>
      <ul>
        <li>
          <a href="https://docs.google.com/forms/d/e/1FAIpQLSfcuZ-BfYXPTlZ5Fmb_GV5JvcO1baCuWx48Cqc9Kof-3WPSNQ/viewform?usp=publish-editor" target="_blank">
            Let us know!
          </a>
        </li>
      </ul>
    </div>
    <div id="social">
      <h2>Support the ice creams</h2>
      <p>Ice creams are expensive. You can support our adventure to buy more ice creams with a small donation!</p>
      <ul>
        <li>
          <a href="https://ko-fi.com/ina553388" target="_blank">
            Buy us an ice cream!
          </a>
        </li>
      </ul>
    </div>
  </section>

  <div class="ticks"></div>
  <section id="spacer"></section>
</template>