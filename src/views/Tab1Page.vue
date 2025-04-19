<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Dashboard</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Dashboard</ion-title>
        </ion-toolbar>
      </ion-header>

      <h1 style="margin-left: 20px;color: #565656;">Hi Cedric!</h1>
      <ion-grid :fixed="true">
        <ion-row>
            <ion-card >
              <ion-card-content>
                <ion-grid :fixed="true">
                  <ion-row>
                    <ion-col>
                    <h1>Income</h1>
                    </ion-col>
                    <ion-col>
                    <h1>{{ totalIncome ?? 0.00 }}</h1>
                    </ion-col>
                  </ion-row>
                </ion-grid>
              </ion-card-content>
            </ion-card>
            <ion-card >
              <ion-card-content>
                <ion-grid :fixed="true">
                  <ion-row>
                    <ion-col>
                    <h1>Expenses</h1>
                    </ion-col>
                    <ion-col>
                    <h1>{{ totalExpenses ?? 0.00 }}</h1>
                    </ion-col>
                  </ion-row>
                </ion-grid>
              </ion-card-content>
            </ion-card>
            <ion-card >
              <ion-card-content>
                <ion-grid :fixed="true">
                  <ion-row>
                    <ion-col>
                    <h1>Balance</h1>
                    </ion-col>
                    <ion-col>
                    <h1>{{ (totalIncome - totalExpenses).toFixed(2) ?? 0.00 }}</h1>
                    </ion-col>
                  </ion-row>
                </ion-grid>
              </ion-card-content>
            </ion-card>
        </ion-row>
      </ion-grid>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
import { IonCard, IonCardContent, IonCardHeader, IonCardSubtitle, IonCardTitle } from '@ionic/vue';
import { IonCol, IonGrid, IonRow } from '@ionic/vue';
import { ref, onMounted, computed } from 'vue';
import { Storage } from '@ionic/storage';
import ExploreContainer from '@/components/ExploreContainer.vue';

const store = new Storage();
const transactions = ref<any[]>([]);

onMounted(async () => {
  await store.create();
  const stored = await store.get('transactions');
  if (Array.isArray(stored)) {
    transactions.value = stored;
  }
});

const totalIncome = computed(() => {
  return transactions.value
    .filter(t => t.category === 'income')
    .reduce((sum, t) => sum + t.amount, 0)
    .toFixed(2); // format to 2 decimal places
});

const totalExpenses = computed(() => {
  return transactions.value
    .filter(t => t.category === 'expenses')
    .reduce((sum, t) => sum + t.amount, 0)
    .toFixed(2); // format to 2 decimal places
});
</script>


<style>
ion-card {
  width: 100%;
}
</style>