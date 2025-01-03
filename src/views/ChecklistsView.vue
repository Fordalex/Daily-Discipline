<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue'
import { useAccessTokenStore } from '@/stores/accessTokenStore'
import PageHeader from '@/components/shared/PageHeader.vue'
import ChecklistForm from '@/components/checklists/ChecklistForm.vue';

export default defineComponent({
  name: 'ChecklistsView',
  components: {
    PageHeader,
    ChecklistForm,
  },
  setup() {
    const checklists = ref<Array<any>>([])
    const errorMessage = ref<string | null>(null)
    const loading = ref(true)

    const fetchChecklists = async () => {
      const accessTokenStore = useAccessTokenStore()
      const apiUrl = `${import.meta.env.VITE_API_URL}checklists`
      const response = await fetch(apiUrl, {
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${accessTokenStore.accessToken}`,
        },
      })

      if (response.ok) {
        loading.value = false
        const responseBody = await response.json()
        checklists.value = responseBody
        console.log(checklists.value)
      } else {
        loading.value = false
        const responseBody = await response.json()
        errorMessage.value = responseBody.error

        // Not ideal but it works for now
        if (errorMessage.value === 'Access token has expired or is invalid.') {
          accessTokenStore.$reset()
          accessTokenStore.clearState()
          localStorage.clear()
          sessionStorage.clear()
        }
      }
    }

    onMounted(() => {
      fetchChecklists()
    })

    return {
      checklists,
    }
  },
})
</script>

<template>
  <div class="page-header">
    <PageHeader title="Checklists" />
  </div>

  <v-list>
    <v-list-item
      v-for="checklist in checklists"
      :key="checklist.id"
      :to="`/checklists/${checklist.id}`"
    >
      <v-list-item-title>{{ checklist.name }}</v-list-item-title>
    </v-list-item>
  </v-list>

  <ChecklistForm>
    <template #trigger="{ openDialog }">
      <v-btn
        density="comfortable"
        variant="tonal"
        text="New Checklist"
        class="mr-2"
        @click="openDialog"
      ></v-btn>
    </template>
  </ChecklistForm>
</template>
