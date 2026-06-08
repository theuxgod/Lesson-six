<script setup lang="ts">
import { onMounted, ref } from 'vue'

type Status = 'idle' | 'locating' | 'loading' | 'ready' | 'error'
type Source = 'gps' | 'ip' | 'default'

interface WeatherData {
  temperature: number
  windSpeed: number
  weatherCode: number
  unit: string
}

interface ResolvedLocation {
  lat: number
  lon: number
  label?: string
  source: Source
}

const DEFAULT_LOCATION: ResolvedLocation = {
  lat: 40.7128,
  lon: -74.006,
  label: 'New York, NY',
  source: 'default',
}

const status = ref<Status>('idle')
const error = ref<string>('')
const weather = ref<WeatherData | null>(null)
const location = ref<ResolvedLocation | null>(null)

// Subset of WMO weather interpretation codes → label + MDI icon
const codeMap: Record<number, { label: string; icon: string }> = {
  0: { label: 'Clear sky', icon: 'mdi-weather-sunny' },
  1: { label: 'Mainly clear', icon: 'mdi-weather-partly-cloudy' },
  2: { label: 'Partly cloudy', icon: 'mdi-weather-partly-cloudy' },
  3: { label: 'Overcast', icon: 'mdi-weather-cloudy' },
  45: { label: 'Fog', icon: 'mdi-weather-fog' },
  48: { label: 'Rime fog', icon: 'mdi-weather-fog' },
  51: { label: 'Light drizzle', icon: 'mdi-weather-partly-rainy' },
  53: { label: 'Drizzle', icon: 'mdi-weather-partly-rainy' },
  55: { label: 'Heavy drizzle', icon: 'mdi-weather-rainy' },
  61: { label: 'Light rain', icon: 'mdi-weather-rainy' },
  63: { label: 'Rain', icon: 'mdi-weather-rainy' },
  65: { label: 'Heavy rain', icon: 'mdi-weather-pouring' },
  71: { label: 'Light snow', icon: 'mdi-weather-snowy' },
  73: { label: 'Snow', icon: 'mdi-weather-snowy' },
  75: { label: 'Heavy snow', icon: 'mdi-weather-snowy-heavy' },
  80: { label: 'Rain showers', icon: 'mdi-weather-pouring' },
  81: { label: 'Rain showers', icon: 'mdi-weather-pouring' },
  82: { label: 'Violent showers', icon: 'mdi-weather-pouring' },
  95: { label: 'Thunderstorm', icon: 'mdi-weather-lightning' },
  96: { label: 'Thunderstorm w/ hail', icon: 'mdi-weather-lightning' },
  99: { label: 'Thunderstorm w/ hail', icon: 'mdi-weather-lightning' },
}

const describe = (code: number) =>
  codeMap[code] ?? { label: 'Unknown', icon: 'mdi-weather-cloudy-alert' }

const fetchWeather = async (loc: ResolvedLocation) => {
  status.value = 'loading'
  try {
    const url = new URL('https://api.open-meteo.com/v1/forecast')
    url.searchParams.set('latitude', loc.lat.toFixed(4))
    url.searchParams.set('longitude', loc.lon.toFixed(4))
    url.searchParams.set('current_weather', 'true')
    url.searchParams.set('temperature_unit', 'fahrenheit')
    url.searchParams.set('wind_speed_unit', 'mph')

    const res = await fetch(url.toString())
    if (!res.ok) throw new Error(`Weather API responded ${res.status}`)
    const data = await res.json()
    const cw = data.current_weather
    if (!cw) throw new Error('No current weather in response')

    weather.value = {
      temperature: cw.temperature,
      windSpeed: cw.windspeed,
      weatherCode: cw.weathercode,
      unit: '°F',
    }
    location.value = loc
    status.value = 'ready'
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Failed to load weather'
    status.value = 'error'
  }
}

const getBrowserLocation = () =>
  new Promise<ResolvedLocation>((resolve, reject) => {
    if (!('geolocation' in navigator)) {
      reject(new Error('Geolocation not supported'))
      return
    }
    navigator.geolocation.getCurrentPosition(
      (pos) =>
        resolve({
          lat: pos.coords.latitude,
          lon: pos.coords.longitude,
          source: 'gps',
        }),
      (err) => reject(new Error(err.message || 'Location access denied')),
      { timeout: 10000, maximumAge: 5 * 60 * 1000 },
    )
  })

const getIpLocation = async (): Promise<ResolvedLocation> => {
  const res = await fetch('https://get.geojs.io/v1/ip/geo.json')
  if (!res.ok) throw new Error(`IP lookup failed (${res.status})`)
  const data = await res.json()
  const lat = Number(data.latitude)
  const lon = Number(data.longitude)
  if (!Number.isFinite(lat) || !Number.isFinite(lon)) {
    throw new Error('IP lookup returned no coordinates')
  }
  const label = [data.city, data.region].filter(Boolean).join(', ') || data.country
  return { lat, lon, label, source: 'ip' }
}

const load = async () => {
  status.value = 'locating'
  error.value = ''
  try {
    const loc = await getBrowserLocation()
    await fetchWeather(loc)
  } catch {
    try {
      const loc = await getIpLocation()
      await fetchWeather(loc)
    } catch {
      await fetchWeather(DEFAULT_LOCATION)
    }
  }
}

onMounted(load)
</script>

<template>
  <div class="weather-widget pa-3 rounded-lg">
    <template v-if="status === 'ready' && weather">
      <div class="d-flex align-center justify-center ga-3">
        <v-icon :icon="describe(weather.weatherCode).icon" size="36" />
        <div class="text-left">
          <div class="text-h6 font-weight-bold">
            {{ Math.round(weather.temperature) }}{{ weather.unit }}
          </div>
          <div class="text-caption text-medium-emphasis">
            {{ describe(weather.weatherCode).label }}
          </div>
        </div>
      </div>
      <div
        v-if="location && location.source !== 'gps'"
        class="text-caption text-medium-emphasis text-center mt-2"
      >
        <v-icon
          :icon="location.source === 'ip' ? 'mdi-map-marker-outline' : 'mdi-map-marker-off-outline'"
          size="12"
          class="mr-1"
        />
        {{
          location.source === 'ip'
            ? `Approximate location: ${location.label}`
            : `Showing default: ${location.label}`
        }}
      </div>
    </template>

    <template v-else-if="status === 'locating' || status === 'loading'">
      <div class="d-flex align-center justify-center ga-2">
        <v-progress-circular indeterminate size="20" width="2" />
        <span class="text-caption text-medium-emphasis">
          {{ status === 'locating' ? 'Finding your location…' : 'Loading weather…' }}
        </span>
      </div>
    </template>

    <template v-else-if="status === 'error'">
      <div class="d-flex flex-column align-center ga-2">
        <span class="text-caption text-medium-emphasis">
          <v-icon icon="mdi-alert-circle-outline" size="14" class="mr-1" />
          {{ error }}
        </span>
        <v-btn size="x-small" variant="text" @click="load">Retry</v-btn>
      </div>
    </template>
  </div>
</template>

<style scoped>
.weather-widget {
  background-color: rgba(127, 127, 127, 0.08);
}
</style>
