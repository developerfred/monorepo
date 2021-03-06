<template>
  <div class="cart">
    <div v-for="item in cart" class="cart-item" :key="item.address">
      <div class="project">
        <img class="project-image" :src="item.imageUrl" :alt="item.name">
        <div class="project-name">{{ item.name }}</div>
      </div>
      <form class="contribution-form">
        <input
          :value="item.amount"
          @input="updateAmount(item, $event.target.value)"
          class="input contribution-amount"
          name="amount"
          placeholder="Amount"
        >
        <div class="contribution-currency">{{ tokenSymbol }}</div>
        <div class="remove-cart-item" @click="removeItem(item)">
          <img src="@/assets/remove.svg" />
        </div>
      </form>
    </div>
    <div class="contribute-btn-wrapper">
      <button
        v-if="this.cart.length > 0"
        class="btn contribute-btn"
        :disabled="!canContribute()"
        @click="contribute()"
      >
        Contribute {{ total }} {{ tokenSymbol }} to {{ cart.length }} projects
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'
import { DateTime } from 'luxon'

import ContributionModal from '@/components/ContributionModal.vue'

import { CartItem } from '@/api/contributions'
import {
  ADD_CART_ITEM,
  UPDATE_CART_ITEM,
  REMOVE_CART_ITEM,
} from '@/store/mutation-types'

const CART_STORAGE_KEY = 'clrfund-cart'

@Component({
  watch: {
    cart(items: CartItem[]) {
      // Save cart to local storage on changes
      const storage = window.localStorage
      storage.setItem(CART_STORAGE_KEY, JSON.stringify(items))
    },
  },
})
export default class Cart extends Vue {

  mounted() {
    // Restore cart from local storage
    const storage = window.localStorage
    const serializedCart = storage.getItem(CART_STORAGE_KEY)
    if (serializedCart) {
      for (const item of JSON.parse(serializedCart)) {
        this.$store.commit(ADD_CART_ITEM, item)
      }
    }
  }

  get tokenSymbol(): string {
    const currentRound = this.$store.state.currentRound
    return currentRound ? currentRound.nativeTokenSymbol : ''
  }

  get cart(): CartItem[] {
    return this.$store.state.cart
  }

  updateAmount(item: CartItem, value: string) {
    if (value) {
      const amount = parseFloat(value)
      this.$store.commit(UPDATE_CART_ITEM, { ...item, amount })
    }
  }

  removeItem(item: CartItem) {
    this.$store.commit(REMOVE_CART_ITEM, item)
  }

  canContribute(): boolean {
    const currentRound = this.$store.state.currentRound
    return (
      currentRound &&
      DateTime.local() < currentRound.contributionDeadline &&
      this.$store.state.account &&
      this.$store.state.contribution.isZero() &&
      this.cart.length > 0
    )
  }

  get total(): number {
    return this.cart.reduce((acc: number, item: CartItem) => {
      return acc + item.amount
    }, 0)
  }

  contribute() {
    this.$modal.show(
      ContributionModal,
      { },
      {
        clickToClose: false,
        height: 'auto',
        width: 450,
      },
    )
  }
}
</script>

<style scoped lang="scss">
@import '../styles/vars';

.cart {
  display: flex;
  flex-direction: column;
  height: calc(100vh - 110px);
}

.cart-item {
  border-bottom: $border;
  padding: $content-space;
}

$project-image-size: 50px;

.project {
  display: flex;
  flex-direction: row;

  .project-image {
    border-radius: 10px;
    box-sizing: border-box;
    display: block;
    height: $project-image-size;
    margin-right: 15px;
    min-width: $project-image-size;
    object-fit: cover;
    width: $project-image-size;
  }

  .project-name {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    flex-grow: 1;
    height: $project-image-size;
    overflow: hidden;
    text-overflow: ellipsis;
  }
}

.contribution-form {
  align-items: center;
  display: flex;
  flex-direction: row;
  font-size: 12px;
  margin-top: 15px;
  padding-left: $project-image-size + 15px;

  .contribution-amount {
    font-size: 12px;
    width: 60%;
  }

  .contribution-currency {
    flex-grow: 1;
    margin-left: 7px;
  }

  .remove-cart-item {
    cursor: pointer;
    margin-left: 7px;
    min-width: 20px;
    width: 20px;

    &:hover {
      filter: saturate(0%);
    }
  }
}

.contribute-btn-wrapper {
  align-self: flex-end;
  box-sizing: border-box;
  margin-top: auto;
  padding: $content-space;
  width: 100%;

  .contribute-btn {
    width: 100%;
  }
}
</style>
