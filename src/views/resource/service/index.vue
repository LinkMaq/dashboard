<template>
  <v-container fluid>
    <BaseViewportHeader />
    <BaseBreadcrumb />
    <v-card>
      <v-card-title class="py-4">
        <BaseFilter
          :default="{ items: [], text: '服务名称', value: 'search' }"
          :filters="filters"
          @refresh="m_filter_list"
        />
        <NamespaceFilter />
        <v-spacer />
        <v-menu v-if="m_permisson_resourceAllow" left>
          <template #activator="{ on }">
            <v-btn icon>
              <v-icon color="primary" small v-on="on"> fas fa-ellipsis-v </v-icon>
            </v-btn>
          </template>
          <v-card>
            <v-card-text class="pa-2">
              <v-flex>
                <v-btn color="primary" text @click="addService">
                  <v-icon left>mdi-plus-box</v-icon>
                  创建服务
                </v-btn>
              </v-flex>
              <v-flex>
                <v-btn color="error" text @click="m_table_batchRemoveResource('服务', 'Service', serviceList)">
                  <v-icon left>mdi-minus-box</v-icon>
                  删除服务
                </v-btn>
              </v-flex>
            </v-card-text>
          </v-card>
        </v-menu>
      </v-card-title>
      <v-data-table
        class="mx-4"
        :headers="headers"
        hide-default-footer
        :items="items"
        :items-per-page="params.size"
        no-data-text="暂无数据"
        :page.sync="params.page"
        show-select
        @toggle-select-all="m_table_onResourceToggleSelect"
        @update:sort-by="m_table_sortBy"
        @update:sort-desc="m_table_sortDesc"
      >
        <template #[`item.data-table-select`]="{ item, index }">
          <v-checkbox
            v-model="m_table_batchResources[`${item.metadata.name}-${index}`].checked"
            color="primary"
            hide-details
            @change="m_table_onResourceChange($event, item, index)"
            @click.stop
          />
        </template>
        <template #[`item.name`]="{ item }">
          <a class="text-subtitle-2" @click="serviceDetail(item)">
            {{ item.metadata.name }}
          </a>
        </template>
        <template #[`item.namespace`]="{ item }">
          {{ item.metadata.namespace }}
        </template>
        <template #[`item.externalip`]="{ item }">
          <div>
            {{ item.spec.type !== 'clusterIP' ? item.spec.type : '' }}
          </div>
          <div class="text-caption kubegems__text">
            {{ getExternalIp(item) }}
          </div>
        </template>
        <template #[`item.clusterIP`]="{ item }">
          {{ item.spec.clusterIP !== 'None' ? item.spec.clusterIP : 'Headless' }}
        </template>
        <template #[`item.ports`]="{ item, index }">
          <BaseCollapseChips
            :id="`s_port_${index}`"
            :chips="item.services || []"
            icon="mdi-directions-fork"
            single-line
          />
        </template>
        <template #[`item.createAt`]="{ item }">
          {{ item.metadata.creationTimestamp ? $moment(item.metadata.creationTimestamp).format('lll') : '' }}
        </template>
        <template #[`item.action`]="{ item }">
          <v-flex :id="`r${item.metadata.resourceVersion}`" />
          <v-menu :attach="`#r${item.metadata.resourceVersion}`" left>
            <template #activator="{ on }">
              <v-btn icon>
                <v-icon color="primary" x-small v-on="on"> fas fa-ellipsis-v </v-icon>
              </v-btn>
            </template>
            <v-card>
              <v-card-text class="pa-2">
                <v-flex>
                  <v-btn color="primary" small text @click="updateService(item)"> 编辑 </v-btn>
                </v-flex>
                <v-flex>
                  <v-btn color="error" small text @click="removeService(item)"> 删除 </v-btn>
                </v-flex>
              </v-card-text>
            </v-card>
          </v-menu>
        </template>
      </v-data-table>
      <BasePagination
        v-if="pageCount >= 1"
        v-model="params.page"
        :page-count="pageCount"
        :size="params.size"
        @changepage="onPageIndexChange"
        @changesize="onPageSizeChange"
        @loaddata="serviceList"
      />
    </v-card>

    <AddService ref="addService" @refresh="serviceList" />
    <UpdateService ref="updateService" @refresh="serviceList" />
  </v-container>
</template>

