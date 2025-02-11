<template>
  <v-card class="metrics-item">
    <div class="metrics-item__title">
      <span class="text-body-1 kubegems__text">{{ title }}</span>
      <v-btn color="primary" small @click="setAlert"> 设置告警 </v-btn>
    </div>
    <v-row :style="{ maxHeight: `${maxHeight}px` }">
      <v-col v-for="label in labels" :key="label.text" class="py-1 px-4" cols="4">
        <v-autocomplete
          attach
          class="my-1"
          dense
          flat
          hide-details
          hide-selected
          :items="label.items"
          :label="label.text"
          multiple
          no-data-text="暂无可选数据"
          solo
          :value="labelpairs[label.text]"
          @change="onLabelChange($event, label.text)"
          @focus="onLoadLabelFocus(label.text)"
        >
          <template #selection="{ item, parent, index }">
            <v-chip v-if="index === 0" class="my-1" color="primary" small>
              {{ item }}
            </v-chip>
            <v-chip v-if="index === 1" class="my-1" color="primary" small> +{{ parent.value.length - 1 }} </v-chip>
          </template>
        </v-autocomplete>
      </v-col>
    </v-row>

    <div class="metrics-item__chart">
      <div ref="container" class="metrics-item__container">
        <BaseApexAreaChart
          v-if="data"
          chart-type="line"
          :class="`clear-zoom-${Scale.toString().replaceAll('.', '-')}`"
          colorful
          :extend-height="295"
          label="all"
          :label-show="false"
          :metrics="data ? data.data : []"
          :no-data-offset-y="-25"
          type=""
          :unit="getUnit(unit)"
        />
      </div>
    </div>
  </v-card>
</template>

<script>
  import { mapState } from 'vuex';

  import { debounce } from '@/utils/helpers';
  import { SERVICE_MONITOR_NS } from '@/utils/namespace';

  export default {
    name: 'MetricsItem',
    props: {
      data: {
        type: Object,
        default: () => null,
      },
      labelpairs: {
        type: Object,
        default: () => ({}),
      },
      labelObject: {
        type: Object,
        default: () => ({}),
      },
      title: {
        type: String,
        default: '',
      },
      unit: {
        type: String,
        default: '',
      },
    },
    data() {
      this._onResize = debounce(this.onResize, 200);

      return {
        size: {
          width: 0,
          height: 0,
        },
      };
    },
    computed: {
      ...mapState(['Scale']),
      labels() {
        return Object.values(this.labelObject);
      },
      maxHeight() {
        if (this.labels.length % 3 === 0) {
          return (48 * this.labels.length) / 3;
        } else {
          return 48 * parseInt(this.labels.length / 3 + 1);
        }
      },
    },
    mounted() {
      this.$nextTick(() => {
        this.onResize();
      });
      window.addEventListener('resize', this._onResize);
    },
    beforeDestroy() {
      window.removeEventListener('resize', this._onResize);
    },
    methods: {
      onResize() {
        if (this.$refs.container) {
          this.size = {
            width: this.$refs.container.offsetWidth,
            height: this.$refs.container.offsetHeight,
          };
        }
      },
      setAlert() {
        const { resource, rule, unit, cluster, environment, ql, expr } = this.data?._$origin;
        const labelpairs = {};
        for (const key in this.labelpairs) {
          if (this.labelpairs[key] && this.labelpairs[key].length) {
            labelpairs[key] = this.labelpairs[key].reduce(
              (pre, current, index, arr) => pre + current + `${index === arr.length - 1 ? '' : '|'}`,
              '',
            );
          }
        }
        let params = {};
        if (ql) {
          params = { expr: expr };
        } else {
          params = {
            promqlGenerator: {
              resource: resource._$value,
              rule: rule._$value,
              unit: unit?.value,
            },
          };
        }
        this.$emit(
          'alert',
          Object.assign(
            {
              name: '',
              for: '1m',
              promqlGenerator: null,
              labelpairs,
              alertLevels: [],
              receivers: [],
            },
            params,
          ),
        );

        this.$router.replace({
          query: {
            cluster: environment?.Cluster.ClusterName || cluster?.text,
            namespace: environment?.Namespace || SERVICE_MONITOR_NS,
          },
        });
      },
      onLoadLabelFocus(label) {
        this.$emit('loadLabel', label);
      },
      onLabelChange(value, label) {
        this.$emit('change', { label, value });
      },
      onRefresh() {
        this.$emit('refresh');
      },
      getUnit(unit) {
        if (unit === 'short') {
          return 'short';
        }
        if (unit && unit.indexOf('-') > -1) {
          return unit.substr(unit.indexOf('-') + 1);
        }
        return unit;
      },
    },
  };
</script>

<style lang="scss" scoped>
  .metrics-item {
    position: relative;
    display: flex;
    flex-direction: column;
    height: 456px;
    padding: 12px;

    &__title {
      display: flex;
      justify-content: space-between;
      margin-bottom: 24px;
    }

    &__chart {
      position: relative;
      flex: auto;
    }

    &__container {
      position: absolute;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
    }
  }
</style>
