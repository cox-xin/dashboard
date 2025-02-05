<script>
import BrandImage from '@shell/components/BrandImage';
import TypeDescription from '@shell/components/TypeDescription';
import ResourceTable from '@shell/components/ResourceTable';
import Masthead from '@shell/components/ResourceList/Masthead';
import Loading from '@shell/components/Loading';
import { HARVESTER_NAME as VIRTUAL } from '@shell/config/product/harvester-manager';
import { CAPI, HCI, MANAGEMENT } from '@shell/config/types';
import { isHarvesterCluster } from '@shell/utils/cluster';
import { allHash } from '@shell/utils/promise';

export default {
  components: {
    BrandImage,
    ResourceTable,
    Masthead,
    TypeDescription,
    Loading
  },

  props:      {
    schema: {
      type:     Object,
      required: true,
    },
  },

  async fetch() {
    const inStore = this.$store.getters['currentProduct'].inStore;

    const hash = await allHash({
      hciClusters:  this.$store.dispatch(`${ inStore }/findAll`, { type: HCI.CLUSTER }),
      mgmtClusters: this.$store.dispatch(`${ inStore }/findAll`, { type: MANAGEMENT.CLUSTER })
    });

    this.hciClusters = hash.hciClusters;
    this.mgmtClusters = hash.mgmtClusters;
  },

  data() {
    const resource = CAPI.RANCHER_CLUSTER;

    return {
      VIRTUAL,
      hciDashboard: HCI.DASHBOARD,
      resource,
      hResource:    HCI.CLUSTER,
      realSchema:   this.$store.getters['management/schemaFor'](CAPI.RANCHER_CLUSTER),
      hciClusters:  [],
      mgmtClusters: []
    };
  },

  computed: {
    importLocation() {
      return {
        name:   'c-cluster-product-resource-create',
        params: {
          product:  this.$store.getters['currentProduct'].name,
          resource: this.schema.id,
        },
      };
    },

    canCreateCluster() {
      const schema = this.$store.getters['management/schemaFor'](CAPI.RANCHER_CLUSTER);

      return !!schema?.collectionMethods.find(x => x.toLowerCase() === 'post');
    },

    rows() {
      return this.hciClusters.filter((c) => {
        const cluster = this.mgmtClusters.find(cluster => cluster?.metadata?.name === c?.status?.clusterName);

        return isHarvesterCluster(cluster);
      });
    },

    typeDisplay() {
      return this.t(`typeLabel."${ HCI.CLUSTER }"`, { count: this.row?.length || 0 });
    },
  }
};
</script>

<template>
  <div>
    <Loading v-if="$fetchState.pending" />
    <template v-else>
      <Masthead
        :schema="realSchema"
        :resource="resource"
        :is-creatable="false"
        :type-display="typeDisplay"
      >
        <template #typeDescription>
          <TypeDescription :resource="hResource" />
        </template>

        <template v-if="canCreateCluster" slot="extraActions">
          <n-link
            :to="importLocation"
            class="btn role-primary"
          >
            {{ t('cluster.importAction') }}
          </n-link>
        </template>
      </Masthead>

      <ResourceTable
        v-if="rows && rows.length"
        :schema="schema"
        :rows="rows"
        :is-creatable="true"
        :namespaced="false"
      >
        <template #col:name="{row}">
          <td>
            <span>
              <n-link
                v-if="row.isReady"
                :to="{
                  name: `${VIRTUAL}-c-cluster-resource`,
                  params: {
                    cluster: row.status.clusterName,
                    product: VIRTUAL,
                    resource: hciDashboard
                  }
                }"
              >
                {{ row.nameDisplay }}
              </n-link>
              <span v-else>
                {{ row.nameDisplay }}
              </span>
            </span>
          </td>
        </template>

        <template #cell:harvester="{row}">
          <n-link
            class="btn btn-sm role-primary"
            :to="row.detailLocation"
          >
            {{ t('harvester.virtualizationManagement.manage') }}
          </n-link>
        </template>
      </ResourceTable>
      <div v-else>
        <div class="no-clusters">
          {{ t('harvester.manager.cluster.none') }}
        </div>
        <hr class="info-section" />
        <div class="logo">
          <BrandImage file-name="harvester.png" height="64" />
        </div>
        <div class="tagline">
          <div>{{ t('harvester.manager.cluster.description') }}</div>
        </div>
        <div class="tagline sub-tagline">
          <div v-html="t('harvester.manager.cluster.learnMore', {}, true)"></div>
        </div>
      </div>
    </template>
  </div>
</template>

<style lang="scss" scoped>
  .no-clusters {
    text-align: center;
  }

  .info-section {
    margin-top: 60px;
  }

  .logo {
    display: flex;
    justify-content: center;
    margin: 60px 0 40px 0;
  }

  .tagline {
    display: flex;
    justify-content: center;
    margin-top: 30px;

    > div {
      font-size: 16px;
      line-height: 22px;
      max-width: 80%;
      text-align: center;
    }
  }
</style>
