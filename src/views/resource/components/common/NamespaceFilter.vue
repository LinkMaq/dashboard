<template>
  <v-sheet>
    <v-sheet v-if="AdminViewport">
      <v-sheet class="text-subtitle-2 ml-4 mr-2 float-left font-weight-medium kubegems__text font-line-height ns__tip">
        命名空间
      </v-sheet>
      <v-sheet class="float-left ns__combox" width="400">
        <v-combobox
          v-model="namespaceFilter"
          chips
          color="primary"
          dense
          flat
          full-width
          hide-details
          hide-selected
          :items="m_select_namespaceItems"
          label="命名空间"
          no-data-text="无数据"
          prepend-inner-icon="mdi-cube"
          solo
          @change="onNamespaceFilterChange"
          @focus="onNamespaceSelectFocus(Cluster().ClusterName)"
        >
          <template #selection="{ attrs, item, selected }">
            <v-chip v-if="namespaceFilter" color="primary" :input-value="selected" label small v-bind="attrs">
              <span class="pr-2">{{ item.text }}</span>
              <v-icon small @click="removeNamespaceFilter"> mdi-close </v-icon>
            </v-chip>
          </template>
        </v-combobox>
      </v-sheet>
      <div class="kubegems__clear-float" />
    </v-sheet>
    <div class="kubegems__clear-float" />
  </v-sheet>
</template>

<script>
  import { mapGetters, mapState } from 'vuex';

  import BaseSelect from '@/mixins/select';

  export default {
    name: 'NamespaceFilter',
    mixins: [BaseSelect],
    inject: ['reload'],
    data() {
      return {
        namespaceFilter: null,
      };
    },
    computed: {
      ...mapState(['NamespaceFilter', 'AdminViewport']),
      ...mapGetters(['Cluster', 'Environment']),
    },
    mounted() {
      if (this.NamespaceFilter) {
        this.$router.replace({
          query: {
            ...this.$route.query,
            ...{ namespace: this.NamespaceFilter.Namespace, page: 1 },
          },
        });
      }
      if (this.$route.query.namespace) {
        this.$store.commit('SET_NAMESPACE_FILTER', {
          Namespace: this.$route.query.namespace,
          Mounted: true,
        });
      }
      if (this.NamespaceFilter) {
        this.namespaceFilter = {
          text: this.NamespaceFilter.Namespace,
          value: this.NamespaceFilter.Namespace,
        };
      }
    },
    methods: {
      onNamespaceFilterChange() {
        const namespace = this.m_select_namespaceItems.find((n) => {
          return this.namespaceFilter && n.value === this.namespaceFilter.value;
        });
        if (namespace) {
          this.$router.replace({
            query: {
              ...this.$route.query,
              ...{ namespace: namespace.text },
              ...{ page: 1 },
            },
          });
          this.$store.commit('SET_NAMESPACE_FILTER', {
            Namespace: namespace.text,
            Mounted: false,
          });
        } else {
          this.namespaceFilter = null;
          this.removeNamespaceFilter();
        }
      },
      removeNamespaceFilter() {
        this.$store.commit('SET_NAMESPACE_FILTER', null);
        this.namespaceFilter = '';
        this.$router.replace({
          query: {
            ...this.$route.query,
            ...{ namespace: null },
          },
        });
        this.reload();
      },
      onNamespaceSelectFocus(clusterName) {
        this.m_select_namespaceSelectData(clusterName, { noprocessing: true });
      },
    },
  };
</script>

<style lang="scss" scoped>
  .font-line-height {
    line-height: 40px;
  }

  .ns__tip {
    display: block;

    @media (max-width: 1300px) {
      display: none;
    }
  }

  .ns__combox {
    @media (max-width: 1300px) {
      margin-top: 4px;
      width: 700px;
    }
  }
</style>