<script>
  import { mapState } from 'vuex';

  import AddService from './components/AddService';
  import UpdateService from './components/UpdateService';
  import { deleteService, getServiceList } from '@/api';
  import BaseFilter from '@/mixins/base_filter';
  import BasePermission from '@/mixins/permission';
  import BaseResource from '@/mixins/resource';
  import BaseTable from '@/mixins/table';
  import NamespaceFilter from '@/views/resource/components/common/NamespaceFilter';

  export default {
    name: 'Service',
    components: {
      AddService,
      NamespaceFilter,
      UpdateService,
    },
    mixins: [BaseFilter, BasePermission, BaseResource, BaseTable],
    data: () => ({
      items: [],
      pageCount: 0,
      params: {
        page: 1,
        size: 10,
      },
      filters: [{ text: '服务名称', value: 'search', items: [] }],
    }),
    computed: {
      ...mapState(['JWT', 'AdminViewport']),
      headers() {
        const items = [
          { text: '服务名', value: 'name', align: 'start' },
          {
            text: 'ClusterIP',
            value: 'clusterIP',
            align: 'start',
            sortable: false,
          },
          {
            text: '服务类型(ExternalIp/LoadBalancerIp)',
            value: 'externalip',
            align: 'start',
            sortable: false,
          },
          { text: '访问信息', value: 'ports', align: 'start', sortable: false },
          { text: '创建时间', value: 'createAt', align: 'center' },
        ];
        if (this.m_permisson_resourceAllow) {
          items.push({
            text: '',
            value: 'action',
            align: 'center',
            width: 20,
            sortable: false,
          });
        }
        if (this.AdminViewport) {
          items.splice(1, 0, {
            text: '命名空间',
            value: 'namespace',
            align: 'start',
            sortable: false,
          });
        }
        return items;
      },
    },
    watch: {
      '$store.state.NamespaceFilter': {
        handler: function (namespace) {
          if (namespace && !namespace.Mounted) {
            this.params.page = 1;
            this.params.namespace = namespace.Namespace;
            this.serviceList();
          }
        },
        deep: true,
      },
      m_table_sortparam: {
        handler: function (newV, oldV) {
          if (oldV.name !== newV.name) return;
          if (oldV.desc === null) return;
          this.serviceList(true);
        },
        deep: true,
      },
    },
    mounted() {
      if (this.JWT) {
        this.$nextTick(() => {
          if (this.ThisCluster === '') {
            this.$store.commit('SET_SNACKBAR', {
              text: `请创建或选择集群`,
              color: 'warning',
            });
            return;
          }
          this.m_table_generateParams();
          this.serviceList();
        });
      }
    },
    methods: {
      async serviceList(noprocess = false) {
        const data = await getServiceList(
          this.ThisCluster,
          this.ThisNamespace,
          Object.assign(this.params, {
            noprocessing: noprocess,
            sort: this.m_table_generateResourceSortParamValue(),
          }),
        );
        data.List = data.List.map((d) => {
          const services = [];
          if (d.spec?.ports) {
            d.spec.ports.forEach((s) => {
              if (s.nodePort !== undefined) {
                services.push(`${s.port}:${s.nodePort} ｜ ${s.protocol}`);
              } else {
                services.push(`${s.port} ｜ ${s.protocol}`);
              }
            });
          }
          return { ...d, services: services };
        });
        this.items = data.List;
        this.pageCount = Math.ceil(data.Total / this.params.size);
        this.params.page = data.CurrentPage;
        this.$router.replace({ query: { ...this.$route.query, ...this.params } });
        this.m_table_generateSelectResource();
      },
      serviceDetail(item) {
        this.$router.push({
          name: 'service-detail',
          params: Object.assign(this.$route.params, {
            name: item.metadata.name,
          }),
          query: {
            namespace: item.metadata.namespace,
          },
        });
      },
      addService() {
        this.$refs.addService.open();
      },
      updateService(item) {
        this.$refs.updateService.init(item);
        this.$refs.updateService.open();
      },
      removeService(item) {
        this.$store.commit('SET_CONFIRM', {
          title: `删除服务`,
          content: {
            text: `删除服务 ${item.metadata.name}`,
            type: 'delete',
            name: item.metadata.name,
          },
          param: { item },
          doFunc: async (param) => {
            if (param.item.metadata.name.length > 0) {
              await deleteService(this.ThisCluster, param.item.metadata.namespace, param.item.metadata.name);
              this.serviceList();
            }
          },
        });
      },
      onPageSizeChange(size) {
        this.params.page = 1;
        this.params.size = size;
      },
      onPageIndexChange(page) {
        this.params.page = page;
      },
      getExternalIp(item) {
        if (item.spec.externalName) return item.spec.externalName;
        if (item.spec.externalIPs) return item.spec.externalIPs?.join(',');
        if (item.status?.loadBalancer?.ingress)
          return item.status?.loadBalancer?.ingress
            ?.map((i) => {
              return i.ip;
            })
            ?.join(',');
        if (item.spec.loadBalancerIP) return item.spec.loadBalancerIP;
        return 'None';
      },
    },
  };
</script>
