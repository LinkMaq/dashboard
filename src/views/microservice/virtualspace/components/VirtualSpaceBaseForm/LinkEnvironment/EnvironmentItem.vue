<template>
  <v-flex>
    <v-sheet v-for="(item, index) in environments" :key="index" class="grey lighten-4 rounded mb-3">
      <v-list-item two-line>
        <v-list-item-content class="py-2">
          <v-list-item-subtitle class="text-body-2 py-0">
            <v-list-item class="float-left py-0 pl-0" style="width: 260px" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1">
                  {{ item.EnvironmentName }}
                </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 环境 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item class="float-left py-0" style="width: 260px" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1">
                  {{ item.clusterName || item.Cluster.ClusterName }}
                </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 集群 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item class="float-left py-0" style="width: 260px" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1">
                  {{ item.type || item.MetaType }}
                </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 类型 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-subtitle>
          <div class="kubegems__clear-float" />
        </v-list-item-content>
        <v-btn color="primary" dark fab right text x-small @click="updateEnvironment(index)">
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
        <v-btn color="error" dark fab right text x-small @click="removeEnvironment(index)">
          <v-icon>mdi-delete</v-icon>
        </v-btn>
      </v-list-item>
    </v-sheet>
    <v-flex class="grey lighten-4 rounded">
      <v-list-item two-line>
        <v-list-item-content class="py-2">
          <v-list-item-subtitle class="text-body-2 py-0 text-center">
            <v-btn color="primary" text @click="expandCard">
              <v-icon left small> mdi-plus </v-icon>
              添加环境
            </v-btn>
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-flex>
  </v-flex>
</template>

<script>
  export default {
    name: 'EnvironmentItem',
    props: {
      environments: {
        type: Array,
        default: () => [],
      },
    },
    methods: {
      updateEnvironment(index, type) {
        this.$emit('updateEnvironment', index, type);
      },
      removeEnvironment(index, type) {
        this.$emit('removeEnvironment', index, type);
      },
      expandCard() {
        this.$emit('expandCard', false);
      },
    },
  };
</script>
