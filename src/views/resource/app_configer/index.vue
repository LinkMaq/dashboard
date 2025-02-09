<template>
  <v-container fluid>
    <BaseViewportHeader />

    <BaseBreadcrumb>
      <template #extend>
        <v-flex class="kubegems__full-right">
          <div class="float-left provider text-body-2 kubegems__text mr-2">Provider</div>
          <div class="float-left provider__img"><img src="/icon/nacos.png" width="80" /></div>

          <v-menu
            bottom
            :close-delay="200"
            :close-on-content-click="false"
            content-class="conf__menu"
            max-width="320px"
            offset-y
            open-on-hover
            origin="top center"
            transition="scale-transition"
          >
            <template #activator="{ on }">
              <v-btn class="kubegems__full-right" color="primary" small text v-on="on">获取访问信息</v-btn>
            </template>
            <v-card>
              <v-flex class="text-body-2 text-center primary white--text py-2">
                <v-icon color="white" left small> mdi-brightness-7 </v-icon>
                <span>访问信息</span>
              </v-flex>
              <v-list class="pa-0 kubegems__tip" dense>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item class="float-left pa-0" two-line>
                      <v-list-item-content class="py-0">
                        <v-list-item-title> endpoint </v-list-item-title>
                        <v-list-item-content class="text-caption kubegems__text">
                          nacos-client.nacos:8848
                        </v-list-item-content>
                      </v-list-item-content>
                    </v-list-item>
                    <v-list-item class="float-left pa-0" two-line>
                      <v-list-item-content class="py-0">
                        <v-list-item-title> namespace </v-list-item-title>
                        <v-list-item-content class="text-caption kubegems__text">
                          {{ baseInfoData.nacos_tenant }}
                        </v-list-item-content>
                      </v-list-item-content>
                    </v-list-item>
                    <v-list-item class="float-left pa-0" two-line>
                      <v-list-item-content class="py-0">
                        <v-list-item-title> group </v-list-item-title>
                        <v-list-item-content class="text-caption kubegems__text">
                          {{ baseInfoData.nacos_group }}
                        </v-list-item-content>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-card>
          </v-menu>
        </v-flex>
      </template>
    </BaseBreadcrumb>

    <v-card>
      <v-card-title class="py-4">
        <BaseFilter
          :default="{ items: [], text: '配置项', value: 'search' }"
          :filters="filters"
          :reload="false"
          @filter="customFilter"
          @refresh="m_filter_list"
        />

        <v-spacer />
        <v-menu left>
          <template #activator="{ on }">
            <v-btn icon>
              <v-icon color="primary" small v-on="on"> fas fa-ellipsis-v </v-icon>
            </v-btn>
          </template>
          <v-card>
            <v-card-text class="pa-2">
              <v-flex>
                <v-btn color="primary" text @click="openCreateDialog">
                  <v-icon left>mdi-key-plus</v-icon>
                  创建配置项
                </v-btn>
              </v-flex>
            </v-card-text>
          </v-card>
        </v-menu>
      </v-card-title>
      <v-data-table
        class="mx-4"
        :headers="headers"
        hide-default-footer
        :items="items"
        :items-per-page="params.size"
        no-data-text="暂无数据"
        :page.sync="params.page"
        @page-count="pageCount = $event"
      >
        <template #[`item.createdTime`]="{ item }">
          <span> {{ item.createdTime ? $moment(item.createdTime).format('lll') : '' }} </span>
        </template>
        <template #[`item.lastModifiedTime`]="{ item }">
          <span> {{ item.lastModifiedTime ? $moment(item.lastModifiedTime).format('lll') : '' }} </span>
        </template>
        <template #[`item.action`]="{ item, index }">
          <v-flex :id="`r${index}`" />
          <v-menu :attach="`#r${index}`" left>
            <template #activator="{ on }">
              <v-btn icon>
                <v-icon color="primary" x-small v-on="on"> fas fa-ellipsis-v </v-icon>
              </v-btn>
            </template>
            <v-card>
              <v-card-text class="pa-2">
                <v-flex>
                  <v-btn color="primary" small text @click="editConfig(item)"> 编辑/查看 </v-btn>
                </v-flex>
                <v-flex>
                  <v-btn color="primary" small text @click="showSDK(item)"> SDK示例 </v-btn>
                </v-flex>
                <v-flex>
                  <v-btn color="primary" small text @click="showHistory(item)"> 历史 </v-btn>
                </v-flex>
                <v-flex>
                  <v-btn color="primary" small text @click="showListener(item)"> 监听查询 </v-btn>
                </v-flex>
                <v-flex>
                  <v-btn color="error" small text @click="removeConfig(item, index)"> 删除 </v-btn>
                </v-flex>
              </v-card-text>
            </v-card>
          </v-menu>
        </template>
      </v-data-table>
      <BasePagination
        v-if="pageCount >= 1"
        v-model="params.page"
        :front-page="true"
        :page-count="pageCount"
        :size="params.size"
        @changepage="onPageIndexChange"
        @changesize="onPageSizeChange"
        @loaddata="appConfigList"
      />
    </v-card>

    <ConfigEditor
      :is-create="editor.isCreate"
      :item="editor.currentEditItem"
      :show-edit-dialog="editor.showEditDialog"
      @close="closeEditDialog"
      @submit="submitEditContent"
    />
    <ConfigSDK ref="configSDK" />
    <ConfigListener ref="configListener" />
    <DeleteItem
      :idx="editor.deleteIdx"
      :item="editor.currentDeleteItem"
      :show-delete-dialog="editor.showDeleteDialog"
      @close="closeDeleteDialog"
      @delete="submitDeleteItem"
    />
    <HistoryView
      :history-item="editor.currentHistoryItem"
      :show-history-dialog="editor.showHistoryDialog"
      @close="closeHistoryDialog"
      @rollback="rollbackHistory"
    />
  </v-container>
