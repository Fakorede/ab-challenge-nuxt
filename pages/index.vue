<template>
  <div>
    <div v-if="!$apollo.queries.accounts.loading" class="bg-gray-100">
      <li v-for="(a, idx) in accounts" :key="idx">
        <NuxtLink
          :to="`/transaction/${a.id}`"
          class="hover:font-bold hover:text-red-200"
        >
          {{ a.name }}
        </NuxtLink>
      </li>
    </div>
    <div v-else>loading...</div>
  </div>
</template>

<script>
import gql from 'graphql-tag'

const accountId = '343db37d-804e-4892-bf7d-bc36f601706a'
const startDate = '2021-12-29'
const endDate = '2021-12-30'

const GET_ACCOUNTS = gql`
  query accounts {
    accounts {
      id
      name
    }
  }
`

const GET_TRANSACTIONS = {
  query: gql`
    query transactions(
      $accountId: String
      $startDate: String
      $endDate: String
    ) {
      transactions(
        filters: {
          accountId: $accountId
          startDate: $startDate
          endDate: $endDate
        }
      ) {
        id
        date
        accountId
      }
    }
  `,
  variables() {
    return {
      accountId: this.accountId,
      startDate: this.startDate,
      endDate: this.endDate,
    }
  },
}

export default {
  apollo: {
    accounts: {
      query() {
        return GET_ACCOUNTS
      },
      update: (data) => data.accounts,
      loadingKey: 'loading...',
    },
    transactions: GET_TRANSACTIONS,
  },
  data() {
    return {
      accountId: '' || accountId,
      startDate: '' || startDate,
      endDate: '' || endDate,
    }
  },
}
</script>
