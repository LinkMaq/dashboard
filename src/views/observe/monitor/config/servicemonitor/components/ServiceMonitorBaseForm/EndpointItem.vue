<template>
  <v-flex>
    <v-sheet v-for="(item, index) in endpoints" :key="index" class="grey lighten-4 rounded mb-3">
      <v-list-item two-line>
        <v-list-item-content class="py-2">
          <v-list-item-subtitle class="text-body-2 py-0">
            <v-list-item v-if="item.port" class="float-left py-0 pl-0" style="width: 220px" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1 primary--text"> 端口名 </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1">
                  {{ item.port ? item.port : '-' }}
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item v-if="item.targetPort" class="float-left py-0 pl-0" style="width: 220px" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1 primary--text"> 端口号 </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1">
                  {{ item.targetPort ? item.targetPort : '-' }}
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item class="float-left py-0 pl-0" style="width: 220px" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1 primary--text"> 路径 </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1">
                  {{ item.path ? item.path : '-' }}
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item class="float-left py-0 pl-0" style="width: 220px" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1 primary--text"> 拉取间隔 </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1">
                  {{ item.interval ? item.interval : '-' }}
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item class="float-left py-0 pl-0" style="width: 150px" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1 primary--text"> 超时时间 </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1">
                  {{ item.scrapeTimeout ? item.scrapeTimeout : '-' }}
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-subtitle>
          <div class="kubegems__clear-float" />
        </v-list-item-content>
        <v-btn color="primary" dark fab right text x-small @click="updateEndpoint(index)">
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
        <v-btn color="error" dark fab right text x-small @click="removeEndpoint(index)">
          <v-icon>mdi-delete</v-icon>
        </v-btn>
      </v-list-item>
    </v-sheet>
    <v-flex class="grey lighten-4 rounded">
      <v-list-item two-line>
        <v-list-item-content class="py-2">
          <v-list-item-subtitle class="text-body-2 py-0 text-center">
            <v-btn color="primary" text @click="expandCard('EndpointForm')">
              <v-icon left small> mdi-plus </v-icon>
              添加端点
            </v-btn>
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-flex>
  </v-flex>
</template>

<script>
  export default {
    name: 'EndpointItem',
    props: {
      endpoints: {
        type: Array,
        default: () => [],
      },
    },
    methods: {
      updateEndpoint(index) {
        this.$emit('updateEndpoint', index);
      },
      removeEndpoint(index) {
        this.$emit('removeEndpoint', index);
      },
      expandCard() {
        this.$emit('expandCard', 'endpointForm');
      },
    },
  };
</script>
