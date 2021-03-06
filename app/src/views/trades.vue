<template>
  <div>
    <h1>Trade Requests</h1>
    <br/>
    <div class="progress progress-striped active" v-if="loading">
      <div class="progress-bar" style="width: 100%"></div>
    </div>
    <template v-else>
      <h2>Requests for you: </h2>
      <p>You can accept or decline requests. </p>
      <template v-if="incomingTrades.length > 0">
        <div class="row">
          <div class="col-md-2 col-sm-3 col-xs-6" v-for="trade in incomingTrades" :key="trade._id">
            <div class="book">
              <div class="controls">
                <i class="fa fa-check text-success" aria-hidden="true" @click="updateTrade(trade, true)" title="Approve"></i>
                <i class="fa fa-ban text-danger" aria-hidden="true" @click="updateTrade(trade, false)" title="Reject"></i>
              </div>
              <img alt="thumbnail" :title="trade.book.title" :src="trade.book.thumbnail" />
            </div>
          </div>
        </div>
      </template>
      <template v-else>
        <div class="alert alert-info">
          <p>No incoming book requests.</p>
        </div>
      </template>
      <hr/>
      <h2>Your requests: </h2>
      <p>Your pending requests. </p>
      <template v-if="outgoingTrades.length > 0">
        <div class="row">
          <div class="col-md-2 col-sm-3 col-xs-6" v-for="trade in outgoingTrades" :key="trade._id">
            <div class="book">
              <div class="controls">
                <i class="fa fa-trash text-danger" aria-hidden="true" @click="cancelTrade(trade)" title="Cancel"></i>
              </div>
              <img alt="thumbnail" :title="trade.book.title" :src="trade.book.thumbnail" />
            </div>
          </div>
        </div>
      </template>
      <template v-else>
        <div class="alert alert-info">
          <p>You have no pending requests.</p>
        </div>
      </template>
      <hr/>
      <h2>Completed Trades: </h2>
      <p>All completed trades will show here.</p>
      <template v-if="completedRequests.length > 0">
        <div class="row">
          <div class="col-md-2 col-sm-3 col-xs-6" v-for="trade in completedRequests" :key="trade._id">
            <div class="book">
              <div class="controls">
                <i class="small text-success" :class="{'text-danger': !getTradeTitle(trade).success}" aria-hidden="true">{{getTradeTitle(trade).message}}</i>
              </div>
              <img alt="thumbnail" :title="trade.book.title" :src="trade.book.thumbnail" />
            </div>
            <p v-if="trade.owner._id == getUser._id">
              <span>
                <strong>Contact Number: </strong>{{trade.trader.contact_number}}</span><br/>
              <span>
                <strong>Name: </strong>{{trade.trader.first_name + ' ' + trade.trader.last_name}}</span><br/>
              <span>
                <strong>Adress: </strong>{{trade.trader.city + ', ' + trade.trader.state}}</span><br/>
            </p>
            <p v-else>
              <span>
                <strong>Contact Number: </strong>{{trade.owner.contact_number}}</span><br/>
              <span>
                <strong>Name: </strong>{{trade.owner.first_name + ' ' + trade.owner.last_name}}</span><br/>
              <span>
                <strong>Adress: </strong>{{trade.owner.city}} {{trade.owner.state}}</span><br/>
            </p>
          </div>
        </div>
      </template>
      <template v-else>
        <div class="alert alert-info">
          <p>You have no requests completed. </p>
        </div>
      </template>
    </template>
  </div>
</template>

<script>
  import router from '../utilities/router'
  import { mapGetters } from 'vuex'
  import axios from 'axios'

  export default {
    data () {
      return {
        trades: [],
        loading: true
      }
    },
    computed: {
      ...mapGetters(['getAPI', 'getUser']),
      incomingTrades () {
        return this.trades.filter((item) => {
          return item.owner._id === this.getUser._id && !item.isCompleted
        })
      },
      outgoingTrades () {
        return this.trades.filter((item) => {
          return item.trader._id === this.getUser._id && !item.isCompleted
        })
      },
      completedRequests () {
        return this.trades.filter((item) => {
          return item.isCompleted
        })
      }
    },
    methods: {
      getTradeTitle (trade) {
        let id = this.getUser._id
        if (trade.isApproved) {
          if (trade.owner._id === id) {
            return {
              success: true,
              message: 'Approved by you'
            }
          } else {
            return {
              success: true,
              message: 'Approved by ' + trade.owner.username
            }
          }
        } else {
          if (trade.owner._id === id) {
            return {
              success: false,
              message: 'Rejected by you'
            }
          } else {
            return {
              success: false,
              message: 'Rejected by ' + trade.owner.username
            }
          }
        }
      },
      updateTrade (trade, approved) {
        axios.put(this.getAPI.url + '/api/trades/', {
          trade: trade,
          isApproved: approved
        }).then((res) => {
          if (res.data.redirect) {
            router.push(res.data.redirect)
          } else {
            trade.isApproved = approved
            trade.isCompleted = true
          }
        }).catch(err => {
          console.log(err)
        })
      },
      cancelTrade (trade) {
        axios.delete(this.getAPI.url + '/api/trades/' + trade._id).then(() => {
          this.trades = this.trades.filter((item) => {
            return item._id !== trade._id
          })
        }).catch(err => {
          console.log(err)
        })
      },
      getTrades () {
        axios.get(this.getAPI.url + '/api/trades/').then(res => {
          this.trades = res.data
        }).catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
      }
    },
    mounted () {
      this.getTrades()
    }
  }
</script>

<style>

</style>
