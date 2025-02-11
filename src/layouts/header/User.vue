<template>
  <v-menu
    v-model="menu"
    bottom
    :close-on-content-click="false"
    left
    max-width="320px"
    min-width="200px"
    nudge-bottom="15px"
    offset-y
    origin="top right"
    transition="scale-transition"
    z-index="1000"
  >
    <template #activator="{ on }">
      <v-btn class="mr-1" dark icon v-on="on" @click="expand = false">
        <v-icon>fas fa-user-circle</v-icon>
      </v-btn>
    </template>

    <v-card>
      <v-list-item class="py-2 primary white--text" two-line>
        <v-list-item-avatar>
          <!-- <v-icon large>fas fa-user-circle</v-icon> -->
          <v-avatar
            class="primary--text font-weight-medium"
            color="white"
            :size="45"
            style="min-width: 40px; width: 40px"
          >
            <span class="text-h5">
              {{ User.Username ? User.Username[0].toLocaleUpperCase() : 'N' }}
            </span>
          </v-avatar>
        </v-list-item-avatar>
        <v-list-item-content>
          <v-list-item-title class="text-h6 white--text kubegems__text">
            {{ User.Username }}
            <v-chip class="mr-1 primary--text mt-n1 ml-2" color="white" pill small>
              <v-avatar class="mr-0" color="white" left>
                <v-btn class="primary--text" color="white" small>
                  <BaseLogo
                    class="primary--text logo-margin mt-1"
                    :icon-name="User.SourceVendor ? User.SourceVendor.toLocaleLowerCase() : 'kubegems'"
                    :ml="0"
                    :width="20"
                  />
                </v-btn>
              </v-avatar>
              <span class="font-weight-medium primary--text kubegems__text">
                {{ $VENDOR[User.SourceVendor] || 'Selfhosted' }}
              </span>
            </v-chip>
            <div class="kubegems__clear-float" />
          </v-list-item-title>
          <v-list-item-subtitle class="white--text">
            {{ User && User.Email && User.Email.length === 0 ? '暂无邮箱' : User.Email }}
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
      <v-divider class="mb-2" />
      <v-list class="pt-0 px-2">
        <v-list-item @click="showTenantSelect">
          <v-list-item-avatar height="25" min-width="25" width="25">
            <v-icon color="primary">mdi-account-multiple</v-icon>
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title class="text-body-2 font-weight-medium kubegems__text">
              租户
              <v-flex class="float-right white--text blue-grey lighten-2 px-1 user-chip label">
                {{ Tenant().TenantName }}
              </v-flex>
              <div class="kubegems__clear-float" />
            </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click="toUserCenter">
          <v-list-item-avatar height="25" min-width="25" width="25">
            <v-icon color="primary">mdi-account</v-icon>
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title class="text-body-2 font-weight-medium kubegems__text">
              用户中心
              <v-flex class="float-right white--text blue-grey lighten-2 px-1 user-chip label">
                {{ Admin ? '管理员' : '普通用户' }}
              </v-flex>
              <div class="kubegems__clear-float" />
            </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click="toBook">
          <v-list-item-avatar height="25" min-width="25" width="25">
            <v-icon color="primary">mdi-book-open-variant</v-icon>
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title class="text-body-2 font-weight-medium kubegems__text"> 产品手册 </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click="showAbout">
          <v-list-item-avatar height="25" min-width="25" width="25">
            <v-icon color="primary">mdi-vimeo</v-icon>
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title class="text-body-2 font-weight-medium kubegems__text"> 关于 </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
      <v-divider />
      <v-card-actions>
        <v-spacer />
        <v-btn color="primary" text @click="logout">
          <v-icon left small> fas fa-sign-out-alt </v-icon>
          <span class="font-weight-medium kubegems__text">退出</span>
        </v-btn>
        <v-spacer />
      </v-card-actions>
    </v-card>

    <About ref="about" />
    <TenantSelect ref="tenantSelect" />
  </v-menu>
</template>

<script>
  import { mapGetters, mapState } from 'vuex';

  import About from './components/About';
  import TenantSelect from './components/TenantSelect';
  import BasePermission from '@/mixins/permission';
  import BaseSelect from '@/mixins/select';

  export default {
    name: 'User',
    components: {
      About,
      TenantSelect,
    },
    mixins: [BasePermission, BaseSelect],
    data() {
      return {
        menu: false,
        expand: false,
      };
    },
    computed: {
      ...mapState(['Admin', 'User', 'JWT']),
      ...mapGetters(['Tenant', 'Project']),
    },
    methods: {
      async logout() {
        this.$store.commit('CLEARALL');
        await this.$router.push({ name: 'login' });
        this.$store.commit('SET_VERSION', process.env.VUE_APP_RELEASE);
      },
      showAbout() {
        this.$refs.about.init();
        this.$refs.about.open();
        this.closeUserMenu();
      },
      showTenantSelect() {
        this.$refs.tenantSelect.init();
        this.$refs.tenantSelect.open();
        this.closeUserMenu();
      },
      toBook() {
        window.open(`https://www.kubegems.io/docs/concepts/what-is-kubegems`);
        this.closeUserMenu();
      },
      toUserCenter() {
        this.$router.push({ name: 'user-center' });
        this.closeUserMenu();
      },
      closeUserMenu() {
        this.menu = false;
        this.expand = false;
      },
    },
  };
</script>

<style lang="scss" scoped>
  .label {
    line-height: 22px;
  }
  .user-chip {
    border-radius: 3px;
    min-width: 80px;
    text-align: center;
  }
</style>
