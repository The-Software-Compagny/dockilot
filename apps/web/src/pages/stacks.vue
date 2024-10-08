<template lang="pug">
q-page.flex.column
  q-custom-twopan.col(
    :simple='false'
    :loading='pending'
    :rows='stacks.data'
    :total='stacks.total'
    :columns='columns'
    :targetId='targetId'
    row-key='Name'
  )
    template(#top-table)
      q-card-section.q-pa-none
        q-input(v-model='filters' label='Search' fill dense)
    template(v-slot:body-cell-state='props')
      q-td(:props='props')
        q-icon(name='mdi-layers-outline' size='sm' :color='getStackStateColor(props.row)')
    template(v-slot:row-actions='{ row }')
      q-btn(:to='toPathWithQueries(`/stacks/${row.name}`)' color='primary' icon='mdi-eye' size='md' flat round dense)
    template(#after-content)
      nuxt-page(ref='page')
</template>

<script lang="ts">
import type { QTableProps } from 'quasar'
import Page from './stacks/[name].vue'
import type { LocationQueryValue } from 'vue-router'

export default {
  name: 'ContainersPage',
  data() {
    return {
      columns: [
        {
          name: 'state',
          required: true,
          align: 'center',
          label: 'State',
          field: 'state',
          sortable: true,
        },
        {
          name: 'Name',
          required: true,
          align: 'left',
          label: 'Name',
          field: (row) => row.name,
          sortable: true,
        },
      ] as QTableProps['columns'],
    }
  },
  async setup() {
    const page = ref<typeof Page | null>(null)
    const $route = useRoute()
    const $appConfig = useAppConfig()
    const { getDefaults } = usePagination()
    const { toPathWithQueries } = useRouteQueries()

    const computedQuery = computed(() => {
      return {
        ...getDefaults(),
        ...$route.query,
      }
    })
    const queryDebounced = refDebounced(computedQuery, 700)

    const {
      data: stacks,
      error,
      pending,
      refresh,
    } = await useHttp('/docker/stacks', {
      method: 'get',
      query: queryDebounced,
    })
    if (error.value) {
      console.error(error.value)
      throw showError({
        statusCode: 500,
        statusMessage: 'Internal Server Error',
      })
    }

    return {
      page,
      stacks,
      pending,
      toPathWithQueries,
    }
  },
  computed: {
    targetId(): LocationQueryValue[] | string {
      return this.$route.params.name || ''
    },
    filters: {
      get(): LocationQueryValue[] | string {
        return this.$route.query['filters[name]'] || ''
      },
      set(v: string): void {
        this.$router.replace({
          query: {
            ...this.$route.query,
            'filters[name]': v ? `${v}` : undefined,
          },
        })
      },
    },
  },
  methods: {
    getStackStateColor(row) {
      const states = row.containers.map((container) => container.State)
      const uniqueStates = [...new Set(states)]

      if (uniqueStates.length === 1) {
        switch (true) {
          case uniqueStates.includes('running'):
            return 'positive'

          case uniqueStates.includes('exited'):
            return 'negative'

          case uniqueStates.includes('paused'):
            return 'warning'

          default:
            return 'grey'
        }
      }

      return 'warning'
    },
  },
}
</script>
