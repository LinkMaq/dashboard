<template>
  <v-flex class="my-1">
    <v-menu
      bottom
      :close-delay="200"
      :close-on-content-click="false"
      max-width="200px"
      offset-y
      open-on-hover
      origin="top center"
      transition="scale-transition"
    >
      <template #activator="{ on }">
        <span class="kubegems__pointer" v-on="on">
          <BaseLogo :icon-name="type" />
        </span>
      </template>
      <v-card flat>
        <v-flex class="text-body-2 text-center primary white--text py-2 font-weight-medium">
          <v-icon color="white" left small> mdi-memory </v-icon>
          <span>{{ type === 'tke' ? 'Tencent Vcuda' : 'Nvidia Gpu' }}</span>
        </v-flex>
        <v-list class="pa-0 kubegems__tip" dense>
          <v-list-item>
            <v-list-item-content>
              <template v-if="type === 'tke'">
                <v-list-item class="float-left pa-0" two-line>
                  <v-list-item-content class="py-0">
                    <v-list-item-title> Gpu </v-list-item-title>
                    <v-list-item-content class="text-caption kubegems__text">
                      {{ parseInt(item.TkeGpu || 0) / 100 }} Gpu
                      <div v-if="allocated"
                        >已分配：{{ parseInt(item.AllocatedTkeGpu || 0) / 100 }} Gpu ({{ item.TkeGpuPercentage }}%)</div
                      >
                    </v-list-item-content>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item class="float-left pa-0" two-line>
                  <v-list-item-content class="py-0">
                    <v-list-item-title> 显存 </v-list-item-title>
                    <v-list-item-content class="text-caption kubegems__text">
                      {{ parseInt(item.TkeMemory || 0) / 100 }} Gi
                      <div v-if="allocated"
                        >已分配：{{ (parseInt(item.AllocatedTkeMemory || 0) * 256) / 1024 }} Gi ({{
                          item.TkeMemoryPercentage
                        }}%)</div
                      >
                    </v-list-item-content>
                  </v-list-item-content>
                </v-list-item>
              </template>
              <template v-if="type === 'nvidia'">
                <v-list-item class="float-left pa-0" two-line>
                  <v-list-item-content class="py-0">
                    <v-list-item-title> Gpu </v-list-item-title>
                    <v-list-item-content class="text-caption kubegems__text">
                      {{ parseInt(item.NvidiaGpu || 0) }} Gpu
                      <div v-if="allocated"
                        >已分配：{{ parseInt(item.AllocatedNvidiaGpu || 0) / 100 }} Gpu ({{
                          item.NvidiaGpuPercentage
                        }}%)</div
                      >
                    </v-list-item-content>
                  </v-list-item-content>
                </v-list-item>
              </template>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-card>
    </v-menu>
  </v-flex>
</template>

<script>
  export default {
    name: 'GpuTip',
    props: {
      allocated: {
        type: Boolean,
        default: () => true,
      },
      item: {
        type: Object,
        default: () => {},
      },
      type: {
        type: String,
        default: () => 'tke',
      },
    },
  };
</script>
