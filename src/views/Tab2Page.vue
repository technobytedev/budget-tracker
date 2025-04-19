<script setup lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonInput, IonItem, IonList, IonLabel } from '@ionic/vue';
import { IonRadio, IonRadioGroup } from '@ionic/vue';
import { ref, onMounted } from 'vue';
import { Storage } from '@ionic/storage';

const amount = ref('');
const date = ref('');
const category = ref();
const notes = ref('');

const store = new Storage();
const transactions = ref<any[]>([]);

onMounted(async () => {
  await store.create();
  //   await store.remove('transactions');

  // // Optional: set default
  // transactions.value = [];
  const stored = await store.get('transactions');
  if (Array.isArray(stored)) {
    transactions.value = stored;
  }
});

async function deleteAll() {
  await store.remove('transactions');
  transactions.value = [];
  
}

const saveTransaction = async () => {

  if (!amount.value || !date.value || !document.getElementById('category').value || !notes.value) {
    console.warn('Amount and Date are required.');
    return;
  }


  const newRecord = {
    amount: parseFloat(parseFloat(amount.value).toFixed(2)),
    date: date.value,
    category: category.value ?? document.getElementById('category').value,
    notes: notes.value
  };


  transactions.value.push(newRecord);

  // ✅ Store a plain array, not the reactive Proxy
  await store.set('transactions', JSON.parse(JSON.stringify(transactions.value)));

  // Reset form
  amount.value = '';
  date.value = '';
  category.value = '';
  notes.value = '';
};

</script>


<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Transactions</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Add Transaction </ion-title>
        </ion-toolbar>
      </ion-header>
      <h1 style="margin-left: 10px;" @click="deleteAll">Delete All</h1>
      <ion-list class="page-wrapper">
      
        <ion-item>
          <ion-input  
          v-model="amount" 
          type="number"
          step="0.01" 
          label="Amount" 
          placeholder="100.00"
          ></ion-input>
          <ion-radio-group v-model="category" id="category">
            <ion-item>
              <ion-label>Income</ion-label>
              <ion-radio slot="start" value="income"></ion-radio>
            </ion-item>
            <ion-item>
              <ion-label>Expenses</ion-label>
              <ion-radio slot="start" value="expenses"></ion-radio>
            </ion-item>
          </ion-radio-group>

        </ion-item>
        <ion-item>
          <ion-input v-model="date" type="date" label="Date" placeholder=""></ion-input>
        </ion-item>
        <ion-item>
          <ion-input v-model="notes" label="Notes" placeholder="Notes.."></ion-input>
        </ion-item>
        <ion-button @click="saveTransaction" style="width: 100%;">Save</ion-button>
      </ion-list>

      <ion-list v-if="transactions.length > 0">
        <ion-item v-for="(record, index) in transactions" :key="index">
          <ion-label>
            <h2>Amount: {{ record.amount.toFixed(2) }}</h2>
            <p>Date: {{ record.date }}</p>
            <p>Category: {{ record.category }}</p>
            <p>Notes: {{ record.notes }}</p>
          </ion-label>
        </ion-item>
      </ion-list>

    </ion-content>
  </ion-page>
</template>

<style>
.input-wrapper .sc-ion-input-ios {
  width: 110px!important; /* Adjust this value to fit your design */
}
.page-wrapper {
  margin: 5px;
  border: 1px solid black;
  padding: 8px;
}
</style>