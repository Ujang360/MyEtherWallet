<template>
  <div>
    <the-wrapper-dapp
      :is-new-header="true"
      :dapp-img="headerImg"
      :banner-text="header"
      :tab-items="tabs"
      :active-tab="activeTab"
      external-contents
      :on-tab="tabChanged"
      :valid-networks="validNetworks"
    >
      <!--
    =====================================================================================
      Collective
    =====================================================================================
    -->
      <template #tabContent1>
        <v-sheet
          max-width="1000px"
          min-height="500px"
          color="transparent"
          class="px-3 pb-13 pt-10 px-md-15 pb-md-15 pt-md-8 mx-auto"
        >
          <list-collective></list-collective>
        </v-sheet>
      </template>

      <!--
    =====================================================================================
     Proposal
    =====================================================================================
    -->
      <template #tabContent2>
        <v-sheet
          max-width="1000px"
          min-height="1000px"
          color="transparent"
          class="px-3 pb-13 pt-10 px-md-15 pb-md-15 pt-md-8 mx-auto"
        >
          <list-proposal> </list-proposal>
        </v-sheet>
      </template>
    </the-wrapper-dapp>
  </div>
</template>
<script>
import { SUPPORTED_NETWORKS } from './handlers/helpers/supportedNetworks';
import { ROOTSTOCK_COLLECTIVE_ROUTE } from './routes';
import handlerAnalytics from '@/modules/analytics-opt-in/handlers/handlerAnalytics.mixin.js';

export default {
  name: 'RootstockCollectiveLayout',
  components: {
    TheWrapperDapp: () => import('@/dapps/TheWrapperDapp.vue'),
    ListCollective: () => import('./components/collection/ListCollective'),
    ListProposal: () => import('./components/proposal/ListProposal')
  },
  mixins: [handlerAnalytics],
  data() {
    return {
      validNetworks: SUPPORTED_NETWORKS,
      headerImg: require('@/assets/images/icons/dapps/icon-dapp-rsk-collective.svg'),
      header: {
        title: this.$t('rootstockCollective.title'),
        subtext: this.$t('rootstockCollective.dapp-desc')
      },
      activeTab: 0,
      tabs: [
        {
          name: this.$t('rootstockCollective.collective.title'),
          route: { name: ROOTSTOCK_COLLECTIVE_ROUTE.MYCOLLECTIVE.NAME },
          id: 0
        },
        {
          name: this.$t('rootstockCollective.proposal.title'),
          route: {
            name: ROOTSTOCK_COLLECTIVE_ROUTE.PROPOSALS.NAME,
            path: ROOTSTOCK_COLLECTIVE_ROUTE.PROPOSALS.PATH
          },
          id: 1
        }
      ]
    };
  },
  computed: {},
  watch: {
    $route() {
      this.detactUrlChangeTab();
    }
  },
  mounted() {},
  beforeDestroy() {},
  methods: {
    tabChanged(tab) {
      this.activeTab = tab;
    },
    detactUrlChangeTab() {
      const currentRoute = this.$route.name;
      if (currentRoute === ROOTSTOCK_COLLECTIVE_ROUTE.PROPOSALS.NAME) {
        this.activeTab = this.tabs[1].id;
      } else {
        this.activeTab = this.tabs[0].id;
      }
    }
  }
};
</script>
