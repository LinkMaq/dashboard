<template>
  <v-flex>
    <v-sheet v-for="(item, index) in templates" :key="index" class="grey lighten-4 rounded mb-3">
      <v-list-item two-line>
        <v-list-item-content class="py-0">
          <v-list-item-subtitle class="text-subtitle-2 py-1 primary--text">
            <v-flex>
              <v-list-item class="pl-0 list-item-height" two-line>
                <v-list-item-title class="text-subtitle-1 py-1 primary--text font-weight-medium">
                  <v-menu :close-delay="200" open-on-hover top>
                    <template #activator="{ on }">
                      <span v-on="on">{{ item.metadata.name }}</span>
                    </template>
                    <v-card>
                      <v-card-text class="pa-2">
                        {{ item.metadata.name }}
                      </v-card-text>
                    </v-card>
                  </v-menu>
                </v-list-item-title>
              </v-list-item>
            </v-flex>
            <v-flex class="float-left">
              <v-list-item class="float-left px-0" style="width: 200px" two-line>
                <v-list-item-content class="py-0">
                  <v-list-item-title class="text-subtitle-2 py-1">
                    {{ item.spec.storageClassName }}&nbsp;
                  </v-list-item-title>
                  <v-list-item-subtitle class="text-body-2 py-1"> 类型 </v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>
              <v-list-item class="float-left" style="width: 200px" two-line>
                <v-list-item-content class="py-0">
                  <v-list-item-title class="text-subtitle-2 py-1">
                    {{ item.spec.resources.requests.storage }}&nbsp;
                  </v-list-item-title>
                  <v-list-item-subtitle class="text-body-2 py-1"> 容量 </v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>
              <v-list-item class="float-left px-0" style="width: 200px" two-line>
                <v-list-item-content class="py-0">
                  <v-list-item-title class="text-subtitle-2 py-1">
                    {{ item.spec.accessModes[0] }}&nbsp;
                  </v-list-item-title>
                  <v-list-item-subtitle class="text-body-2 py-1"> 访问模式 </v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>
            </v-flex>
            <div class="kubegems__clear-float" />
          </v-list-item-subtitle>
          <v-list-item-content v-for="(container, i) in containers" :key="`containerT${i}`" class="pb-4">
            <v-row v-if="container.volumeMounts && containerMap[container.name][item.metadata.name]">
              <v-col class="py-1" cols="2">
                <span class="text-body-2">容器:</span>
                <span class="text-subtitle-2 ml-2">
                  {{ container.name }}
                </span>
              </v-col>
              <v-col class="py-1" cols="5">
                <span class="text-body-2">挂载:</span>
                <span class="text-subtitle-2 ml-2">
                  {{ containerMap[container.name][item.metadata.name].mountPath }}
                </span>
                <span class="text-subtitle-2 ml-2">
                  ({{ containerMap[container.name][item.metadata.name].readOnly ? '只读' : '读写' }})
                </span>
              </v-col>
              <v-col v-if="containerMap[container.name][item.metadata.name].subPath" class="py-1" cols="4">
                <span class="text-body-2">子路径:</span>
                <span class="text-subtitle-2 ml-2">
                  {{ containerMap[container.name][item.metadata.name].subPath }}
                </span>
              </v-col>
            </v-row>
            <v-row v-else>
              <v-col class="py-1" cols="2">
                <span class="text-body-2">容器:</span>
                <span class="text-subtitle-2 ml-2">
                  {{ container.name }}
                </span>
              </v-col>
              <v-col class="py-1" cols="5">
                <span class="text-body-2">挂载:</span>
                <span class="text-subtitle-2 ml-2">不挂载</span>
              </v-col>
            </v-row>
          </v-list-item-content>
        </v-list-item-content>
        <v-btn color="primary" dark fab right text x-small @click="updateVolumeTemplateData(index)">
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
        <v-btn color="error" dark fab right text x-small @click="removeVolumeTemplateData(index)">
          <v-icon>mdi-delete</v-icon>
        </v-btn>
      </v-list-item>
    </v-sheet>
    <v-flex class="grey lighten-4 rounded">
      <v-list-item two-line>
        <v-list-item-content class="py-2">
          <v-list-item-subtitle class="text-body-2 py-0 text-center">
            <v-btn color="primary" text @click="expandTemplateCard">
              <v-icon left small> mdi-plus </v-icon>
              添加卷挂载模版
            </v-btn>
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-flex>
  </v-flex>
</template>

<script>
  import BaseResource from '@/mixins/resource';

  export default {
    name: 'VolumeClaimTemplateItem',
    mixins: [BaseResource],
    props: {
      containers: {
        type: Array,
        default: () => [],
      },
      templates: {
        type: Array,
        default: () => [],
      },
    },
    data() {
      return {
        containerMap: {},
        initContainerMap: {},
      };
    },
    watch: {
      containers() {
        this.containers.forEach((c) => {
          this.containerMap[c.name] = {};
          if (c.volumeMounts) {
            c.volumeMounts.forEach((v) => {
              this.containerMap[c.name][v.name] = v;
            });
          }
        });
      },
      initContainers() {
        this.initContainers.forEach((c) => {
          this.initContainerMap[c.name] = {};
          if (c.volumeMounts) {
            c.volumeMounts.forEach((v) => {
              this.initContainerMap[c.name][v.name] = v;
            });
          }
        });
      },
    },
    methods: {
      updateVolumeTemplateData(index) {
        this.$emit('updateVolumeTemplateData', index);
      },
      removeVolumeTemplateData(index) {
        this.$emit('removeVolumeTemplateData', index);
      },
      expandTemplateCard() {
        this.$emit('expandTemplateCard');
      },
    },
  };
</script>

<style lang="scss" scoped>
  .list-item-height {
    min-height: 20px !important;
  }
</style>
