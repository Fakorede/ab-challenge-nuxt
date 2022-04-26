<template>
  <div>
    <p>single page</p>
  </div>
</template>

<script>
import gql from 'graphql-tag'

const GET_TRANSACTION = {
  query: gql`
    query transaction($accountId: String) {
      transactions(accountId: $accountId) {
        id
        date
        accountId
      }
    }
  `,
  variables() {
    return {
      accountId: this.$route.params.id,
    }
  },
}

export default {
  apollo: {
    transactions: GET_TRANSACTION,
  },
  data() {
    return {}
  },
}
</script>
