<template>
  <BasePanel v-model="panel" icon="fas fa-spinner" title="实时状态" :width="`50%`" @dispose="dispose">
    <template #header>
      <span class="ml-2"> {{ resource ? resource.kind : '' }}/{{ resource ? resource.name : '' }} </span>
    </template>
    <template #action>
      <v-sheet v-if="resource && resource.kind === 'Pod'" class="text-subtitle-1 mt-n1 white--text primary">
        <v-menu
          v-model="containerMenu"
          bottom
          left
          nudge-bottom="5px"
          offset-y
          origin="top center"
          transition="scale-transition"
        >
          <template #activator="{ on }">
            <v-btn class="white--text primary" color="white" dark depressed v-on="on">
              {{ container }}
              <v-icon v-if="containerMenu" right> fas fa-angle-up </v-icon>
              <v-icon v-else right> fas fa-angle-down </v-icon>
            </v-btn>
          </template>
          <v-data-iterator hide-default-footer :items="[{ text: '容器', values: containers }]">
            <template #no-data>
              <v-card>
                <v-card-text> 暂无容器 </v-card-text>
              </v-card>
            </template>
            <template #default="props">
              <v-card v-for="item in props.items" :key="item.text" min-width="120">
                <v-list dense>
                  <v-flex class="text-subtitle-2 text-center ma-2">
                    <span>容器</span>
                  </v-flex>
                  <v-divider class="mx-2" />
                  <v-list-item
                    v-for="(con, index) in item.values"
                    :key="index"
                    class="text-body-2 text-center mx-2"
                    link
                    :style="con.text === container ? `color: #1e88e5 !important;` : ``"
                    @click="setContainer(con)"
                  >
                    <v-list-item-content>
                      <span class="font-weight-medium">{{ con.text }}</span>
                    </v-list-item-content>
                  </v-list-item>
                </v-list>
              </v-card>
            </template>
          </v-data-iterator>
        </v-menu>
      </v-sheet>
    </template>
    <template #content>
      <v-card-text class="ma-0 pa-0">
        <v-tabs v-model="tab" class="rounded-t pa-0 v-tabs--default" fixed-tabs height="45">
          <v-tab v-for="item in tabItems" :key="item.value">
            {{ item.text }}
          </v-tab>
        </v-tabs>

        <component :is="tabItems[tab].value" :ref="tabItems[tab].value" :container="container" :resource="resource" />
      </v-card-text>
    </template>
  </BasePanel>
</template>

<script>
  import { mapGetters, mapState } from 'vuex';

  import DeployDiffYaml from './DeployDiffYaml';
  import DeployEvent from './DeployEvent';
  import DeployLiveYaml from './DeployLiveYaml';
  import DeployLog from './DeployLog';
  import DeployResult from './DeployResult';
  import { getPodDetail } from '@/api';
  import BaseResource from '@/mixins/resource';
  import { deepCopy } from '@/utils/helpers';

  export default {
    name: 'DeployLive',
    components: {
      DeployDiffYaml,
      DeployEvent,
      DeployLiveYaml,
      DeployLog,
      DeployResult,
    },
    mixins: [BaseResource],
    data: () => ({
      panel: false,
      containerMenu: false,
      resource: null,
      items: [],
      tab: 0,
      container: '',
      containers: [],
    }),
    computed: {
      ...mapState(['Scale']),
      ...mapGetters(['Tenant', 'Project']),
      tabItems() {
        if (this.resource && this.resource.kind === 'Pod') {
          return [
            { text: '资源Live', value: 'DeployLiveYaml' },
            { text: '资源Diff', value: 'DeployDiffYaml' },
            { text: '事件', value: 'DeployEvent' },
            { text: '日志', value: 'DeployLog' },
          ];
        } else if (this.resource && this.resource.kind === 'Application') {
          return [
            { text: '资源Live', value: 'DeployLiveYaml' },
            { text: '应用状态', value: 'DeployResult' },
          ];
        } else {
          return [
            { text: '资源Live', value: 'DeployLiveYaml' },
            { text: '资源Diff', value: 'DeployDiffYaml' },
            { text: '事件', value: 'DeployEvent' },
          ];
        }
      },
    },
    methods: {
      open() {
        this.panel = true;
      },
      init(resource) {
        this.resource = deepCopy(resource);
        if (this.resource && this.resource.kind === 'Pod') {
          this.podDetail();
        }
      },
      async podDetail() {
        const data = await getPodDetail(this.ThisCluster, this.resource.namespace, this.resource.name);
        this.containers = [];
        data.spec.containers.forEach((container) => {
          this.containers.push({ text: container.name, value: container.name });
        });
        if (data.spec.containers && data.spec.containers.length > 0) {
          this.container = data.spec.containers[0].name;
        }
      },
      setContainer(con) {
        if (this.container !== con.value) {
          this.container = con.value;
          this.$nextTick(() => {
            this.$refs[this.tabItems[this.tab].value].refresh();
          });
        }
      },
      dispose() {
        this.items = [];
        this.tab = 0;
      },
    },
  };
</script>