</template>

<script>
  import { mapGetters, mapState } from 'vuex';

  import { delConfigItems, listConfigItems, pubConfigItems, baseInfo } from './api/index.js';
  import ConfigEditor from './components/ConfigEditor';
  import ConfigListener from './components/ConfigListener';
  import ConfigSDK from './components/ConfigSDK';
  import DeleteItem from './components/DeleteItem';
  import HistoryView from './components/HistoryView';
  import BaseFilter from '@/mixins/base_filter';
  import { deepCopy } from '@/utils/helpers';

  export default {
    name: 'ConfigeUI',
    components: {
      ConfigEditor,
      ConfigSDK,
      ConfigListener,
      DeleteItem,
      HistoryView,
    },
    mixins: [BaseFilter],
    data() {
      return {
        editor: {
          currentEditItem: {
            tenant: '',
            project: '',
            environment: '',
            application: '',
            key: '',
            value: '',
          },
          showEditDialog: false,
          isCreate: false,
          showDeleteDialog: false,
          currentDeleteItem: {},
          deleteIdx: -1,
          showHistoryDialog: false,
          currentHistoryItem: {},
        },
        items: [],
        itemsCopy: [],
        baseInfoData: {
          provider: '',
          nacos_tenant: '',
          nacos_group: '',
        },
        headers: [
          { text: 'dataid', value: 'key', align: 'start' },
          { text: 'app', value: 'application', align: 'start' },
          { text: 'create', value: 'createdTime', align: 'center', width: 260, sortable: false },
          { text: 'update', value: 'lastModifiedTime', align: 'center', width: 260, sortable: false },
          { text: 'operator', value: 'lastUpdateUser', align: 'center', width: 160, sortable: false },
          { text: '', value: 'action', align: 'center', width: 20, sortable: false },
        ],
        pageCount: 0,
        params: {
          page: 1,
          size: 10,
        },
        filters: [{ text: '配置名称', value: 'search', items: [] }],
      };
    },
    computed: {
      ...mapState(['JWT', 'Admin', 'AdminViewport', 'MessageStreamWS']),
      ...mapGetters(['Tenant', 'Environment', 'Project']),
    },
    mounted() {
      this.appConfigList();
      this.baseInfo();
    },
    methods: {
      async baseInfo() {
        const data = await baseInfo(
          this.Tenant().TenantName || '',
          this.Project().ProjectName || '',
          this.Environment().EnvironmentName || '',
          { noprocessing: true },
        );
        this.baseInfoData = data;
      },
      async appConfigList() {
        const datas = await listConfigItems(
          this.Tenant().TenantName || '',
          this.Project().ProjectName || '',
          this.Environment().EnvironmentName || '',
        );
        this.items = datas;
        this.itemsCopy = deepCopy(datas);
      },
      customFilter() {
        if (this.$route.query.search && this.$route.query.search.length > 0) {
          this.items = this.itemsCopy.filter((item) => {
            return (
              item.application &&
              item.application.toLocaleLowerCase().indexOf(this.$route.query.search.toLocaleLowerCase()) > -1
            );
          });
        } else {
          this.items = this.itemsCopy;
        }
      },
      onPageSizeChange(size) {
        this.params.page = 1;
        this.params.size = size;
      },
      onPageIndexChange(page) {
        this.params.page = page;
      },
      editConfig(item) {
        this.editor.currentEditItem = {
          tenant: (this.Tenant() || { TenantName: '' }).TenantName,
          project: (this.Project() || { ProjectName: '' }).ProjectName,
          environment: (this.Environment() || { EnvironmentName: '' }).EnvironmentName,
          ...item,
        };
        this.editor.isCreate = false;
        this.editor.showEditDialog = true;
      },
      openCreateDialog() {
        this.editor.currentEditItem = {
          tenant: (this.Tenant() || { TenantName: '' }).TenantName,
          project: (this.Project() || { ProjectName: '' }).ProjectName,
          environment: (this.Environment() || { EnvironmentName: '' }).EnvironmentName,
          application: '',
          key: '',
          value: '',
        };
        this.editor.isCreate = true;
        this.editor.showEditDialog = true;
      },
      removeConfig(item, idx) {
        this.editor.currentDeleteItem = item;
        this.editor.deleteIdx = idx;
        this.editor.showDeleteDialog = true;
      },
      closeEditDialog() {
        this.editor.showEditDialog = false;
        this.editor.currentEditItem = {
          tenant: (this.Tenant() || { TenantName: '' }).TenantName,
          project: (this.Project() || { ProjectName: '' }).ProjectName,
          environment: (this.Environment() || { EnvironmentName: '' }).EnvironmentName,
          application: '',
          key: '',
          value: '',
        };
      },
      closeDeleteDialog() {
        this.editor.currentDeleteItem = {};
        this.editor.deleteIdx = -1;
        this.editor.showDeleteDialog = false;
      },
      closeHistoryDialog() {
        this.editor.showHistoryDialog = false;
        this.editor.currentHistoryItem = {};
      },
      async showHistory(item) {
        this.editor.showHistoryDialog = true;
        this.editor.currentHistoryItem = item;
      },
      async submitDeleteItem(idx) {
        await delConfigItems(
          this.editor.currentDeleteItem.tenant,
          this.editor.currentDeleteItem.project,
          this.editor.currentDeleteItem.application,
          this.editor.currentDeleteItem.environment,
          this.editor.currentDeleteItem.key,
        );
        this.items.splice(idx, 1);
        this.closeDeleteDialog();
      },
      rollbackHistory(ritem) {
        const citem = this.items.find((item) => {
          return item.key === ritem.key;
        });
        citem.value = ritem.value;
        citem.lastModifiedTime = ritem.lastModifiedTime;
        citem.createdTime = ritem.createdTime;
        citem.lastUpdateUser = ritem.lastUpdateUser;
      },
      async submitEditContent(editItem, isCreate) {
        if (editItem.value === this.editor.currentEditItem.value) {
          this.closeEditDialog();
          return;
        }
        const res = await pubConfigItems(
          editItem.tenant,
          editItem.project,
          editItem.application,
          editItem.environment,
          editItem.key,
          editItem.value,
        );
        if (isCreate) {
          this.items.push(res);
        } else {
          const citem = this.items.find((item) => {
            return item.key === editItem.key;
          });
          citem.value = editItem.value;
          citem.lastModifiedTime = res.lastModifiedTime;
          citem.createdTime = res.createdTime;
          citem.lastUpdateUser = res.lastUpdateUser;
        }
        this.closeEditDialog();
      },
      showSDK() {
        this.$refs.configSDK.open();
      },
      showListener(item) {
        this.$refs.configListener.open(item);
      },
    },
  };
</script>

<style lang="scss" scoped>
  .provider {
    line-height: 64px;
    font-weight: bold !important;

    &__img {
      margin-right: 120px;
      margin-top: 23px;
    }
  }

  .conf__menu {
    right: 10px !important;
    left: auto !important;
  }
</style>
