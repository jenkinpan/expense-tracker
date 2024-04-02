<template>
  <Header />
  <div class="container">
    <Balance :total="+total" />
    <IncomeExpenses :income="+income" :expenses="+expenses" />
    <TransactionList
      :transactions="transactions"
      @transactionDeleted="handleDeleted"
    />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import { useToast } from "vue-toastification";
import Header from "./components/Header.vue";
import Balance from "./components/Balance.vue";
import IncomeExpenses from "./components/IncomeExpenses.vue";
import TransactionList from "./components/TransactionList.vue";
import AddTransaction from "./components/AddTransaction.vue";

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
  const storedTransactions = JSON.parse(localStorage.getItem("transactions"));
  if (storedTransactions) {
    transactions.value = storedTransactions;
  }
});

// get total balance
const total = computed(() => {
  return transactions.value.reduce((acc, curr) => {
    return (acc += curr.amount);
  }, 0);
});

// get total income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, curr) => {
      return (acc += curr.amount);
    }, 0)
    .toFixed(2);
});

// get total expenses
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, curr) => {
      return (acc += curr.amount);
    }, 0)
    .toFixed(2);
});

// Add transaction
const handleTransactionSubmitted = (transaction) => {
  transactions.value.push({
    id: generateUniqueId(),
    text: transaction.text,
    amount: transaction.amount,
  });

  saveTransactions();

  toast.success("Transaction added successfully");
};

// Generate unique id
const generateUniqueId = () => {
  return Math.floor(Math.random() * 1000000);
};

// Delete transaction
const handleDeleted = (id) => {
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== id,
  );

  saveTransactions();

  toast.success("Transaction deleted successfully");
};

// Save transactions
const saveTransactions = () => {
  localStorage.setItem("transactions", JSON.stringify(transactions.value));
};
</script>
