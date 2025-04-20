<script setup lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonInput, IonItem, IonList, IonLabel, IonModal, IonButton, IonButtons } from '@ionic/vue';
import { IonRadio, IonRadioGroup } from '@ionic/vue';
import { IonFab, IonFabButton, IonIcon } from '@ionic/vue';
import { IonAlert } from '@ionic/vue';

import { ref, onMounted } from 'vue';
import { Storage } from '@ionic/storage';
import { ellipse, square, triangle, appsOutline, walletOutline, add } from 'ionicons/icons';

const isAlertOpen = ref(false);
const transactionToDelete = ref<string | null>(null);

const promptDelete = (id: string) => {
  transactionToDelete.value = id;
  isAlertOpen.value = true;
};

const deleteTransaction = async () => {
  transactions.value = transactions.value.filter(
    (record) => record.id !== transactionToDelete.value
  );
  await store.set('transactions', JSON.parse(JSON.stringify(transactions.value)));
  transactionToDelete.value = null;
};

const alertButtons = [
  {
    text: 'Cancel',
    role: 'cancel',
    handler: () => {
      transactionToDelete.value = null;
    }
  },
  {
    text: 'Delete',
    role: 'destructive',
    handler: deleteTransaction
  }
];
const amount = ref('');
const date = ref('');
const category = ref();
const notes = ref('');
const isModelOpen = ref(false);

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
    id: crypto.randomUUID(),
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
  cancel()
};

const modal = ref();
const cancel = () => modal.value.$el.dismiss(null, 'cancel');

const confirm = () => {

};

const formatAmount = (amount: number) => {
  return new Intl.NumberFormat('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  }).format(amount);
};


</script>


<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title style="padding-top: 20px;">Transactions</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <!-- <h1 style="margin-left: 10px;" @click="deleteAll">Delete All</h1> -->

      <!-- <ion-button >Add New Transaction</ion-button> -->

      <ion-fab  slot="fixed" vertical="bottom" horizontal="end" id="open-modal" expand="block">
    <ion-fab-button>
      <ion-icon :icon="add"></ion-icon>
    </ion-fab-button>
  </ion-fab>

  <ion-content :fullscreen="true">
  <ion-modal ref="modal" trigger="open-modal">
    <ion-header>
      <ion-toolbar style="margin-top:20px;">
        <ion-buttons slot="start">
          <ion-button @click="cancel()">Cancel</ion-button>
        </ion-buttons>
        <!-- <ion-title>Transaction</ion-title> -->
        <ion-buttons slot="end">
          <ion-button strong @click="saveTransaction">Save</ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content fullscreen keyboard-attach :scroll-events="true">
      <ion-list class="">

        <h6 style="text-align: center;">New Transaction</h6>

        <!-- Amount -->
        <ion-item>
          <ion-label position="floating" style="margin-bottom: 10px;">Amount</ion-label>
          <ion-input 
            v-model="amount" 
            type="number" 
            inputmode="decimal" 
            placeholder="100.00">
          </ion-input>
        </ion-item>

        <!-- Category -->
        <ion-item lines="none" class="ion-margin-top">
          <ion-label class="ion-text-wrap">Category</ion-label>
        </ion-item>
        <div class="ion-padding-start">
          <ion-radio-group v-model="category" id="category">
            <ion-item lines="none">
              <ion-label>Income</ion-label>
              <ion-radio slot="start" value="income"></ion-radio>
            </ion-item>
            <ion-item lines="none">
              <ion-label>Expenses</ion-label>
              <ion-radio slot="start" value="expenses"></ion-radio>
            </ion-item>
          </ion-radio-group>
        </div>

        <!-- Date -->
        <ion-item class="ion-margin-top">
          <ion-label position="floating" style="margin-bottom: 10px;">Date</ion-label>
          <ion-input v-model="date" type="date"></ion-input>
        </ion-item>

        <!-- Notes -->
        <ion-item class="ion-margin-top">
          <ion-label position="floating" style="margin-bottom: 10px;">Notes</ion-label>
          <ion-input v-model="notes" placeholder="Enter notes..."></ion-input>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-modal>


  <ion-list v-if="transactions?.length > 0">
    <ion-item
      button
      v-for="(record, index) in transactions"
      :key="record.id"
      @click="promptDelete(record.id)"
    >
      <ion-label>
        <h2> Amount: <span :class="record.category == 'expenses' ? 'text-red' : 'text-green'">
           {{ record.category == 'expenses' ? '-' : '+' }}  {{ formatAmount(record.amount) }}
          </span>
        </h2>
        <p>Date: {{ record.date }}</p>
        <p>Category: {{ record.category }}</p>
        <p>Notes: {{ record.notes }}</p>
        <!-- <p>ID: {{ record.id }}</p> -->
      </ion-label>
    </ion-item>
  </ion-list>

  <!-- Alert Confirmation -->
  <ion-alert
    :is-open="isAlertOpen"
    header="Delete this transaction?"
    message="Once deleted, this record cannot be recovered."
    :buttons="alertButtons"
    @didDismiss="isAlertOpen = false"
  />
    </ion-content>
    </ion-content>
  </ion-page>
</template>

<style>
ion-content {
  --keyboard-offset: 0px;
  --overflow: auto;
}
.text-green {
  color: #00d336;
}
.text-red {
  color:#df1800
}
</style>