<script setup lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonInput, IonItem, IonList, IonLabel, IonModal, IonButton, IonButtons } from '@ionic/vue';
import { IonRadio, IonRadioGroup } from '@ionic/vue';
import { IonFab, IonFabButton, IonIcon, IonDatetime, IonDatetimeButton } from '@ionic/vue';
import {  IonSelect, IonSelectOption } from '@ionic/vue';
import { IonToast } from '@ionic/vue';


import { IonAlert } from '@ionic/vue';

import { ref, onMounted, computed, watch } from 'vue';
import { Storage } from '@ionic/storage';
import { ellipse, square, triangle, appsOutline, walletOutline, add } from 'ionicons/icons';

const isAlertOpen = ref(false);
const transactionToDelete = ref<string | null>(null);

const promptDelete = (id: string) => {
  transactionToDelete.value = id;
  isAlertOpen.value = true;
};

const deleteTransaction = async () => {
  // Delete from full list
  allTransactions.value = allTransactions.value.filter(
    (record: any) => record.id !== transactionToDelete.value
  );

  // Apply filter to update current view
  filterTransactionsByMonthYear();

  // Save to storage
  await store.set('transactions', JSON.parse(JSON.stringify(allTransactions.value)));

  transactionToDelete.value = null;
  isToastOpen.value = true;
  message.value = 'Transaction deleted successfully.';
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
const message = ref('');
const amount = ref('');
const date = ref(new Date().toISOString()); // ISO 8601 format
const monthYear = ref(new Date().toISOString()); // ISO 8601 format
const isToastOpen = ref(false);

const category = ref();
const notes = ref('');
const isModelOpen = ref(false);
const allTransactions = ref()

const store = new Storage();
const transactions = ref<any[]>([]);

onMounted(async () => {
  await store.create();

  const stored = await store.get('transactions');
  if (Array.isArray(stored)) {
  allTransactions.value = stored;
} else {
  allTransactions.value = [];
}
});

// Watch for changes to monthYear
watch(monthYear, () => {
  filterTransactionsByMonthYear();
});

function filterTransactionsByMonthYear() {
  const monthYearStr = monthYear.value.slice(0, 7); // "YYYY-MM"
  const [year, month] = monthYearStr.split('-').map(Number);

  const filtered = allTransactions.value.filter(t => {
    const date = new Date(t.date);
    return date.getFullYear() === year && date.getMonth() === month - 1;
  });

  transactions.value = filtered;
}


function formatDateOnly(dateString: string): string {
  if (!dateString) return ''
  const d = new Date(dateString)
  return d.toLocaleDateString(undefined, {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

async function deleteAll() {
  await store.remove('transactions');
  transactions.value = [];
  
}

const saveTransaction = async () => {

  if (!amount.value || !date.value || !category.value || !notes.value) {
    console.warn('Amount and Date are required.');
    return;
  }


  const newRecord = {
    id: crypto.randomUUID(),
    amount: parseFloat(parseFloat(amount.value).toFixed(2)),
    date: date.value,
    category: category.value,
    notes: notes.value
  };

console.log(newRecord)

allTransactions.value.push(newRecord);

  // ✅ Store a plain array, not the reactive Proxy
  await store.set('transactions', JSON.parse(JSON.stringify(allTransactions.value)));

  isToastOpen.value = true;
  message.value = 'Transaction addedd successfully.'

  // Reset form
  amount.value = '';
  date.value = new Date().toISOString()
  category.value = '';
  notes.value = '';
  filterTransactionsByMonthYear();
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
      <ion-fab slot="fixed" vertical="bottom" horizontal="end" id="open-modal" expand="block">
        <ion-fab-button class="appPrimary">
          <ion-icon :icon="add"></ion-icon>
        </ion-fab-button>
      </ion-fab>

  <ion-content :fullscreen="true" scroll-y keyboard-attach style="margin: 10px!important;">
    <ion-datetime-button  datetime="topdatetime" presentation="month-year" style="margin-bottom: 10px;margin-left: 0px;margin-top: 10px; "  />


      <ion-toast
      :is-open="isToastOpen"
      :message="message"
      :duration="2000"
      @didDismiss="isToastOpen = false">
    </ion-toast>

  <!-- Modal with Month Picker Only -->
  <ion-modal keep-contents-mounted >
    <ion-datetime
      id="topdatetime"
      v-model="monthYear"
      presentation="month-year"
      show-default-buttons
    ></ion-datetime>
  </ion-modal>
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

    <ion-content  style="height:120vh!important; overflow: auto!important;">
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
        <ion-item class="ion-margin-top" >
          <ion-label position="floating"  style="margin-bottom: 10px;">Category</ion-label>
          <ion-select v-model="category" interface="popover" placeholder="Select Category">
            <ion-select-option value="income">Income</ion-select-option>
            <ion-select-option value="expenses">Expenses</ion-select-option>
          </ion-select>
        </ion-item>

<!-- Date -->
<ion-item class="ion-margin-top"> 
  <ion-label position="floating" style="margin-bottom: 25px;">Date:</ion-label>

  <!-- Date Picker Button -->
  <ion-datetime-button datetime="datetime" presentation="date" style="margin-bottom: 20px;" />

  <!-- Modal with Only Date Picker -->
  <ion-modal keep-contents-mounted>
    <ion-datetime
      id="datetime"
      presentation="date"
      v-model="date"
      show-default-buttons
    ></ion-datetime>
  </ion-modal>
</ion-item>


        <!-- Notes -->
        <ion-item class="ion-margin-top">
          <ion-label position="floating" style="margin-bottom: 10px;">Notes</ion-label>
          <ion-input v-model="notes" placeholder="Enter notes..."></ion-input>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-modal>


  <ion-list v-if="allTransactions?.length > 0">
    <ion-item
      button
      v-for="(record, index) in allTransactions"
      :key="record.id"
      @click="promptDelete(record.id)"
    >
      <ion-label style="margin: 5px!important;">
        <h2> Amount: <span :class="record.category == 'expenses' ? 'text-red' : 'text-green'">
           {{ record.category == 'expenses' ? '-' : '+' }}  {{ formatAmount(record.amount) }}
          </span>
        </h2>
        <p>Date: {{ formatDateOnly(record.date) }}</p>
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
  --overflow-y: auto;
}
.text-green {
  color: #00d336;
}
.text-red {
  color:#df1800
}
</style>