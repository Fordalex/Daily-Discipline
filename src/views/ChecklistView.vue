<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue'
import { useAccessTokenStore } from '@/stores/accessTokenStore'
import PageHeader from '@/components/shared/PageHeader.vue'
import ChecklistForm from '@/components/checklists/ChecklistForm.vue';
import { useRoute } from 'vue-router'

export interface Checklist {
  id?: number
  name: string
}

export default defineComponent({
  name: 'ChecklistView',
  components: {
    PageHeader,
    ChecklistForm,
  },
  setup() {
    const checklist = ref<Checklist>({
      name: '',
    })
    const errorMessage = ref<string | null>(null)
    const loading = ref(true)
    const breadcrumbs = ref([
      {
        title: 'Checklists',
        disabled: false,
        href: '/daily-discipline/#/checklists',
      }
    ])

    const fetchChecklists = async () => {
      const accessTokenStore = useAccessTokenStore()
      const checklistId = useRoute().params.id
      const apiUrl = `${import.meta.env.VITE_API_URL}checklists/${checklistId}`
      const response = await fetch(apiUrl, {
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${accessTokenStore.accessToken}`,
        },
      })

      if (response.ok) {
        loading.value = false
        const responseBody = await response.json()
        checklist.value = responseBody
        breadcrumbs.value.push({
          title: responseBody.name,
          disabled: true,
          href: `/checklists/${checklistId}`,
        })
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
      checklist,
      breadcrumbs
    }
  },
})
</script>

<template>
  <div class="page-header">
    <PageHeader :title="checklist.name" />
    <v-breadcrumbs :items="breadcrumbs">
      <template v-slot:prepend>
        <v-icon icon="$vuetify" size="small"></v-icon>
      </template>
    </v-breadcrumbs>
  </div>
</template>

