<template>
  <ion-page>
    <ion-content :fullscreen="true" overflow-scroll="true">
      <ion-progress-bar v-if="!loaded" type="indeterminate"></ion-progress-bar>
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Front Page</ion-title>
        </ion-toolbar>
      </ion-header>
      <div class="fade-off"></div>
      <ion-refresher
        slot="fixed"
        @ionRefresh="refreshData($event)"
        v-if="refresher"
      >
        <ion-refresher-content></ion-refresher-content>
      </ion-refresher>
      <!-- WHAT'S HAPPENING FEED -->
      <Feed
        ref="feed"
        v-if="session && prefs.enableFeed"
        @openProject="openProject"
        @openUser="openUser"
        @fullscreen="refresher ? (refresher = false) : (refresher = true)"
      ></Feed>
      <!-- FEATURED PROJECTS -->
      <ion-text>
        <h2><ion-icon :icon="ribbonOutline"></ion-icon>Featured</h2>
      </ion-text>
      <div class="sidescroll">
        <div v-for="project in featuredProjects" :key="project.thumbnail_url">
          <ProjectCard
            :title="project.title"
            :author="project.creator"
            :thumb="project.thumbnail_url"
            :id="project.id"
            :instructions="project.instructions"
            :credits="project.description"
          ></ProjectCard>
        </div>
      </div>
      <!-- PROJECTS FRIENDS LOVED -->
      <ion-text v-if="session && prefs.enableFeed">
        <h2><ion-icon :icon="happyOutline"></ion-icon>Friends Loved</h2>
      </ion-text>
      <div class="sidescroll" v-if="session && prefs.enableFeed">
        <div
          v-for="project in followingLovedProjects"
          :key="project.thumbnail_url"
        >
          <ProjectCard
            :title="project.title"
            :author="project.author.username"
            :thumb="project.image"
            :id="project.id"
            :instructions="project.instructions"
            :credits="project.description"
          ></ProjectCard>
        </div>
      </div>
      <!-- TOP LOVED PROJECTS -->
      <ion-text>
        <h2><ion-icon :icon="heartOutline"></ion-icon>Top Loved</h2>
      </ion-text>
      <div class="sidescroll">
        <div v-for="project in lovedProjects" :key="project.thumbnail_url">
          <ProjectCard
            :title="project.title"
            :author="project.creator"
            :thumb="project.thumbnail_url"
            :id="project.id"
            :instructions="project.instructions"
            :credits="project.description"
          ></ProjectCard>
        </div>
      </div>
      <!-- CURATED PROJECTS -->
      <ion-text>
        <h2><ion-icon :icon="diamondOutline"></ion-icon>Curated</h2>
      </ion-text>
      <div class="sidescroll">
        <div v-for="project in curatedProjects" :key="project.thumbnail_url">
          <ProjectCard
            :title="project.title"
            :author="project.creator"
            :thumb="project.thumbnail_url"
            :id="project.id"
            :instructions="project.instructions"
            :credits="project.description"
          ></ProjectCard>
        </div>
      </div>
      <!-- TOP REMIXED PROJECTS -->
      <ion-text>
        <h2><ion-icon :icon="syncOutline"></ion-icon>Top Remixed</h2>
      </ion-text>
      <div class="sidescroll">
        <div v-for="project in remixedProjects" :key="project.thumbnail_url">
          <ProjectCard
            :title="project.title"
            :author="project.creator"
            :thumb="project.thumbnail_url"
            :id="project.id"
            :instructions="project.instructions"
            :credits="project.description"
          ></ProjectCard>
        </div>
      </div>
      <ion-text>
        <h2><ion-icon :icon="personOutline"></ion-icon>Top Followed</h2>
      </ion-text>
      <div class="sidescroll">
        <UserCard
          v-for="user in topUsers"
          :key="user.id"
          :id="user.id"
          :name="user.username"
          :followers="user.statistics.followers"
        />
      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import LongPress from "vue-directive-long-press";
