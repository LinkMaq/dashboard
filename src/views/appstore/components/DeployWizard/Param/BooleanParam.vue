<template>
  <v-flex>
    <v-switch
      :id="id"
      class="ma-2 float-left"
      color="primary"
      dense
      hide-details
      :input-value="param.value === true ? true : false"
      :label="pathLevel === 1 ? label : label"
      @change="onChange($event)"
      @click="click"
    />
    <div class="kubegems__clear-float" />
  </v-flex>
</template>

<script>
  export default {
    name: 'BooleanParam',
    props: {
      id: {
        type: String,
        default: () => '',
      },
      label: {
        type: String,
        default: () => '',
      },
      param: {
        type: Object,
        default: () => {},
      },
    },
    data: () => ({
      myValue: false,
    }),
    computed: {
      pathLevel() {
        return this.param.path.split('/').length;
      },
    },
    methods: {
      click() {
        // 先监听change事件,设置布尔值状态值, 同时监听到点击事件,向上级发送数据, 注意input-value是v-model绑定的初始值
        this.$emit('changeBasicFormParam', this.param, this.myValue);
      },
      onChange(event) {
        this.myValue = event === true;
      },
    },
  };
</script>
