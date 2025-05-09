<template>
  <div>
    <mew-popup
      max-width="690px"
      :show="openVoteModal"
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
          <form @submit.prevent="voteProposal">
            <v-row class="mx-0">
              <v-col class="pr-0" cols="12">
                <label
                  >&nbsp;{{ $t('rootstockCollective.proposal.votingPower') }}:
                  <b> {{ myVotingPower }} </b>
                </label>
              </v-col>
              <v-col class="pr-0" cols="12">
                <mew-select
                  v-model="provider"
                  :has-filter="false"
                  :label="$t('rootstockCollective.proposal.voteType')"
                  :items="providers"
                  normal-dropdown
                  class="mr-3 flex-grow-1 mb-4"
                />
              </v-col>
            </v-row>
            <v-row class="mx-0">
              <v-col class="pl-0" cols="12">
                <div
                  v-if="Number(myVotingPower) == 0"
                  class="mb-2 pl-3 api-error text-danger"
                >
                  Not enough voting power to vote on proposal. Stake some RIF
                  first to get voting power.
                </div>
                <div class="align-right">
                  <div v-if="txLoading" class="pl-3 mt-2 text-primary mb-1">
                    Please wait few seconds... &nbsp;
                  </div>
                  <mew-button
                    color-theme="#ff9900"
                    :disabled="txLoading || Number(myVotingPower) == 0"
                    :loading="txLoading"
                    :has-full-width="false"
                    :title="$t('rootstockCollective.proposal.voteProposal')"
                    @click.native="voteProposal"
                  />
                </div>

                <div class="mt-2 api-error text-danger">{{ errorMsg }}</div>
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
import { BigNumber } from 'ethers';
import { ERROR, Toast } from '@/modules/toast/handler/handlerToast';

const VOTE_OPTIONS = [
  { name: 'Against' },
  { name: 'For' },
  { name: 'Abstain' }
];

export default {
  name: 'VoteModal',
  props: {
    openVoteModal: {
      default: false,
      type: Boolean
    },
    myVotingPower: {
      default: '',
      type: String
    },
    resetVoteModal: {
      type: Function,
      default: () => {}
    },
    govContract: {
      default: () => {},
      type: Object
    },
    proposalId: {
      default: '',
      type: String
    }
  },
  data() {
    return {
      msg: '',
      errorMsg: '',
      txLoading: false,
      modalTitle: 'Vote Proposal',
      provider: { name: 'For' },
      providers: VOTE_OPTIONS
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
    this.msg = '';
  },
  methods: {
    modalClose() {
      this.errorMsg = '';
      this.resetVoteModal();
    },
    async voteProposal() {
      try {
        this.txLoading = true;
        this.errorMsg = '';

        const hasVoted = await this.govContract.hasVoted(
          this.proposalId,
          this.address
        );

        if (!hasVoted) {
          // can vote proposal
          const VOTES_MAP = {
            against: 0,
            for: 1,
            abstain: 2
          };

          const vote = VOTES_MAP[this.provider.name.toLowerCase()];
          const pid = BigNumber.from(this.proposalId).toBigInt();
          const tx = await this.govContract.castVote(pid, vote);

          await tx.wait();

          this.txLoading = false;
          this.$emit('onVoteDone', tx, vote);
        } else {
          this.txLoading = false;
          this.errorMsg =
            'Voting successful. Your vote has been successfully cast!';
        }
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
.api-error {
  color: #ff445b;
  font-size: 12px;
}
</style>
