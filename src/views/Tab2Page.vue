<script setup lang="ts">
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonInput, IonItem, IonList, IonLabel, IonModal, IonButton, IonButtons,
  IonRadio, IonRadioGroup, IonFab, IonFabButton, IonIcon, IonDatetime, IonDatetimeButton,
  IonSelect, IonSelectOption, IonToast, IonAlert, IonSegment, IonSegmentButton, IonSegmentView, IonSegmentContent
} from '@ionic/vue';
import { ref, onMounted, watch } from 'vue';
import { Storage } from '@ionic/storage';
import { ellipse, square, triangle, appsOutline, walletOutline, add } from 'ionicons/icons';

// -------------------------
// Refs and Reactive Values
// -------------------------
const isAlertOpen = ref(false);
const isToastOpen = ref(false);
const isModelOpen = ref(false);
const transactionToDelete = ref<string | null>(null);

const message = ref('');
const amount = ref('');
const date = ref(new Date().toISOString()); // Default to now
const monthYear = ref(new Date().toISOString().slice(0, 7)); // "YYYY-MM"
const category = ref('');
const notes = ref('');

const allTransactions = ref<any[]>([]);
const transactions = ref<any[]>([]);

const modal = ref();
const store = new Storage();

// -------------------------
// Lifecycle
// -------------------------
onMounted(async () => {
  await store.create();

  const stored = await store.get('transactions');
  if (Array.isArray(stored)) {
    allTransactions.value = stored;
    filterTransactionsByMonthYear();
  } else {
    allTransactions.value = [];
    transactions.value = [];
  }
});

// -------------------------
// Watchers
// -------------------------
watch(monthYear, filterTransactionsByMonthYear);

// -------------------------
// Methods
// -------------------------
const promptDelete = (id: string) => {
  transactionToDelete.value = id;
  isAlertOpen.value = true;
};

const longPressTimeout = ref<number | null>(null);

const handleLongPressStart = (id: string) => {
  longPressTimeout.value = window.setTimeout(() => {
    // Vibrate
    if (navigator.vibrate) {
      navigator.vibrate(100);
    }

    // Show modal or prompt delete
    promptDelete(id);
  }, 500); // 500ms = long press
};

const handleLongPressEnd = () => {
  if (longPressTimeout.value !== null) {
    clearTimeout(longPressTimeout.value);
    longPressTimeout.value = null;
  }
};


function filterTransactionsByMonthYear() {
  if (!monthYear.value) return;

  const [yearStr, monthStr] = monthYear.value.split('-');
  const year = Number(yearStr);
  const month = Number(monthStr);

  if (isNaN(year) || isNaN(month)) return;

  transactions.value = allTransactions.value.filter(t => {
    const transactionDate = new Date(t.date);
    return transactionDate.getFullYear() === year && transactionDate.getMonth() === month - 1;
  });
}

const deleteTransaction = async () => {
  allTransactions.value = allTransactions.value.filter(
    (record: any) => record.id !== transactionToDelete.value
  );

  filterTransactionsByMonthYear();
  await store.set('transactions', JSON.parse(JSON.stringify(allTransactions.value)));

  transactionToDelete.value = null;
  isToastOpen.value = true;
  message.value = 'Transaction deleted successfully.';
};

async function deleteAll() {
  await store.remove('transactions');
  allTransactions.value = [];
  transactions.value = [];
}

const saveTransaction = async () => {
  if (!amount.value || !date.value || !category.value || !notes.value) {
    console.warn('All fields are required.');
    return;
  }

  const newRecord = {
    id: crypto.randomUUID(),
    amount: parseFloat(parseFloat(amount.value).toFixed(2)),
    date: date.value,
    category: category.value,
    notes: notes.value
  };

  allTransactions.value.push(newRecord);
  await store.set('transactions', JSON.parse(JSON.stringify(allTransactions.value)));

  isToastOpen.value = true;
  message.value = 'Transaction added successfully.';

  // Reset form
  amount.value = '';
  date.value = new Date().toISOString();
  category.value = '';
  notes.value = '';
  filterTransactionsByMonthYear();
  cancel();
};

const cancel = () => modal.value?.$el.dismiss(null, 'cancel');
const confirm = () => {}; // placeholder

function formatDateOnly(dateString: string): string {
  if (!dateString) return '';
  const d = new Date(dateString);
  return d.toLocaleDateString(undefined, {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });
}

function formatAmount(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  }).format(amount);
}

// -------------------------
// Alert Buttons
// -------------------------
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
    
    <ion-datetime-button :icon="walletOutline"    datetime="topdatetime" presentation="month-year" style="margin-top: 20px;" class="custom-datetime-btn"/>
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

    <ion-content  :fullscreen="true" class="ion-padding">
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


  <ion-segment>
    <ion-segment-button value="first" content-id="first">
      <ion-label>All</ion-label>
    </ion-segment-button>
    <ion-segment-button value="second" content-id="second">
      <ion-label>Income</ion-label>
    </ion-segment-button>
    <ion-segment-button value="third" content-id="third">
      <ion-label>Expenses</ion-label>
    </ion-segment-button>
  </ion-segment>
  <ion-segment-view padding="true" scroll-y>
    <ion-segment-content id="first" class="transactionWrapper">
      <ion-list v-if="transactions?.length > 0">
        <ion-item
          button
          v-for="(record, index) in transactions"
          :key="record.id"
          @touchstart="handleLongPressStart(record.id)"
          @touchend="handleLongPressEnd"
          @touchcancel="handleLongPressEnd"
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
    </ion-segment-content>
    <ion-segment-content id="second" class="transactionWrapper">
      <ion-list v-if="transactions?.length > 0">
        <template v-for="(record, index) in transactions"
          :key="record.id"
          @click="promptDelete(record.id)">
        <ion-item
          button
          v-if="record.category == 'income'"
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
      </template>
      </ion-list>
    </ion-segment-content>
    <ion-segment-content id="third" class="transactionWrapper">
      <ion-list v-if="transactions?.length > 0">
        <template v-for="(record, index) in transactions"
          :key="record.id"
          @click="promptDelete(record.id)">
        <ion-item
          button
          v-if="record.category == 'expenses'"
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
      </template>
      </ion-list>
    </ion-segment-content>
  </ion-segment-view>

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
.transactionWrapper {
  padding-left: 10px;
  padding-right: 10px;
}

.custom-datetime-btn::part(native) {
  background-color: #20d0ff;  /* Background */
  color: white;               /* Text color */
  font-size: 1.2rem;            /* Font size */
  padding: 10px 20px;         /* Padding */
  border-radius: 8px;         /* Rounded corners */
  margin-bottom: 10px;
}
</style>