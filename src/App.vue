<template>
  <Header />
  <div class="container">
    <Balance v-bind:total="+total"/>
    <!-- add "+" to set type as number -->
    <IncomeExpensive v-bind:income="+income" v-bind:expenses="+expenses"/>
    <TransactionList v-bind:transactions="transactions" @transactionDeleted="handleTransactionDeleted"/>
    <AddTransavtion @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>

<script setup>
 import Header from './components/Header.vue';
 import Balance from './components/Balance.vue';
 import IncomeExpensive from './components/IncomeExpensive.vue';
 import TransactionList from './components/TransactionList.vue';
 import AddTransavtion from './components/AddTransavtion.vue';
 import {useToast} from 'vue-toastification';

// ref make variable reactive
 import { ref, computed, onMounted } from 'vue';

const toast = useToast();

const transactions = ref([]);

onMounted(()=>{
  const saveTransactions = JSON.parse(localStorage.getItem('transactions'));

  if(saveTransactions){
    transactions.value = saveTransactions;
  }
});

// Get total
const total= computed(()=>{
  // reduce function can return accumulated value
  // acc set as 0
  return transactions.value.reduce((acc, transaction) =>{
    return acc + transaction.amount;
  }, 0);
});

// Get Income
// filter amount>0 then accumulate
// fix income to 2 decimals places
const income = computed(()=>{
  return transactions.value
  .filter((transaction)=> transaction.amount > 0)
  .reduce((acc, transaction)=>{
    return acc + transaction.amount;
  }, 0)
  .toFixed(2);
});

// Get Expenses
const expenses = computed(()=>{
  return transactions.value
  .filter((transaction)=> transaction.amount < 0)
  .reduce((acc, transaction)=>{
    return acc + transaction.amount;
  }, 0)
  .toFixed(2);
});

// Add Transaction
const handleTransactionSubmitted = (transactionData)=>{
  transactions.value.push({
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount,
  });

  saveTransactionsToLocalStorage();

  toast.success('Transaction added!');

};

// Generate Unique ID
const generateUniqueId = ()=>{
  return Math.floor(Math.random() * 1000000);
}

// Delete Transaction
const handleTransactionDeleted = (id) =>{
  transactions.value = transactions.value.filter(
    (transaction)=> transaction.id !== id
    );

    saveTransactionsToLocalStorage();

    toast.success('Transaction deleted!')
}

// Save to local storage
const saveTransactionsToLocalStorage = ()=>{
  localStorage.setItem('transactions', JSON.stringify(transactions.value));
};
</script>
