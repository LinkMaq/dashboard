<template>
  <v-container fluid>
    <BaseBreadcrumb>
      <template #extend>
        <v-flex class="kubegems__full-right">
          <v-sheet class="text-body-2 text--darken-1">
            <BaseDatetimePicker
              ref="datetimePicker"
              v-model="date"
              :default-value="30"
              default-value-for-query
              query-end-time-key="CreatedAt_lte"
              query-start-time-key="CreatedAt_gte"
              value-change-to-query
              value-format="yyyy-MM-ddThh:mm:ss"
              @change="onDatetimeChange(undefined)"
            />
            <v-btn class="primary--text" small text @click="refresh">
              <v-icon left small> fas fa-redo </v-icon>
              刷新
            </v-btn>
          </v-sheet>
        </v-flex>
      </template>
    </BaseBreadcrumb>
    <v-card>
      <v-card-title class="py-4">
        <BaseFilter :default="{ items: [], text: '检索', value: 'search' }" :filters="filters" @refresh="filterList" />
      </v-card-title>
      <v-data-table
        class="mx-4 kubegems__table-row-pointer"
        disable-sort
        :headers="headers"
        hide-default-footer
        item-key="ID"
        :items="items"
        :items-per-page="params.size"
        no-data-text="暂无数据"
        :page.sync="params.page"
        show-expand
        single-expand
        @click:row="onRowClick"
        @item-expanded="toYaml"
      >
        <template #[`item.createdAt`]="{ item }">
          {{ item.CreatedAt ? $moment(item.CreatedAt).format('lll') : '' }}
        </template>
        <template #[`item.clientIP`]="{ item }">
          {{ item.ClientIP }}
        </template>
        <template #[`item.tenant`]="{ item }">
          {{ item.Tenant }}
        </template>
        <template #[`item.username`]="{ item }">
          {{ item.Username }}
        </template>
        <template #[`item.action`]="{ item }">
          {{ item.Action }}
        </template>
        <template #[`item.module`]="{ item }">
          {{ item.Module }}
        </template>
        <template #[`item.name`]="{ item }">
          {{ item.Name }}
        </template>
        <template #[`item.success`]="{ item }">
          <v-icon v-if="item.Success" color="success" small> mdi-check-circle </v-icon>
          <v-icon v-else color="error" small> mdi-close-circle </v-icon>
        </template>
        <template #expanded-item="{ headers }">
          <td :colspan="headers.length">
            <pre class="kubegems__word-all-break px-2">{{ yaml }}</pre>
          </td>
        </template>
        <template #[`item.labels`]="{ item }">
          <v-chip
            v-for="(value, key) in item ? item.Labels : {}"
            :key="key"
            class="ma-1"
            color="success"
            small
            text-color="white"
          >
            <strong class="mx-1"> {{ key }} </strong>
            {{ value }}
          </v-chip>
        </template>
      </v-data-table>
      <BasePagination
        v-if="pageCount >= 1"
        v-model="params.page"
        :page-count="pageCount"
        :size="params.size"
        @changepage="onPageIndexChange"
        @changesize="onPageSizeChange"
        @loaddata="auditList"
      />
    </v-card>
  </v-container>
</template>

