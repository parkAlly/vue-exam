<template>
<div>
  <div class="input-group mb-3 p-2">
    <input type="text" class="form-control" id="accountName" placeholder="EOS Account Name" v-model="accountName">
    <div class="input-group-append">
      <button class="btn btn-outline-secondary" type="button" id="button-addon2" @click="getAccountInfo()">조회</button>
    </div>
  </div>
  <div class="row" id="info" v-if="info">
    <h2 class="col-12 text-center" id="account">
      EOS ACCOUNT
    </h2>
    <div class="col-12 row">
      <div class="col-6">
        <div class="row p-2">
          <div class="col-3">TOTAL(EOS)</div>
          <div class="col-9" id="total">{{total}}</div>
        </div>
        <div class="row p-2">
          <div class="col-3">UnStake(EOS)</div>
          <div class="col-9" id="unStake">{{unStake}}</div>
        </div>
        <div class="row p-2">
          <div class="col-3">Stake(EOS)</div>
          <div class="col-9" id="stake">{{stake}}</div>
        </div>
        <div class="row p-2">
          <div class="col-3">Refund(EOS)</div>
          <div class="col-9" id="refund">{{refund}}</div>
        </div>
        <div class="row p-2">
          <div class="col-3">CPU(up)</div>
          <div class="col-9" id="cpu">{{cpu}}</div>
        </div>
        <div class="row p-2">
          <div class="col-3">NET(byte)</div>
          <div class="col-9" id="net">{{net}}</div>
        </div>
        <div class="row p-2">
          <div class="col-3">RAM(byte)</div>
          <div class="col-9" id="ram">{{ram}}</div>
        </div>
      </div>
      <div class="col-6">
        <form>
          <div class="form-group">
            <label for="exampleInputEmail1">상대계정</label>
            <input type="text" class="form-control" id="to" aria-describedby="emailHelp" placeholder="받는계정" v-model="to">
          </div>
          <div class="form-group">
            <label for="exampleInputPassword1">수량</label>
            <input type="number" class="form-control" id="quantity" placeholder="수량" v-model="quantity">
          </div>
          <div class="form-group">
            <label for="exampleInputPassword1">메모</label>
            <textarea class="form-control" id="memo" placeholder="memo" v-model="memo"></textarea>
          </div>
          <div class="form-group">
            <label for="exampleInputPassword1">privateKey</label>
            <input type="text" class="form-control" id="privateKey" placeholder="privateKey" v-model="privateKey">
          </div>
          <button type="button" @click="transfer()" class="btn btn-primary">Submit</button>
        </form>
      </div>
    </div>
  </div>

  <div class="text-danger" id="error" v-if="error">
    <h3>ERROR :( </h3>
    <div id="errorText"> {{errorText}} </div>
  </div>
  <div class="text-success" id="success" v-if="success">
    <h3>SUCCESS :) </h3>
    <a v-bind:href="successUrl">{{successText}}</a>
  </div>

</div>
</template>

<script>

const Eos = require('eosjs')
const config = {
  httpEndpoint: 'https://api-kylin.eosasia.one',
  chainId: '5fff1dae8dc8e2fc4d5b23b2c7665c97f9e9d8edf2b6485a86ba311c25639191'
}

export default {
  name: 'main',
  data: function () {
    return {
      accountName: '',
      info: false,
      error: false,
      errorText: '',
      success: false,
      successUrl: '',
      successText: '',
      account: '',
      stake: '',
      unStake: '',
      refund: '',
      total: '',
      cpu: '',
      net: '',
      ram: '',
      to: '',
      quantity: '',
      memo: '',
      privateKey: ''
    }
  },
  methods: {
    getAccountInfo () {
      Eos(config).getAccount(this.accountName).then(account => {
        const count = {
          stake: account.voter_info ? account.voter_info.staked / 10000 : 0,
          unStake: account.core_liquid_balance ? Number(account.core_liquid_balance.replace(' EOS', '')) : 0,
          refund: account.refund_request ? Number(account.refund_request.net_amount.replace(' EOS', '')) + Number(account.refund_request.cpu_amount.replace(' EOS', '')) : 0
        }
        count.total = count.stake + count.unStake + count.refund
        this.info = true
        this.stake = count.stake
        this.unStake = count.unStake
        this.refund = count.refund
        this.total = count.total
        this.cpu = `${account.cpu_limit.used} up / ${account.cpu_limit.max} up (${account.total_resources.cpu_weight})`
        this.net = `${account.net_limit.used} bytes / ${account.net_limit.max} bytes (${account.total_resources.net_weight})`
        this.ram = `${account.ram_usage} bytes / ${account.ram_quota} bytes`
      }).catch(err => {
        this.info = false
        this.error = true
        this.errorText = err.message
      })
    },
    transfer () {
      const from = this.accountName
      const to = this.to
      const quantity = this.quantity
      const memo = this.memo
      const privateKey = this.privateKey

      if (to && quantity && privateKey) {
        this.error = false
        config.keyProvider = privateKey
        Eos(config).transaction('eosio.token', (coin) => {
          coin.transfer(from, to, `${Number(quantity).toFixed(4)} EOS`, memo)
        })
          .then(trans => {
            this.success = true
            this.successUrl = `https://tools.cryptokylin.io/#/tx/${trans.transaction_id}`
            this.successText = trans.transaction_id
          }).catch(err => {
            this.error = true
            this.errorText = err.message
          })
      } else {
        this.error = true
        this.errorText = '필수값을 입력해주세요'
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
