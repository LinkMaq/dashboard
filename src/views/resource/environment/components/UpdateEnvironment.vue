<template>
  <BaseDialog v-model="dialog" icon="mdi-cube" title="更新环境" :width="1000" @reset="reset">
    <template #content>
      <component :is="formComponent" :ref="formComponent" :edit="true" :step="step" />
    </template>
    <template #action>
      <v-btn class="float-right mx-2" color="primary" :loading="Circular" text @click="updateEnvironment"> 确定 </v-btn>
    </template>
  </BaseDialog>
</template>

<script>
  import { mapGetters, mapState } from 'vuex';

  import EnvironmentBaseForm from './EnvironmentBaseForm';
  import { getEnvironmentDetail, putUpdateEnvironment } from '@/api';
  import BaseSelect from '@/mixins/select';
  import { deepCopy } from '@/utils/helpers';

  export default {
    name: 'UpdateEnvironment',
    components: {
      EnvironmentBaseForm,
    },
    mixins: [BaseSelect],
    data: () => ({
      dialog: false,
      formComponent: 'EnvironmentBaseForm',
      users: [],
      step: 0,
    }),
    computed: {
      ...mapState(['Circular', 'User']),
      ...mapGetters(['Project', 'Tenant']),
    },
    methods: {
      open() {
        this.dialog = true;
      },
      async updateEnvironment() {
        if (!this.$refs[this.formComponent]) {
          return;
        }
        if (!this.$refs[this.formComponent].checkSaved()) {
          this.$store.commit('SET_SNACKBAR', {
            text: '请保存数据',
            color: 'warning',
          });
          return;
        }
        if (this.$refs[this.formComponent].validate()) {
          const data = this.$refs[this.formComponent].getData().data;
          const resdata = await putUpdateEnvironment(data.ID, {
            EnvironmentID: data.ID,
            EnvironmentName: data.EnvironmentName,
            ClusterID: data.ClusterID,
            MetaType: data.MetaType,
            DeletePolicy: data.DeletePolicy,
            Namespace: data.Namespace,
            ProjectID: data.ProjectID,
            Remark: data.Remark,
            CreatorID: this.User.ID,
            ResourceQuota: data.ResourceQuota,
            LimitRange: data.LimitRange,
          });

          if (this.Project().ID === resdata.ProjectID) {
            await this.m_select_projectEnvironmentSelectData(this.Project().ID);
          }
          this.reset();
          this.$emit('refresh');
        }
      },
      async init(item) {
        const data = await getEnvironmentDetail(item.ID);
        data.Users.forEach((u) => {
          u.UserID = u.ID;
        });
        this.users = deepCopy(data.Users);
        this.$refs[this.formComponent].init(data);
      },
      reset() {
        this.dialog = false;
        this.step = 0;
        this.$refs[this.formComponent].reset();
      },
    },
  };
</script>
