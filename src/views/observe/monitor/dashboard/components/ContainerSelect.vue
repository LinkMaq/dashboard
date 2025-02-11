<template>
  <div class="float-left">
    <v-sheet class="text-body-2 float-left text--darken-1 sheet__menu__line">
      <v-menu
        v-model="podMenu"
        bottom
        left
        max-height="300px"
        max-width="350px"
        min-width="120px"
        nudge-bottom="5px"
        offset-y
        origin="top center"
        transition="scale-transition"
      >
        <template #activator="{ on }">
          <v-btn class="primary--text font-weight-medium" color="white" dark depressed small v-on="on">
            容器组 {{ podName }}
            <v-icon v-if="podMenu" right> fas fa-angle-up </v-icon>
            <v-icon v-else right> fas fa-angle-down </v-icon>
          </v-btn>
        </template>
        <v-data-iterator hide-default-footer :items="[{ text: '容器组', values: podItems }]">
          <template #no-data>
            <v-card>
              <v-card-text> 暂无容器组 </v-card-text>
            </v-card>
          </template>
          <template #default="props">
            <v-card v-for="item in props.items" :key="item.text" min-width="100px">
              <v-list dense>
                <v-flex class="text-subtitle-2 text-center ma-2">
                  <span>容器组</span>
                </v-flex>
                <v-divider class="mx-2" />
                <v-list-item
                  v-for="(pod, index) in item.values"
                  :key="index"
                  class="text-body-2 text-start font-weight-medium mx-2"
                  link
                  :style="pod.podName === podName ? `color: #1e88e5 !important;` : ``"
                  @click="setPod(pod)"
                >
                  <v-list-item-content>
                    <span>{{ pod.podName }}</span>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-card>
          </template>
        </v-data-iterator>
      </v-menu>
    </v-sheet>
  </div>
</template>

<script>
  import { getPodList } from '@/api';
  import BaseSelect from '@/mixins/select';

  export default {
    name: 'ProjectEnvSelect',
    mixins: [BaseSelect],
    props: {
      env: {
        type: Object,
        default: () => null,
      },
    },
    data() {
      return {
        pod: undefined,
        podName: '',
        podMenu: false,

        podItems: [],
      };
    },
    watch: {
      env: {
        handler: async function (newValue) {
          if (newValue) {
            this.loadPod();
          }
        },
        deep: true,
      },
    },
    methods: {
      async loadPod() {
        const data = await getPodList(this.env?.clusterName, this.env?.namespace, {
          noprocessing: true,
          size: 1000,
        });
        if (data?.List) {
          this.podItems = data.List.map((d) => {
            return {
              podName: d.metadata.name,
              value: d.metadata.name,
            };
          });
        }
      },
      setPod(pod) {
        this.pod = pod;
        this.podName = pod.podName;
        this.$emit('filterPod', this.pod);
      },
    },
  };
</script>

<style lang="scss" scoped>
  .sheet {
    &__line {
      line-height: 40px;
    }

    &__reverse__line {
      line-height: 30px;
    }

    &__menu {
      &__line {
        line-height: 34px;
      }
    }
  }
</style>
