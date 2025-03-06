<template>
  <Header />
  <div class="container">
    <Balance :total="+total" />
   <IncomeExpenses :income="+income" :expense="+expense" />
   <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted" />
   <AddTransaction  @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import AddTransaction from './components/AddTransaction.vue'
import Balance from './components/Balance.vue'
import Header from './components/Header.vue'
import IncomeExpenses from './components/IncomeExpenses.vue'
import TransactionList from './components/TransactionList.vue'

import { useToast } from 'vue-toastification'

const transactions = ref([ ])

const toast = useToast()

onMounted(() => {
  const savedTransaction = JSON.parse(localStorage.getItem('transactions'))
  if(savedTransaction) {
    transactions.value = savedTransaction
  }
})
// Get total
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
   return acc + transaction.amount
  }, 0)
})

//Get income
const income = computed(() => {
  return transactions.value.filter(({ amount }) => amount > 0)
  .reduce((acc, transaction) => {
    return acc + transaction.amount 
  }, 0).toFixed(2)
})

//Get expense
const expense = computed(() => {
  return transactions.value.filter(({ amount }) => amount < 0)
  .reduce(( acc, transaction) => {
    return acc + transaction.amount
  }, 0).toFixed(2)
})

const handleTransactionSubmitted = (transactionData) => {
  transactions.value.push({
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount
  })
  saveTransactionToLocalStorage()
  toast.success('Transaction added')
}


const generateUniqueId = () => {
  return Math.floor(Math.random() * 1000000)
}

const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter((transaction) => transaction.id !== id)
  saveTransactionToLocalStorage()
  toast.success('Transaction deleted')
}

//save to localeStorage
const saveTransactionToLocalStorage = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value))
}

</script>