<template>
  <div>
    <v-card v-if="!pass" class="mx-0 mt-6" flat :height="`400px`">
      <v-row :style="{ height: `400px` }">
        <v-col class="d-flex align-center justify-center">
          <div class="d-flex align-center pa-10">
            <div class="text-center">
              <h2 class="text-h5 primary--text font-weight-medium">
                该应用环境所在集群暂时还未启用 {{ missingPlugins.join(', ') }} 插件！
              </h2>
              <h6 class="text-subtitle-1 mt-4 primary--text op-5 font-weight-regular">
                您可以联系平台管理员启用该插件
              </h6>
            </div>
          </div>
        </v-col>
      </v-row>
    </v-card>
    <slot v-else />
  </div>
</template>

<script>
  import { getClusterPluginsList } from '@/api';

  export default {
    name: 'PluginPass',
    data() {
      return {
        missingPlugins: [],
        pass: true,
      };
    },
    watch: {
      value: {
        handler(newValue) {
          this.pass = newValue;
        },
        deep: true,
      },
      '$store.state.EnvironmentFilter': {
        handler: function (env) {
          if (env) {
            this.pluginsPass(env.cluster);
          } else {
            if (this.$route.query.cluster) {
              this.pluginsPass(this.$route.query.cluster);
            }
          }
        },
        deep: true,
        immediate: true,
      },
    },
    methods: {
      async pluginsPass(cluster) {
        if (!cluster) {
          cluster = this.$route.query.cluster;
        }
        const data = await getClusterPluginsList(cluster, { simple: true, noprocessing: true });
        const plugins = this.$route.meta?.dependencies || [];
        this.missingPlugins = plugins.filter((p) => {
          return !data[p];
        });
        this.pass = this.missingPlugins?.length === 0;
        this.$emit('change', this.pass);
        this.$emit('input', this.pass);
      },
    },
  };
</script>
