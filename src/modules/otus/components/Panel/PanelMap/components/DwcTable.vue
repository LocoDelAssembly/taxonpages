<template>
  <VModal
    v-if="isModalVisible"
    @close="() => (isModalVisible = false)"
  >
    <template #header>
      <h3 class="font-medium">{{ title }}</h3>
    </template>
    <VTable class="p-4 pt-0">
      <VTableHeader>
        <VTableHeaderRow>
          <VTableHeaderCell>Field</VTableHeaderCell>
          <VTableHeaderCell>Value</VTableHeaderCell>
        </VTableHeaderRow>
      </VTableHeader>
      <VTableBody>
        <VTableBodyRow
          v-for="(value, key) in dwcData"
          :key="key"
        >
          <VTableBodyCell>{{ key }} </VTableBodyCell>
          <VTableBodyCell>{{ value }} </VTableBodyCell>
        </VTableBodyRow>
      </VTableBody>
    </VTable>
    <VSpinner v-if="isLoading" />
  </VModal>
</template>

<script setup>
import { ref } from 'vue'
import { makeAPIRequest } from '@/utils'

const isLoading = ref(false)
const isModalVisible = ref(false)
const dwcData = ref({})
const title = ref()

function show({ label, id }) {
  isModalVisible.value = true
  isLoading.value = true
  dwcData.value = {}
  title.value = label

  makeAPIRequest(`/collection_objects/${id}/dwc`)
    .then(({ data }) => {
      dwcData.value = data
    })
    .catch(() => {})
    .finally(() => {
      isLoading.value = false
    })
}

defineExpose({
  show
})
</script>
