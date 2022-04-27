<template>
  <div class="flex justify-start min-h-screen bg-gray-100 px-5 lg:px-24 py-5">
    <div class="bg-white w-full p-4">
      <h1 class="font-semibold text-base">Transactions</h1>
      <div class="mt-1 flex flex-col space-y-4 lg:flex-row lg:space-x-4">
        <div class="flex flex-col justify-end">
          <span class="text-gray-400 text-sm mb-1">Account</span>
          <select
            id="accounts"
            v-model="accountId"
            name="account"
            class="border border-gray-400 p-1.5 h-10 w-full lg:w-48"
          >
            <option value="" selected>No filter</option>
            <option
              v-for="account in accounts"
              :key="account.id"
              :value="account.id"
            >
              {{ account.name }}
            </option>
          </select>
        </div>
        <div class="flex flex-col w-full lg:w-28">
          <span class="text-gray-400 text-sm mb-1">Starting month</span>
          <input
            id="startDate"
            v-model="startDate"
            type="date"
            name="startDate"
            class="border border-gray-400 p-1.5"
          />
        </div>
        <div class="flex flex-col w-full lg:w-28">
          <span class="text-gray-400 text-sm mb-1">Ending month</span>
          <input
            id="endDate"
            v-model="endDate"
            type="date"
            name="endDate"
            class="border border-gray-400 p-1.5"
          />
        </div>
      </div>
      <div class="w-full overflow-x-auto">
        <table class="w-full table-auto mt-5">
          <thead>
            <tr class="border-b border-t text-gray-300">
              <th
                v-for="(field, idx) in fields"
                :key="idx"
                class="text-left whitespace-nowrap px-3 py-2"
                :colspan="field.span"
              >
                <span class="text-sm">{{ field.label }}</span>
                <template v-if="idx === 2">
                  <span
                    v-if="sortOrder === 'desc'"
                    @click="changeSortOrder('asc')"
                    >&uarr;</span
                  >
                  <span
                    v-if="sortOrder === 'asc'"
                    @click="changeSortOrder('desc')"
                    >&darr;</span
                  >
                </template>
              </th>
            </tr>
          </thead>
          <tbody>
            <template v-if="$apollo.queries.transactions.loading">
              <tr>
                <td colspan="4">Loading data...</td>
              </tr>
            </template>
            <template v-else>
              <template v-if="transactions">
                <tr
                  v-for="transaction in transactions"
                  :key="transaction.id"
                  class="border-b px-3 py-2 hover:bg-gray-100 cursor-pointer"
                  @click="navigate(transaction.id)"
                >
                  <td class="px-3 py-2 text-left" colspan="2">
                    <span
                      class="col-span-2"
                      :class="{ 'text-gray-300': !transaction.reference }"
                      >{{
                        transaction.reference || 'No reference provided'
                      }}</span
                    >
                  </td>
                  <td class="px-3 py-2 text-left" colspan="1">
                    <span
                      class="px-2 py-1 rounded-md"
                      :style="{
                        backgroundColor: `#${transaction.category.color}`,
                      }"
                    >
                      {{ transaction.category.name }}
                    </span>
                  </td>
                  <td class="px-3 py-2 text-left" colspan="1">
                    {{ transaction.date | formatDate }}
                  </td>
                  <td class="px-3 py-2 text-left" colspan="1">
                    {{ transaction.amount | formatAmount }}
                    <span class="text-gray-300">{{
                      transaction.currency
                    }}</span>
                  </td>
                </tr>
              </template>
              <template v-else>
                <tr>
                  <td colspan="4">No available data</td>
                </tr>
              </template>
            </template>
          </tbody>
        </table>
      </div>
      <div class="mt-5 flex">
        <button
          class="ml-auto px-4 py-3 border border-gray-100 rounded hover:bg-gray-700 visited:bg-gray-900 bg-gray-900 text-gray-100"
          type="button"
          @click="fetchMore"
        >
          More results
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import gql from 'graphql-tag'

const GET_ACCOUNTS = {
  query: gql`
    query accounts {
      accounts {
        id
        name
      }
    }
  `,
  update: (data) => data.accounts,
}

const GET_TRANSACTIONS = {
  query: gql`
    query transactions(
      $accountId: String
      $startDate: String
      $endDate: String
      $sortOrder: String
      $take: String
      $cursor: String
    ) {
      transactions(
        filters: {
          accountId: $accountId
          startDate: $startDate
          endDate: $endDate
          sortOrder: $sortOrder
          take: $take
          cursor: $cursor
        }
      ) {
        id
        reference
        date
        amount
        currency
        category {
          name
          color
        }
        account {
          id
          name
        }
      }
    }
  `,
  variables() {
    return {
      accountId: this.accountId,
      startDate: this.startDate,
      endDate: this.endDate,
      sortOrder: this.sortOrder,
      take: this.take,
      cursor: this.cursor,
    }
  },
}

export default {
  apollo: {
    accounts: GET_ACCOUNTS,
    transactions: GET_TRANSACTIONS,
  },
  filters: {
    formatDate(value) {
      if (!value) return ''

      const date = new Date(value)
      const year = date.getFullYear()
      let mm = date.getMonth() + 1
      let dd = date.getDate()

      if (dd < 10) dd = '0' + dd
      if (mm < 10) mm = '0' + mm

      return dd + '/' + mm + '/' + year
    },
    formatAmount(value) {
      if (!value) return ''
      return parseFloat(value).toFixed(2)
    },
  },
  data() {
    return {
      accountId: '',
      startDate: '',
      endDate: '',
      sortOrder: 'desc',
      take: '100',
      cursor: '',
      fields: [
        { label: 'Reference', span: 2 },
        { label: 'Category', span: 1 },
        { label: 'Date', span: 1 },
        { label: 'Amount', span: 1 },
      ],
    }
  },
  methods: {
    navigate(id) {
      this.$router.push(`/transaction/${id}`)
    },
    changeSortOrder(val) {
      this.sortOrder = val
    },
    fetchMore() {
      this.cursor = this.transactions[this.transactions.length - 1].id
    },
  },
}
</script>
