<!-- ./src/components/FieldPluginExample/Address.vue -->
<script setup lang="ts">
import { computed, ref, watch, onMounted } from 'vue'
import { useFieldPlugin } from '@storyblok/field-plugin/vue3'
import Cleave from 'cleave.js'
import 'cleave.js/dist/addons/cleave-phone.at'

// The URL of the OpenStreetMap endpoint
// for fetching location data by address.
const ENDPOINT = 'https://nominatim.openstreetmap.org/search'

const {
  actions: { setContent },
  data: { content },
} = useFieldPlugin()

const latLngNotFound = ref(false)

const address = ref({
  // These are the values which our custom
  // field will return.
  country: '',
  email: '',
  fax: '',
  latitude: null as number | null,
  longitude: null as number | null,
  phone: '',
  postal_code: '',
  street: '',
  town: '',
  // This is the name of our plugin.
  plugin: 'address',
})

const coordinatesByaddress = async () => {
  try {
    // Reset the error message before
    // fetching new location data.
    latLngNotFound.value = false

    // Here we build the query string with
    // all the address data available to us
    const queryString = [
      `city=${encodeURI(address.value.town)}`,
      `country=${encodeURI(address.value.country)}`,
      `postalcode=${encodeURI(address.value.postal_code)}`,
      `street=${encodeURI(address.value.street)}`,
      'format=jsonv2',
    ].join('&')

    // We use the new `fetch` API to query
    // the public OpenStreetMap API.
    const rawResponse = await fetch(`${ENDPOINT}?${queryString}`)
    const responseJson = await rawResponse.json()
    const bestMatch = responseJson[0]

    console.log('bestMatch', bestMatch)

    // Throw error if address is not found.
    if (!bestMatch) throw new Error('address not found')

    // If OpenStreetMap was able to find us
    // some coordinates, we update our model.
    address.value.latitude = parseFloat(bestMatch.lat)
    address.value.longitude = parseFloat(bestMatch.lon)
  } catch (error) {
    latLngNotFound.value = true
  }
}

const phoneOptions = {
  phone: true,
  phoneRegionCode: 'NG',
}

onMounted(() => {
  new Cleave('#phone', phoneOptions)
  new Cleave('#fax', phoneOptions)
})

watch(address.value, (value) => {
  setContent(value)
})
</script>

<template>
  <div class="address">
    <div class="form-field address__field">
      <label class="form__topic"> Street </label>
      <input
        v-model="address.street"
        class="form-input uk-form-small uk-width-1-1"
      />
    </div>

    <div class="form-field address__field">
      <label class="form__topic"> Town </label>
      <input
        v-model="address.town"
        class="form-input uk-form-small uk-width-1-1"
      />
    </div>

    <div class="form-field address__field">
      <label class="form__topic"> Postal code </label>
      <input
        v-model="address.postal_code"
        class="form-input uk-form-small uk-width-1-1"
      />
    </div>

    <div class="form-field address__field">
      <label class="form__topic"> Country </label>
      <input
        v-model="address.country"
        class="form-input uk-form-small uk-width-1-1"
      />
    </div>

    <div class="form-field address__field">
      <span class="form__topic"> Coordinates </span>
      <div class="form-field-group uk-grid">
        <label class="uk-width-1-2">
          Latitude
          <input
            v-model.number="address.latitude"
            class="form-input uk-form-small uk-width-1-1"
          />
        </label>
        <label class="uk-width-1-2">
          Longitude
          <input
            v-model.number="address.longitude"
            class="form-input uk-form-small uk-width-1-1"
          />
        </label>
      </div>

      <a
        class="btn blok__full-btn uk-margin-small-top"
        @click="coordinatesByaddress"
      >
        <i class="uk-icon-search uk-margin-small-right" />
        Generate from address
      </a>
      <p
        v-if="latLngNotFound"
        class="address__error"
      >
        Could not find coordinates for the given address.
      </p>
    </div>

    <div class="form-field address__field">
      <label class="form__topic"> Phone </label>
      <input
        id="phone"
        v-model.lazy="address.phone"
        class="form-input uk-form-small uk-width-1-1"
      />
    </div>

    <div class="form-field address__field">
      <label class="form__topic"> Fax </label>
      <input
        id="fax"
        v-model.lazy="address.fax"
        class="form-input uk-form-small uk-width-1-1"
      />
    </div>

    <div class="form-field address__field">
      <label class="form__topic"> E-Mail </label>
      <input
        v-model="address.email"
        type="email"
        class="form-input uk-form-small uk-width-1-1"
      />
    </div>
  </div>
</template>
<style scoped>
.address {
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
}
.form-field .address__field + .form-field .address__field {
  margin-top: 0.85rem;
}

.address__error {
  margin-top: 5px;
  margin-bottom: 0;
  color: #d85030;
}
</style>
