<template>
  <BaseDialog v-model="dialog" icon="mdi-tag-plus" title="节点GPU调度" :width="500" @reset="reset">
    <template #content>
      <BaseSubTitle title="调度定义" />
      <v-card-text class="px-2 py-0">
        <v-list dense>
          <v-list-item-group v-model="gpuSelected" active-class="primary--text" multiple>
            <v-list-item v-if="Plugins && Plugins['tke-gpu-manager']" class="mb-3" link>
              <template #default="{ active }">
                <v-list-item-action>
                  <v-checkbox hide-details :input-value="active" />
                </v-list-item-action>
                <v-list-item-icon class="mr-4">
                  <BaseLogo icon-name="tke" :width="32" />
                </v-list-item-icon>
                <v-list-item-content class="text-subtitle-1"> TKE GPU vcuda </v-list-item-content>
              </template>
            </v-list-item>
            <v-list-item v-if="Plugins && Plugins['nvidia-device-plugin']" class="mb-3" link>
              <template #default="{ active }">
                <v-list-item-action>
                  <v-checkbox hide-details :input-value="active" />
                </v-list-item-action>
                <v-list-item-icon class="mr-4">
                  <BaseLogo icon-name="nvidia" :width="32" />
                </v-list-item-icon>
                <v-list-item-content class="text-subtitle-1"> Nvidia GPU驱动 </v-list-item-content>
              </template>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-card-text>
    </template>
    <template #action>
      <v-btn class="float-right" color="primary" :loading="Circular" text @click="gpuSchedule"> 确定 </v-btn>
    </template>
  </BaseDialog>
</template>

<script>
  import { mapState } from 'vuex';

  import { patchMetadataNode } from '@/api';
  import BaseResource from '@/mixins/resource';
  import { deepCopy } from '@/utils/helpers';

  export default {
    name: 'GpuScheduleForm',
    mixins: [BaseResource],
    data: () => ({
      dialog: false,
      item: null,
      gpuSelected: [],
    }),
    computed: {
      ...mapState(['Circular', 'Plugins']),
    },
    methods: {
      open() {
        this.dialog = true;
      },
      init(item) {
        this.item = deepCopy(item);
        if (item.metadata.labels['tencent.com/vcuda'] && item.metadata.labels['tencent.com/vcuda'] === 'true') {
          this.gpuSelected = [0];
        }
        if (item.metadata.labels['nvidia.com/gpu'] && item.metadata.labels['nvidia.com/gpu'] === 'true') {
          if (this.gpuSelected.indexOf(0) > -1) {
            this.gpuSelected = [0, 1];
          } else {
            this.gpuSelected = [0];
          }
        }
      },
      async gpuSchedule() {
        if (this.Plugins['tke-gpu-manager']) {
          if (this.gpuSelected.indexOf(0) > -1) {
            this.item.metadata.labels['tencent.com/vcuda'] = 'true';
          } else {
            delete this.item.metadata.labels['tencent.com/vcuda'];
          }
        }
        if (this.Plugins['nvidia-device-plugin']) {
          if (this.Plugins['tke-gpu-manager']) {
            if (this.gpuSelected.indexOf(1) > -1) {
              this.item.metadata.labels['nvidia.com/gpu'] = 'true';
            } else {
              delete this.item.metadata.labels['nvidia.com/gpu'];
            }
          } else {
            if (this.gpuSelected.indexOf(0) > -1) {
              this.item.metadata.labels['nvidia.com/gpu'] = 'true';
            } else {
              delete this.item.metadata.labels['nvidia.com/gpu'];
            }
          }
        }
        await patchMetadataNode(this.ThisCluster, this.item.metadata.name, {
          Annotations: this.item.metadata.annotations,
          Labels: this.item.metadata.labels,
        });
        this.reset();
        this.$emit('refresh');
      },
      reset() {
        this.dialog = false;
        this.gpuSelected = [];
      },
    },
  };
</script>
