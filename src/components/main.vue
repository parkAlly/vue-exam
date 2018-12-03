<template>
<div>
  <search v-model="accountName" placeholder="New todo"  @keydown.enter="getAccountInfo"  @getinfo="getAccountInfo"/>
  <div class="row" id="info" v-if="showInfo">
    <h2 class="col-12 text-center" id="account">
      EOS ACCOUNT
    </h2>
    <div class="col-12 row">
      <div class="col-6">
        <user-info
          v-for="(value, title) in info"
          :title = title
          :value = value
        />
      </div>
      <div class="col-6">
        <form>
          <div class="form-group">
            <label>상대계정</label>
            <input type="text" class="form-control" id="to" aria-describedby="emailHelp" placeholder="받는계정" v-model="to">
          </div>
          <div class="form-group">
            <label>수량</label>
            <input type="number" class="form-control" id="quantity" placeholder="수량" v-model="quantity">
          </div>
          <div class="form-group">
            <label>메모</label>
            <textarea class="form-control" id="memo" placeholder="memo" v-model="memo"></textarea>
          </div>
          <div class="form-group">
            <label>privateKey</label>
            <input type="text" class="form-control" id="privateKey" placeholder="privateKey" v-model="privateKey">
          </div>
          <button type="button" @click="transfer()" class="btn btn-primary">Submit</button>
        </form>
      </div>
    </div>
  </div>
  <message :result="result"/>
</div>
</template>

<script>

const Eos = require('eosjs')
const config = {
  httpEndpoint: 'https://api-kylin.eosasia.one',
  chainId: '5fff1dae8dc8e2fc4d5b23b2c7665c97f9e9d8edf2b6485a86ba311c25639191'
}
import message from "./message.vue";
import search from "./search.vue";
import userInfo from "./userInfo.vue";

export default {
  components: {
    message,
    search,
    userInfo
  },
  data: function () {
    return {
      accountName: '',
      showInfo: false,
      result: {},
      info: {
        total : 0,
        unStake : 0,
        stake : 0,
        refund : 0,
        cpu : 0,
        net : 0,
        ram : 0
      },
      to: '',
      quantity: '',
      memo: '',
      privateKey: '',
    }
  },
  methods: {
    getAccountInfo () {
      Eos(config).getAccount(this.accountName).then(account => {
        this.result = {};
        const count = {
          stake: account.voter_info ? account.voter_info.staked / 10000 : 0,
          unStake: account.core_liquid_balance ? Number(account.core_liquid_balance.replace(' EOS', '')) : 0,
          refund: account.refund_request ? Number(account.refund_request.net_amount.replace(' EOS', '')) + Number(account.refund_request.cpu_amount.replace(' EOS', '')) : 0
        }
        count.total = count.stake + count.unStake + count.refund
        this.showInfo = true
        this.info.stake = count.stake
        this.info.unStake = count.unStake
        this.info.refund = count.refund
        this.info.total = count.total
        this.info.cpu = `${account.cpu_limit.used} up / ${account.cpu_limit.max} up (${account.total_resources.cpu_weight})`
        this.info.net = `${account.net_limit.used} bytes / ${account.net_limit.max} bytes (${account.total_resources.net_weight})`
        this.info.ram = `${account.ram_usage} bytes / ${account.ram_quota} bytes`
      }).catch(err => {
        this.result = { type : 'error', text: err.message}
      })
    },
    transfer () {
      const from = this.accountName
      const to = this.to
      const quantity = this.quantity
      const memo = this.memo
      const privateKey = this.privateKey

      if (to && quantity && privateKey) {
        this.result = {}
        config.keyProvider = privateKey
        Eos(config).transaction('eosio.token', (coin) => {
          coin.transfer(from, to, `${Number(quantity).toFixed(4)} EOS`, memo)
        })
          .then(trans => {
            this.result = { type : 'success', text: trans.transaction_id, url: `https://tools.cryptokylin.io/#/tx/${trans.transaction_id}`}
          }).catch(err => {
            this.result = { type : 'error', text: err.message}
          })
      } else {
        this.result = { type : 'error', text: '필수값을 입력해주세요'}
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
