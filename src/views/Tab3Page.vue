<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title style="padding-top: 35px;padding-bottom: 15px;">Profile</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <!-- <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Profile</ion-title>
        </ion-toolbar>
      </ion-header> -->

      <h1 style="text-align: center;">Version 1.0</h1>
      <!-- <ion-item lines="none" style="width: 200px;">
        <ion-label>Dark Mode</ion-label>
        <ion-toggle @ionChange="toggleDarkMode" :checked="isDark" />
      </ion-item> -->
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonLabel, IonToggle } from '@ionic/vue';
import ExploreContainer from '@/components/ExploreContainer.vue';
import { ref, onMounted } from 'vue';

const isDark = ref(false);

const toggleDarkMode = (event: CustomEvent) => {
  isDark.value = event.detail.checked;
  document.body.classList.toggle('dark', isDark.value);
};

onMounted(() => {
  // Optional: Auto-detect or load saved preference
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
  isDark.value = prefersDark;
  document.body.classList.toggle('dark', isDark.value);
});
</script>
