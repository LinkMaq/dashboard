<template>
  <BaseDialog v-model="dialog" icon="mdi-link" title="关联应用" :width="500" @reset="reset">
    <template #content>
      <component :is="formComponent" :ref="formComponent" />
    </template>
    <template #action>
      <v-btn class="float-right" color="primary" :loading="Circular" text @click="addApp"> 确定 </v-btn>
    </template>
  </BaseDialog>
</template>

<script>
  import { mapGetters, mapState } from 'vuex';

  import LinkAppBaseForm from './LinkAppBaseForm';
  import { postAddApp } from '@/api';

  export default {
    name: 'LinkApp',
    components: {
      LinkAppBaseForm,
    },
    data: () => ({
      dialog: false,
      formComponent: 'LinkAppBaseForm',
    }),
    computed: {
      ...mapState(['Circular']),
      ...mapGetters(['Tenant', 'Environment', 'Project']),
    },
    methods: {
      open() {
        this.dialog = true;
      },
      async addApp() {
        if (this.$refs[this.formComponent].validate()) {
          const data = this.$refs[this.formComponent].getData();
          if (this.Environment().ID === 0) {
            this.$store.commit('SET_SNACKBAR', {
              text: '请选择环境',
              color: 'warning',
            });
            return;
          }
          await postAddApp(this.Tenant().ID, this.Project().ID, this.Environment().ID, data);
          this.reset();
          this.$emit('refresh');
        }
      },
      reset() {
        this.dialog = false;
        this.$refs[this.formComponent].reset();
      },
    },
  };
</script>
