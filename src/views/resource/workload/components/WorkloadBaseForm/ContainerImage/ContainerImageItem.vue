<template>
  <v-flex>
    <v-sheet v-for="(item, index) in containers" :key="`container${index}`" class="grey lighten-4 rounded mb-3">
      <v-list-item two-line>
        <v-list-item-content class="py-2">
          <v-list-item-subtitle class="text-subtitle-2 py-1 primary--text">
            {{ item.name }}(工作容器)
          </v-list-item-subtitle>
          <v-list-item-subtitle class="text-body-2 py-0">
            <v-list-item class="float-left pa-0" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1">
                  {{ item.image }}
                </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 镜像 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item v-if="item.resources" class="float-right pa-0" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1">
                  <span class="mx-1">
                    CPU:
                    <span v-if="item.resources.requests && item.resources.requests.cpu">
                      {{ item.resources.requests.cpu }}
                    </span>
                    <span v-else>∞</span>
                    ~
                    <span v-if="item.resources && item.resources.limits && item.resources.limits.cpu">
                      {{ item.resources.limits.cpu }}
                    </span>
                    <span v-else>∞</span>
                  </span>
                  <span class="mx-1">
                    内存:
                    <span v-if="item.resources && item.resources.requests && item.resources.requests.memory">
                      {{ item.resources.requests.memory }}
                    </span>
                    <span v-else>∞</span>
                    ~
                    <span v-if="item.resources.limits && item.resources.limits.memory">
                      {{ item.resources.limits.memory }}
                    </span>
                    <span v-else>∞</span>
                  </span>
                </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 资源 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item v-else class="float-right pa-0" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1"> 无限制 </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 资源 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-subtitle>
          <div class="kubegems__clear-float" />
        </v-list-item-content>
        <v-btn color="primary" dark fab right text x-small @click="updateData(index, 'worker')">
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
        <v-btn color="error" dark fab right text x-small @click="removeData(index, 'worker')">
          <v-icon>mdi-delete</v-icon>
        </v-btn>
      </v-list-item>
    </v-sheet>
    <v-sheet v-for="(item, index) in initContainers" :key="`initcontainer${index}`" class="grey lighten-4 rounded mb-3">
      <v-list-item two-line>
        <v-list-item-content class="py-2">
          <v-list-item-subtitle class="text-subtitle-2 py-1 primary--text">
            {{ item.name }}(初始化容器)
          </v-list-item-subtitle>
          <v-list-item-subtitle class="text-body-2 py-0">
            <v-list-item class="float-left pa-0" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1">
                  {{ item.image }}
                </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 镜像 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item v-if="item.resources" class="float-right pa-0" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1">
                  <span class="mx-1">
                    CPU:
                    <span v-if="item.resources.requests && item.resources.requests.cpu">
                      {{ item.resources.requests.cpu }}
                    </span>
                    <span v-else>∞</span>
                    ~
                    <span v-if="item.resources && item.resources.limits && item.resources.limits.cpu">
                      {{ item.resources.limits.cpu }}
                    </span>
                    <span v-else>∞</span>
                  </span>
                  <span class="mx-1">
                    内存:
                    <span v-if="item.resources && item.resources.requests && item.resources.requests.memory">
                      {{ item.resources.requests.memory }}
                    </span>
                    <span v-else>∞</span>
                    ~
                    <span v-if="item.resources.limits && item.resources.limits.memory">
                      {{ item.resources.limits.memory }}
                    </span>
                    <span v-else>∞</span>
                  </span>
                </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 资源 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-list-item v-else class="float-right pa-0" two-line>
              <v-list-item-content class="py-0">
                <v-list-item-title class="text-subtitle-2 py-1"> 无限制 </v-list-item-title>
                <v-list-item-subtitle class="text-body-2 py-1"> 资源 </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-subtitle>
          <div class="kubegems__clear-float" />
        </v-list-item-content>
        <v-btn color="primary" dark fab right text x-small @click="updateData(index, 'init')">
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
        <v-btn color="error" dark fab right text x-small @click="removeData(index, 'init')">
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
              添加容器镜像
            </v-btn>
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-flex>
  </v-flex>
</template>

<script>
  export default {
    name: 'ContainerImageItem',
    props: {
      containers: {
        type: Array,
        default: () => [],
      },
      initContainers: {
        type: Array,
        default: () => [],
      },
    },
    methods: {
      updateData(index, type) {
        this.$emit('updateData', index, type);
      },
      removeData(index, type) {
        this.$emit('removeData', index, type);
      },
      expandCard() {
        this.$emit('expandCard', false);
      },
    },
  };
</script>
