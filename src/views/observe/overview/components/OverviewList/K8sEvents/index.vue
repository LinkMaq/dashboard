<template>
  <BasePanel v-model="panel" icon="fas fa-bell" title="Kubernetes事件" @dispose="dispose">
    <template #action>
      <BaseDatetimePicker v-model="date" color="primary" :default-value="60" @change="onDatetimeChange(undefined)" />
    </template>
    <template #content>
      <v-tabs v-model="tab" class="rounded-t pa-0 v-tabs--default" fixed-tabs height="45">
        <v-tab v-for="t in tabItems" :key="t.value">
          {{ t.text }}
        </v-tab>
      </v-tabs>
      <div class="d-flex flex-column mt-0">
        <component :is="tabItems[tab].value" :data="data" :date="date" @loadData="eventList" />
      </div>
    </template>
  </BasePanel>
</template>

<script>
  import Chart from './Chart';
  import EventList from './EventList';
  import { getEventListFromLoki } from '@/api';

  export default {
    name: 'K8sEvents',
    components: {
      Chart,
      EventList,
    },
    props: {
      env: {
        type: Object,
        default: () => null,
      },
    },
    data() {
      return {
        panel: false,
        data: null,
        date: [],
        tab: 0,
        tabItems: [
          { text: '事件图表', value: 'Chart' },
          { text: '事件列表', value: 'EventList' },
        ],
      };
    },
    watch: {
      env: {
        handler(newValue) {
          if (newValue) {
            this.eventList();
          }
        },
        deep: true,
      },
    },
    methods: {
      open() {
        this.panel = true;
      },
      async eventList() {
        let query = '{container="event-exporter"} | json | __error__=``';
        query += ` | line_format "{{.metadata_namespace}}" |= "${this.env.namespace}"`;
        const data = await getEventListFromLoki(this.env.clusterName, {
          query: query,
          limit: 3000,
          start: `${this.date[0]}000000`,
          end: `${this.date[1]}000000`,
        });
        this.data = data || [];
      },
      onDatetimeChange() {
        this.eventList();
      },
      dispose() {},
    },
  };
</script>
