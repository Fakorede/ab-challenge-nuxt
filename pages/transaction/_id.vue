<template>
  <div class="flex justify-start min-h-screen bg-gray-100 px-5 lg:px-24 py-5">
    <div v-if="$apollo.queries.transaction.loading">Loading Data...</div>
    <div v-else class="bg-white w-full p-4">
      <h1 class="font-semibold text-base">Transactions</h1>
      <div class="mt-1 flex flex-col space-y-4">
        <ul v-if="transaction">
          <li>ID: {{ $route.params.id }}</li>
          <li>Reference: {{ transaction.reference }}</li>
          <li>Amount: {{ transaction.currency }} {{ transaction.amount }}</li>
          <li>Account: {{ transaction.account.name }}</li>
          <li>Category: {{ transaction.category.name }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import gql from 'graphql-tag'

const GET_TRANSACTION = {
  query: gql`
    query transaction($transactionId: String) {
      transaction(transactionId: $transactionId) {
        reference
        amount
        currency
        account {
          name
        }
        category {
          name
        }
      }
    }
  `,
  update: (data) => data.transaction,
  variables() {
    return {
      transactionId: this.$route.params.id,
    }
  },
}

export default {
  apollo: {
    transaction: GET_TRANSACTION,
  },
  data() {
    return {}
  },
}
</script>
