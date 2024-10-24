<template>
  <div class="w-screen h-screen fixed top-0 left-0 z-[5000] flex flex-col">
    <VSpinner v-if="isLoading" />
    <SearchBar
      :label="otu.object_tag"
      @close="() => emit('close')"
    />
    <div
      :class="[
        'w-full',
        'h-full',
        'relative',
        { 'disable-zoom-out': disableZoom }
      ]"
    >
      <VMap
        ref="mapRef"
        class="w-full h-full"
        controls
        :dragging="!disableZoom"
        :zoom-bounds="maxZoom"
        :geojson="shapes"
        :geojsonOptions="geojsonOptions"
        @geojson:ready="updateMaxZoom"
        @add:layer="(layer) => loadOTUs(JSON.stringify(layer.geometry))"
        @edit:layer="(layer) => loadOTUs(JSON.stringify(layer.geometry))"
        @drag:layer="(layer) => loadOTUs(JSON.stringify(layer.geometry))"
        @zoom:change="handleZoom"
        :zoom="4"
      />

      <div ref="popupElement">
        <MapPopup
          v-if="popupItem"
          :items="popupItem.base"
          @selected="dwcTableRef.show"
        />
      </div>

      <div
        :class="[
          'h-full',
          'md:w-96',
          'absolute',
          'top-0',
          'bg-base-background',
          'z-[2000]',
          'overflow-auto',
          'ease-in-out',
          'duration-300',
          'w-full',
          {
            'md:-right-96 -right-full': !isTableVisible,
            'right-0': isTableVisible
          }
        ]"
      >
        <ListResults
          v-if="isTableVisible"
          :list="list"
          @close="() => emit('close')"
          @close:table="() => (isTableVisible = false)"
        />
      </div>
    </div>
    <DwcTable ref="dwcTableRef" />
  </div>
</template>

<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue'
import { useGeojsonOptions } from '../../composables/useGeojsonOptions.js'
import TaxonWorks from '../../../../../services/TaxonWorks.js'
import SearchBar from './SearchBar.vue'
import ListResults from './ListResults.vue'
import MapPopup from '../MapPopup.vue'
import DwcTable from '../DwcTable.vue'

const props = defineProps({
  shapes: {
    type: Object,
    default: () => []
  },

  otu: {
    type: Object,
    default: undefined
  }
})

const emit = defineEmits(['close'])

const popupElement = ref(null)
const dwcTableRef = ref(null)
const mapRef = ref(null)
const list = ref([])
const isTableVisible = ref(false)
const isLoading = ref()
const maxZoom = ref(6)
const currentZoom = ref(6)
const { popupItem, geojsonOptions } = useGeojsonOptions({ popupElement })
const disableZoom = computed(
  () => !!props.shapes && currentZoom.value <= maxZoom.value
)

onMounted(() => {
  document.addEventListener('keyup', handleKeyboard)
  document.body.classList.add('overflow-hidden')
})

onUnmounted(() => {
  document.removeEventListener('keyup', handleKeyboard)
  document.body.classList.remove('overflow-hidden')
})

const handleKeyboard = ({ key }) => {
  switch (key) {
    case 'Escape':
      emit('close')
      break
  }
}

function loadOTUs(geojson) {
  const payload = {
    geo_json: geojson,
    taxon_name_id: [props.otu.taxon_name_id],
    descendants: true,
    per: 5000
  }

  isLoading.value = true

  TaxonWorks.getOtus(payload)
    .then(({ data }) => {
      list.value = data
      isTableVisible.value = true
    })
    .finally((_) => {
      isLoading.value = false
    })
}

function handleZoom(e) {
  updateCurrentZoom()
  if (disableZoom.value) {
    mapRef.value.resizeMap()
  }
}

function updateCurrentZoom() {
  currentZoom.value = mapRef.value.getMapObject().getZoom()
}

function updateMaxZoom() {
  maxZoom.value = mapRef.value.getMapObject().getZoom()
  updateCurrentZoom()
}
</script>

<style lang="scss" scoped>
:deep(.disable-zoom-out .leaflet-control-zoom-out) {
  display: none !important;
}
</style>
