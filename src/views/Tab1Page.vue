<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title style="padding-top: 20px;">Dashboard</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <!-- 🌀 Pull to refresh -->
      <ion-refresher slot="fixed" @ionRefresh="handleRefresh">
        <ion-refresher-content pulling-text="Pull to refresh" refreshing-spinner="circles">
        </ion-refresher-content>
      </ion-refresher>

      <h1 style="margin-left: 20px;">  {{ new Date().toLocaleString('default', { month: 'long' }) }}
      </h1>
      <ion-grid :fixed="true">
        <ion-row>
          <ion-card style="background-color: #20d0ff;color: white;">
              <ion-card-content>
                <ion-grid :fixed="true">
                  <ion-row>
                    <ion-col>
                    <h1>Balance</h1>
                    </ion-col>
                    <ion-col>
                    <h1>₱
                      {{ formatAmount(totalIncome - totalExpenses) ?? 0.00 }}</h1>
                    </ion-col>
                  </ion-row>
                </ion-grid>
              </ion-card-content>
            </ion-card>
            <ion-card style="background-color: #20d0ff;color: white;">
              <ion-card-content>
                <ion-grid :fixed="true">
                  <ion-row>
                    <ion-col>
                    <h1>Income</h1>
                    </ion-col>
                    <ion-col>
                    <h1>₱{{ formatAmount(totalIncome) ?? 0.00 }}</h1>
                    </ion-col>
                  </ion-row>
                </ion-grid>
              </ion-card-content>
            </ion-card>
            <ion-card style="background-color: #20d0ff;color: white;">
              <ion-card-content>
                <ion-grid :fixed="true">
                  <ion-row>
                    <ion-col>
                    <h1>Expenses</h1>
                    </ion-col>
                    <ion-col>
                    <h1>₱{{ formatAmount(totalExpenses) ?? 0.00 }}</h1>
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
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonRefresher, IonRefresherContent } from '@ionic/vue';
import { IonCard, IonCardContent, IonCardHeader, IonCardSubtitle, IonCardTitle } from '@ionic/vue';
import { IonCol, IonGrid, IonRow } from '@ionic/vue';
import { ref, onMounted, computed } from 'vue';
import { Storage } from '@ionic/storage';
import ExploreContainer from '@/components/ExploreContainer.vue';

const store = new Storage();
const transactions = ref<any[]>([]);


const formatAmount = (amount: number) => {
  return new Intl.NumberFormat('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  }).format(amount);
};

onMounted(async () => {
  await store.create();
  const stored = await store.get('transactions');
  if (Array.isArray(stored)) {
    transactions.value = stored;
  }
});


const handleRefresh = async (event: CustomEvent) => {
  await store.create();
  const stored = await store.get('transactions');

  if (Array.isArray(stored)) {
    const now = new Date();
    const currentMonth = now.getMonth(); // 0-based (Jan = 0)
    const currentYear = now.getFullYear();

    // Filter transactions by current month and year
    transactions.value = stored.filter((t: any) => {
      const tDate = new Date(t.date);
      return (
        tDate.getMonth() === currentMonth &&
        tDate.getFullYear() === currentYear
      );
    });
  } else {
    transactions.value = [];
  }

  (event.target as HTMLIonRefresherElement).complete();
};


const totalIncome = computed(() => {
  const now = new Date();
  const currentMonth = now.getMonth(); // 0-based
  const currentYear = now.getFullYear();

  return transactions.value
    .filter(t => {
      const tDate = new Date(t.date);
      return (
        t.category === 'income' &&
        tDate.getMonth() === currentMonth &&
        tDate.getFullYear() === currentYear
      );
    })
    .reduce((sum, t) => sum + t.amount, 0)
    .toFixed(2);
});

const totalExpenses = computed(() => {
  const now = new Date();
  const currentMonth = now.getMonth(); // 0-based
  const currentYear = now.getFullYear();

  return transactions.value
    .filter(t => {
      const tDate = new Date(t.date);
      return (
        t.category === 'expenses' &&
        tDate.getMonth() === currentMonth &&
        tDate.getFullYear() === currentYear
      );
    })
    .reduce((sum, t) => sum + t.amount, 0)
    .toFixed(2);
});

</script>


<style>
ion-card {
  width: 100%;
}
</style>