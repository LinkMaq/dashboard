<template>
  <v-flex class="float-left">
    <v-list-item class="float-left px-0" style="width: 296px" two-line>
      <v-list-item-content class="py-0">
        <v-list-item-title class="text-subtitle-2 py-1"> hostPath </v-list-item-title>
        <v-list-item-subtitle class="text-body-2 py-1"> 类型 </v-list-item-subtitle>
      </v-list-item-content>
    </v-list-item>
    <v-list-item class="float-left px-0" style="width: 300px" two-line>
      <v-list-item-content class="py-0">
        <v-list-item-title class="text-subtitle-2 py-1"> {{ item.hostPath.path }}&nbsp; </v-list-item-title>
        <v-list-item-subtitle class="text-body-2 py-1"> 路径 </v-list-item-subtitle>
      </v-list-item-content>
    </v-list-item>
  </v-flex>
</template>

<script>
  export default {
    name: 'HostPath',
    props: {
      item: {
        type: Object,
        default: () => {},
      },
    },
  };
</script>
