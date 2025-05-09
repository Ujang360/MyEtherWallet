<template>
  <div>
    <mew-popup
      max-width="690px"
      :show="openStakingModal"
      :has-buttons="false"
      :has-body-content="true"
      :title="modalTitle"
      :left-btn="leftBtn"
    >
      <v-sheet
        max-width="600px"
        color="transparent"
        class="px-1 py-1 py-md-1 mx-auto"
      >
        <div class="mb-1">
          <h3 v-if="step == 1" class="text-center mb-2">
            Step 1: Allow RIF for Staking.
          </h3>
          <h3 v-if="step == 2" class="text-center mb-2">Step 2: Stake RIF.</h3>
          <form @submit.prevent="approveRIF">
            <v-row class="mx-0">
              <v-col class="pr-0" cols="12">
                <mew-input
                  :value="stakeAmount"
                  :has-clear-btn="true"
                  :label="$t('rootstockCollective.collective.stakeInputLabel')"
                  :placeholder="
                    $t('rootstockCollective.collective.stakeInputPlaceholder')
                  "
                  :disabled="step == 2"
                  class="mr-3 flex-grow-1"
                  @input="setStakeAmount"
                />
                <span
                  >Available Balance: {{ ethersFormat(rifBalance) }} RIF</span
                >

                <div
                  v-if="Number(rifBalance) == 0"
                  class="mt-2 api-error text-danger"
                >
                  Not enough RIF balance for staking.<a
                    href="https://rif.technology/rif-token/"
                    target="_blank"
                    class="theme-color"
                  >
                    &nbsp;See How to get RIF</a
                  >
                </div>
              </v-col>
            </v-row>
            <v-row class="mx-0">
              <v-col class="pl-0" cols="12">
                <div v-if="step == 1" class="pl-3">
                  <mew-alert
                    hide-close-icon
                    class="font-weight-light d-flex justify-space-between align-center mb-4 mt-2"
                  >
                    Please allow us permission to use your RIF tokens to proceed
                    with staking. This is a necessary step to ensure everything
                    runs smoothly.
                  </mew-alert>
                </div>

                <div v-if="step == 2" class="pl-3">
                  <mew-alert
                    hide-close-icon
                    class="font-weight-light d-flex justify-space-between align-center mb-4 mt-2"
                  >
                    Successfully approved {{ stakeAmount }} RIF for Staking.
                    Click stake to complete the staking process.
                  </mew-alert>
                </div>
                <div class="align-right">
                  <mew-button
                    v-if="step == 1"
                    color-theme="#ff9900"
                    :loading="txLoading"
                    :disabled="
                      !stakeAmount ||
                      Number(stakeAmount) <= 0 ||
                      Number(rifBalance) < Number(stakeAmount) ||
                      isNaN(Number(stakeAmount)) ||
                      txLoading
                    "
                    :has-full-width="false"
                    :title="$t('rootstockCollective.collective.approve')"
                    @click.native="approveRIF"
                  />
                </div>

                <div class="align-right">
                  <div v-if="txLoading" class="pl-3 mt-2 text-primary mb-1">
                    Please wait few seconds... &nbsp;
                  </div>
                  <mew-button
                    v-if="step == 2"
                    color-theme="#ff9900"
                    :loading="txLoading"
                    :disabled="txLoading"
                    :has-full-width="false"
                    :title="$t('rootstockCollective.collective.stake')"
                    @click.native="stakeRIF"
                  />
                </div>

                <div class="pl-3 mt-2 api-error text-danger">
                  {{ errorMsg }}
                </div>
              </v-col>
            </v-row>
          </form>
        </div>
      </v-sheet>
    </mew-popup>
  </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex';
import { ethers } from 'ethers';
import { ERROR, Toast } from '@/modules/toast/handler/handlerToast';
import {
  getContractAddress,
  ContractType
} from '../../handlers/helpers/contracts';

export default {
  name: 'StakingModal',
  props: {
    openStakingModal: {
      default: false,
      type: Boolean
    },
    rifBalance: {
      default: '',
      type: String
    },
    resetStakeModal: {
      type: Function,
      default: () => {}
    },
    rifContract: {
      default: () => {},
      type: Object,
      validator(value) {
        return value && typeof value.approve === 'function';
      }
    },
    stRifContract: {
      default: () => {},
      type: Object,
      validator(value) {
        return value && typeof value.depositAndDelegate === 'function';
      }
    }
  },
  data() {
    return {
      errorMsg: '',
      txLoading: false,
      step: 1,
      modalTitle: 'Stake RIF',
      stakeAmount: ''
    };
  },
  computed: {
    ...mapState('wallet', ['balance', 'address', 'web3', 'instance']),
    ...mapGetters('global', ['network']),
    leftBtn() {
      return {
        text: 'Cancel',
        color: 'basic',
        method: this.modalClose
      };
    }
  },
  mounted() {
    this.errorMsg = '';
    this.txLoading = false;
    this.step = 1;
    this.modalTitle = 'Stake RIF';
    this.stakeAmount = '';
  },
  methods: {
    ethersFormat(rifBalance) {
      return Number(rifBalance).toFixed(2);
    },
    setStakeAmount(v) {
      this.stakeAmount = v;
    },
    modalClose() {
      this.setStakeAmount('');
      this.errorMsg = '';
      this.resetStakeModal();
    },
    async stakeRIF() {
      try {
        this.txLoading = true;
        this.errorMsg = '';

        const stakeTx = await this.stRifContract.depositAndDelegate(
          this.address.toLowerCase(),
          ethers.utils.parseEther(this.stakeAmount)
        );

        await stakeTx.wait();

        this.txLoading = false;
        this.errorMsg = '';
        this.txLoading = false;
        this.step = 1;
        this.$emit('onStakeDone', stakeTx);
      } catch (e) {
        this.txLoading = false;
        this.errorMsg = 'Transaction rejected!';
        Toast(e, ERROR);
      }
    },
    async approveRIF() {
      try {
        if (Number(this.stakeAmount) > Number(this.rifBalance)) {
          return (this.errorMsg = 'Not enough RIF balance!');
        }

        this.txLoading = true;
        this.errorMsg = '';
        const strifAddress = getContractAddress(
          ContractType.STRIF,
          this.network.type.chainID
        );
        const rifTx = await this.rifContract.approve(
          strifAddress,
          ethers.utils.parseEther(this.stakeAmount)
        );

        await rifTx.wait();
        this.txLoading = false;
        this.step = 2;
      } catch (e) {
        this.txLoading = false;
        this.errorMsg = 'Transaction rejected!';
        Toast(e, ERROR);
      }
    }
  }
};
</script>
<style lang="scss" scoped>
.align-right {
  display: flex;
  align-items: flex-end;
  justify-content: flex-end;
}
.dao-table {
  width: 100%;
  text-align: left;
  .table-header {
    text-align: left;
    th {
      padding: 1rem;
    }
  }

  td {
    padding: 1rem;
  }
}

.theme-color {
  color: #ff9900;
}
.api-error {
  color: #ff445b;
  font-size: 12px;
}
.btc-amount {
  position: relative;
  .notes {
    position: absolute;
    right: 21px;
    bottom: 36px;
    font-size: 10px;
    font-style: italic;
  }
}
</style>