<script>
  import { mapGetters, mapState } from 'vuex';

  import { getAuditList } from '@/api';
  import BaseFilter from '@/mixins/base_filter';
  import BaseSelect from '@/mixins/select';
  import { convertStrToNum } from '@/utils/helpers';

  export default {
    name: 'Audit',
    mixins: [BaseFilter, BaseSelect],
    data: () => ({
      items: [],
      headers: [
        { text: '时间', value: 'createdAt', align: 'start', width: 220 },
        { text: '租户', value: 'tenant', align: 'start', width: 100 },
        { text: '用户', value: 'username', align: 'start', width: 100 },
        { text: '操作', value: 'action', align: 'start', width: 100 },
        { text: 'Kind', value: 'module', align: 'start', width: 250 },
        { text: '对象', value: 'name', align: 'start', width: 300 },
        { text: '标签', value: 'labels', align: 'start' },
        { text: 'ClientIP', value: 'clientIP', align: 'start', width: 150 },
        { text: '状态', value: 'success', align: 'start', width: 100 },
        { text: '', value: 'data-table-expand' },
      ],
      pageCount: 0,
      date: [],
      params: {
        page: 1,
        size: 20,
        // todo
        CreatedAt_gte: null,
        CreatedAt_lte: null,
        order: '-CreatedAt',
      },
      yaml: '',
      userItems: [],
    }),
    computed: {
      ...mapState(['JWT', 'Admin', 'AdminViewport']),
      ...mapGetters(['Tenant']),
      filters() {
        const userItems = [];
        this.userItems.forEach((user) => {
          userItems.push({
            text: user.text,
            value: user.text,
            parent: 'Username',
          });
        });
        return [
          { text: '检索', value: 'search', items: [] },
          {
            text: '用户',
            value: 'Username',
            items: userItems,
          },
          {
            text: '状态',
            value: 'Success',
            items: [
              { text: '成功', value: 'true', parent: 'Success' },
              { text: '失败', value: 'false', parent: 'Success' },
            ],
          },
          {
            text: '操作',
            value: 'Action',
            items: [
              { text: '创建', value: '创建', parent: 'Action' },
              { text: '删除', value: '删除', parent: 'Action' },
              { text: '更新', value: '更新', parent: 'Action' },
              { text: '执行命令', value: '执行命令', parent: 'Action' },
              { text: '登录', value: '登录', parent: 'Action' },
              { text: '启用', value: '启用', parent: 'Action' },
              { text: '禁用', value: '禁用', parent: 'Action' },
            ],
          },
        ];
      },
    },
    mounted() {
      this.$nextTick(() => {
        this.generateSelectData();
        this.onDatetimeChange();
      });
    },
    methods: {
      async auditList() {
        // 点击更多
        this.$set(this.params, 'Tenant', this.Tenant().TenantName);
        if (this.AdminViewport) {
          this.$delete(this.params, 'Tenant');
        }
        const data = await getAuditList(this.params);
        this.items = data.List;
        this.pageCount = Math.ceil(data.Total / this.params.size);
        this.params.page = data.CurrentPage;
        this.$router.replace({ query: { ...this.$route.query, ...this.params } });
      },
      async generateSelectData() {
        if (this.AdminViewport) {
          await this.m_select_userSelectData();
          this.userItems = this.m_select_userItems;
        } else {
          await this.m_select_tenantUserSelectData();
          this.userItems = this.m_select_tenantUserItems;
        }
      },
      async onDatetimeChange() {
        Object.assign(this.params, convertStrToNum(this.$route.query));
        this.params.CreatedAt_gte = this.$moment(this.date[0]).format();
        this.params.CreatedAt_lte = this.$moment(this.date[1]).format();
        this.$router.replace({ query: { ...this.$route.query, ...this.params } });
        this.params.page = 1;
        this.auditList();
      },
      filterList(params) {
        const defaultparams = {
          page: 1,
          size: 20,
          CreatedAt_gte: null,
          CreatedAt_lte: null,
          order: '-CreatedAt',
        };
        for (const key in defaultparams) {
          if (this.params[key]) {
            defaultparams[key] = this.params[key];
          }
        }
        Object.assign(defaultparams, params);
        this.params = defaultparams;
        this.$router.replace({ query: { ...this.$route.query, ...this.params } });
        this.params.page = 1;
        this.auditList();
      },
      refresh() {
        this.$refs.datetimePicker.refresh();
        this.onDatetimeChange();
      },
      toYaml({ item, value }) {
        if (value) {
          if (item.RawData.request) {
            try {
              item.RawData.request = JSON.parse(item.RawData.request);
            } catch {
              //
            }
          }
          if (item.RawData.response) {
            try {
              item.RawData.response = JSON.parse(item.RawData.response);
            } catch {
              //
            }
          }
          this.yaml = this.$yamldump(item.RawData);
        }
      },
      onPageSizeChange(size) {
        this.params.page = 1;
        this.params.size = size;
      },
      onPageIndexChange(page) {
        this.params.page = page;
      },
      onRowClick(item, { expand, isExpanded }) {
        expand(!isExpanded);
      },
    },
  };
</script>