const utils = require("../utils.js");
import {
  IonProgressBar,
  IonPage,
  IonHeader,
  IonRefresher,
  IonRefresherContent,
  IonToolbar,
  IonTitle,
  IonContent,
  IonText,
  IonIcon,
  modalController,
  alertController,
  // actionSheetController
} from "@ionic/vue";
import Feed from "../components/Feed.vue";
import ProjectCard from "../components/ProjectCard.vue";
import ProjectModal from "../components/ProjectModal.vue";
import UserCard from "../components/UserCard.vue";
import UserModal from "../components/UserModal.vue";
import {
  ribbonOutline,
  heartOutline,
  diamondOutline,
  syncOutline,
  happyOutline,
  personOutline,
} from "ionicons/icons";
export default {
  name: "ExplorePage",
  components: {
    IonProgressBar,
    IonHeader,
    IonRefresher,
    IonRefresherContent,
    IonToolbar,
    IonTitle,
    IonContent,
    IonPage,
    IonText,
    IonIcon,
    UserCard,
    ProjectCard,
    Feed,
  },
  directives: {
    "long-press": LongPress,
  },
  setup() {
    return {
      ribbonOutline,
      heartOutline,
      diamondOutline,
      syncOutline,
      happyOutline,
      personOutline,
    };
  },
  data() {
    return {
      refresher: true,
      featuredProjects: [],
      lovedProjects: [],
      curatedProjects: [],
      remixedProjects: [],
      followingLovedProjects: [],
      topUsers: [],
      loaded: false,
      session: JSON.parse(window.localStorage.getItem("session"))
        ? JSON.parse(window.localStorage.getItem("session"))
        : null,
      prefs: JSON.parse(window.localStorage.getItem("preferences"))
        ? JSON.parse(window.localStorage.getItem("preferences"))
        : null,
    };
  },
  mounted() {
    this.detectParams();
    this.refreshData();
  },
  watch: {
    // call again the method if the route changes
    $route: "detectParams",
  },
  methods: {
    detectParams() {
      let params = utils.getParams(window.location.href);
      if (params.project) {
        this.openProject(params.project);
      } else if (params.studio) {
        this.presentAlert(
          "Under construction",
          "",
          "Studios have not yet been implemented.  We're working on it!"
        );
      } else if (params.user) {
        this.openUser(params.user);
      }
    },
    async presentAlert(title, code, message) {
      const alert = await alertController.create({
        header: title,
        subHeader: code,
        message: message,
        buttons: ["OK"],
      });
      return alert.present();
    },
    async openProject(id) {
      const modal = await modalController.create({
        component: ProjectModal,
        cssClass: "open-modal",
        componentProps: {
          embed: `https://turbowarp.org/${id}/embed`,
          id: id,
        },
      });
      return modal.present();
    },
    async openUser(name) {
      const modal = await modalController.create({
        component: UserModal,
        cssClass: "open-modal",
        componentProps: {
          username: name,
        },
      });
      return modal.present();
    },
    async refreshData(event) {
      this.loaded = false;
      if (window.localStorage.getItem("session")) {
        this.$refs.feed.loadFeed(4);
      }
      fetch("https://api.scratch.mit.edu/proxy/featured").then(
        async (response) => {
          if (response.status == 200) {
            let data = await response.json();
            this.featuredProjects = data.community_featured_projects;
            this.lovedProjects = data.community_most_loved_projects;
            this.remixedProjects = data.community_most_remixed_projects;
            this.curatedProjects = data.curator_top_projects;
            this.loaded = true;
          } else {
            this.presentAlert(
              response.status,
              "",
              "We encountered an error while fetching data."
            );
          }
        }
      );
      if (this.session && this.prefs.enableFeed) {
        fetch(
          `https://api.scratch.mit.edu/users/${this.session.username}/following/users/loves`,
          {
            method: "GET",
            url: `https://api.scratch.mit.edu/users/${this.session.username}/following/users/loves`,
            headers: {
              "x-requested-with": "XMLHttpRequest",
              origin: "https://scratch.mit.edu/",
              referer: `https://scratch.mit.edu/`,
              "x-token": this.session.token,
            },
          }
        ).then(async (response) => {
          if (response.status == 200) {
            this.followingLovedProjects = await response.json();
          } else {
            this.presentAlert(
              response.status,
              "",
              "We encountered an error while fetching data."
            );
          }
        });
      }
      fetch("https://scratchdb.lefty.one/v3/user/rank/global/followers/0").then(
        async (res) => {
          if (res.status == 200) {
            this.topUsers = await res.json();
          } else {
            this.presentAlert(
              res.status,
              "",
              "We encountered an error while fetching some data."
            );
          }
        }
      );
      if (event) {
        event.target.complete();
      }
    },
  },
  computed: {
    loading: function () {
      return this.loaded.length < this.requestCount;
    },
  },
};
</script>
<style scoped>
ion-icon {
  color: var(--ion-text-color);
  transform: translateY(4px);
  padding-right: 10px;
}

@media (orientation: landscape) {
  ion-card {
    width: 30vw;
  }
}
</style>
