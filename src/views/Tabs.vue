<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet></ion-router-outlet>
      <ion-tab-bar slot="bottom">
        <ion-tab-button tab="tab1" href="/tabs/explore" @click="vibrate">
          <ion-icon :icon="earthOutline" />
          <ion-label>Explore</ion-label>
        </ion-tab-button>

        <ion-tab-button tab="tab2" href="/tabs/search" @click="vibrate">
          <ion-icon :icon="search" />
          <ion-label>Search</ion-label>
        </ion-tab-button>

        <ion-tab-button tab="tab3" href="/tabs/messages" @click="vibrate">
          <ion-icon :icon="mailOutline" />
          <ion-label>Messages</ion-label>
          <ion-badge v-if="userSignedIn && messageCount != 0" color="primary">{{
            messageCount
          }}</ion-badge>
        </ion-tab-button>

        <ion-tab-button tab="tab4" href="/tabs/settings" @click="vibrate">
          <ion-icon :icon="settingsOutline" />
          <ion-label>Settings</ion-label>
        </ion-tab-button>
      </ion-tab-bar>
    </ion-tabs>
  </ion-page>
</template>

<script>
import "@capacitor-community/http";
import { Haptics, ImpactStyle } from "@capacitor/haptics";
import {
  IonTabBar,
  IonTabButton,
  IonTabs,
  IonLabel,
  IonIcon,
  IonPage,
  IonBadge,
  IonRouterOutlet,
} from "@ionic/vue";
import {
  earthOutline,
  search,
  mailOutline,
  settingsOutline,
} from "ionicons/icons";
import { getPrefs } from "../utils";
export default {
  name: "Tabs",
  components: {
    IonLabel,
    IonTabs,
    IonTabBar,
    IonTabButton,
    IonIcon,
    IonPage,
    IonBadge,
    IonRouterOutlet,
  },
  setup() {
    let userSignedIn, username;
    if (window.localStorage.getItem("session")) {
      let session = JSON.parse(window.localStorage.getItem("session"));
      username = session.username;
      userSignedIn = true;
    } else {
      userSignedIn = false;
    }
    return {
      earthOutline,
      search,
      mailOutline,
      settingsOutline,
      userSignedIn,
      username,
    };
  },
  data() {
    return {
      messageCount: 0,
    };
  },
  created() {
    if (this.userSignedIn) {
      fetch(`https://api.scratch.mit.edu/users/${this.username}/messages/count`)
        .then((res) => res.json())
        .then((data) => {
          this.messageCount = data.count;
        });
      window.setInterval(() => {
        this.refreshMessages();
      }, 15000);
    }
  },
  methods: {
    async vibrate() {
      if (getPrefs().haptics) {
        await Haptics.impact({
          style: ImpactStyle.Light,
        });
      }
    },
    refreshMessages() {
      fetch(`https://api.scratch.mit.edu/users/${this.username}/messages/count`)
        .then((res) => res.json())
        .then((data) => {
          this.messageCount = data.count;
        });
    },
  },
};
</script>
