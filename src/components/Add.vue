<template v-if="isLoggedIn">
  <div id="dashboard">
    <HeaderTemplate/>

    <div class="box p-5 m-4">
      <h1 class="text-center">Fund my project</h1>
      <p class="text-center text-clear font-weight-500">First, lets select a category</p>
      <div class="d-flex align-items-center justify-content-center p-5">
        <div v-for="(category, i) in categories" width="226px" class="category d-flex flex-column align-items-center px-5 pt-5 pb-3 mx-3" :class="{active: i === selectedCategory}" @click="selectCategory(i)">
          <img :src="category.image" alt="" width="80px">
          <h4 class="text-center mt-4">{{ category.title }}</h4>
          <p class="text-center mt-4">Funding closes in <br/></p>
          <vac :end-time="category.endDate">
            <p
              class="text-center"
              slot="process"
              slot-scope="{ timeObj }">{{ `${ timeObj.d }days, ${timeObj.m}:${timeObj.s}` }}</p>
            <p slot="finish">Ended !</p>
          </vac>
        </div>
      </div>

      <div class="form--one-line d-flex align-items-center flex-column mx-5 mb-5">
        <input type="text" v-model="newForm.title" placeholder="Title">
        <input type="text" v-model="newForm.description" placeholder="Description">
        <input type="text" v-model="newForm.minAmount" placeholder="Minimum amount">
        <!-- <input type="text" v-model="newForm.image"> -->
      </div>

      <div class="d-flex justify-content-center">
        <button class="form-btn" @click="addProject">NEXT</button>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex"
import contract from 'truffle-contract'
import FundingToken from '@/../build/contracts/FundingToken.json'

export default {
  name: 'Dashboard',
  components: {
    HeaderTemplate,
  },
  computed: {
    ...mapState({
      categories: state => state.categories
    })
  },
  props: {
    currentView: {
      type: Object,
      default: null
    }
  },
  data: function () {
    return {
      selectedCategory: 1,
      newForm: {
        title: null,
        description: null,
        image: null,
        minAmount: 0
      },
    }
  },
  methods: {
    selectCategory(i) {
      this.selectedCategory = i;
    },
    addProject() {
      if (this.$store.state.web3.instance) {
        const web3 = this.$store.state.web3.instance()
        const fundingContract = new web3.eth.Contract(FundingToken.abi, FundingToken.address)
        web3.eth.getAccounts().then(accounts => {
          fundingContract.methods.newProject(this.newForm.minAmount * 100000000000000, this.selectedCategory).send({ from: accounts[0] }).then(ret => {
            this.$toasted.show('Sent', {type: 'success', position: 'bottom-center'})
          }).catch(err => {
            console.log(err);
            this.$toasted.show('Error', {type: 'error', position: 'bottom-center'})
          })
        })
      }
    }
  },
}

import HeaderTemplate from './layout/HeaderTemplate'
</script>

<style scoped>
  #dashboard {
    width: 100%;
  }
  .category {
    border-radius: 4px;
    border: 1px solid rgb(230, 235, 237);
    cursor: pointer;
  }
  .category:hover {
    background-color: rgba(36, 161, 241, 0.07);
  }
  .category.active {
    border-color: rgb(0, 102, 255) !important;
  }
  .form--one-line input {
      border: 0;
      border-bottom: 1px solid rgb(230, 227, 227);
      outline: none;
      padding: 10px 0;
      font-size: 1.1rem;
      font-weight: 500;
      color: rgb(41, 41, 41);
      width: 100%;
      max-width: 500px;
  }

</style>
