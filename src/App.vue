<template>
  <Header></Header>
  <div class="container">
    <Balance :total="total"/>
    <IncomeExpenses :income="+income" :expenses="+expenses"/>
    <TransactionList :transactions="transactions" @transactionDelete="handleTransactionDelete"/>
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>

<script setup>
  import Header from './components/Header.vue';
  import Balance from './components/Balance.vue';
  import IncomeExpenses from './components/IncomeExpenses.vue';
  import TransactionList from './components/TransactionList.vue';
  import AddTransaction from './components/AddTransaction.vue';

  import { computed, onMounted, ref } from "vue"
  import { toast } from 'vue3-toastify';

  const transactions = ref([]);

  onMounted(() => {
    const saveTransactions = JSON.parse(localStorage.getItem(transactions));
    
    if (saveTransactions) {
      transactions.value = saveTransactions;
    }
  });

  const total = computed(() => {
    return transactions.value.reduce((accumulator, transaction) => {
      return accumulator + transaction.amount;
    }, 0);
  });

  const income = computed(() => {
    return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((accumulator, transaction) => {
      return accumulator + transaction.amount;
    }, 0)
    .toFixed(2);
  });

  const expenses = computed(() => {
    return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((accumulator, transaction) => {
      return accumulator + transaction.amount;
    }, 0)
    .toFixed(2);
  });

  const handleTransactionSubmitted = (transactionData) => {
    let esixtId = true;
    let count = 0;
    var id = 0;

    while (esixtId) {
      id = generateUniqueId();
      esixtId = transactions.value.some((transaction) => transaction.id == id);

      count++;
      if(count > 10) {
        toast.error('Add transaction not success!', {
          autoClose: 2000,
        });
        return;
      }
    }
    transactions.value.push({
      id: id,
      text: transactionData.text,
      amount: transactionData.amount,
    });

    saveTransactionsToLocalStorage();
    toast.success("Add transaction suceess!", {
      autoClose: 1000,
    });
  };

  function generateUniqueId() {
    return Math.floor(Math.random() * 100000000)
  }

  function handleTransactionDelete(id) {
    transactions.value = transactions.value.filter((transaction) => transaction.id != id);
    saveTransactionsToLocalStorage();
    toast.success("Delete transaction suceess!", {
      autoClose: 1000,
    });
  }

  const saveTransactionsToLocalStorage = () => {
    localStorage.setItem('transactions', JSON.stringify(transactions.value));
  }

</script>
