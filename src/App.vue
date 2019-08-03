<template>
  <f7-app>
    <f7-statusbar />

    <f7-panel left cover theme-dark>
      <f7-navbar title="Left Panel" />
      <f7-block>Left panel content</f7-block>
    </f7-panel>

    <f7-panel right reveal>
      <f7-navbar title="Right Panel" />
      <f7-block>Right panel content</f7-block>
    </f7-panel>

    <f7-view main>
      <f7-page :page-content="false" class="color-theme-green">
        <f7-navbar title="TuRa Sound App"></f7-navbar>
        <f7-toolbar tabbar bottom>
          <f7-link tab-link="#tab-1" tab-link-active>
            <f7-icon material="queue_music"></f7-icon>
            <span class="tabbar-label">Herren</span>
          </f7-link>
          <f7-link tab-link="#tab-2">
            <f7-icon material="queue_music"></f7-icon>
            <span class="tabbar-label">Damen</span>
          </f7-link>
          <f7-link tab-link="#tab-3">
            <f7-icon material="cloud_download">
              <f7-badge v-if="count>0" color="red">{{count}}</f7-badge>
            </f7-icon>
            <span class="tabbar-label">Update</span>
          </f7-link>
        </f7-toolbar>
        <f7-tabs>
          <f7-tab id="tab-1" class="page-content" tab-active>
            <f7-block-title>Herren</f7-block-title>
            <f7-block>
              <f7-row>
                <f7-col>
                  <f7-button large fill>Tom</f7-button>
                </f7-col>
                <f7-col>
                  <f7-button large fill>Lasse</f7-button>
                </f7-col>
                <f7-col>
                  <f7-button large fill>Arndt</f7-button>
                </f7-col>
                <f7-col>
                  <f7-button large fill>Peter</f7-button>
                </f7-col>
              </f7-row>
            </f7-block>
          </f7-tab>
          <f7-tab id="tab-2" class="page-content">
            <f7-block-title>Damen</f7-block-title>
            <f7-block>
              <f7-row>
                <f7-col>
                  <f7-button large fill>Louisa</f7-button>
                </f7-col>
                <f7-col>
                  <f7-button large fill>Naja</f7-button>
                </f7-col>
                <f7-col>
                  <f7-button large fill>Jule</f7-button>
                </f7-col>
                <f7-col>
                  <f7-button large fill>Johanna</f7-button>
                </f7-col>
              </f7-row>
            </f7-block>
          </f7-tab>
          <f7-tab id="tab-3" class="page-content">
            <f7-block>
              <f7-block-title v-if="updates.length>0">Updates verfügbar</f7-block-title>
              <f7-block-title v-else>Keine Updates verfügbar</f7-block-title>
<f7-list simple-list>
   <f7-progressbar infinite v-if="loading"></f7-progressbar>
  <f7-list-item v-for="spieler in updates" :title=spieler.anzeigename></f7-list-item>
</f7-list>
              <f7-button v-if="updates.length>0" large fill @click="downloadAll">Alle downloaden</f7-button>
            </f7-block>
          </f7-tab>
        </f7-tabs>
      </f7-page>
    </f7-view>
  </f7-app>
</template>

<script>
import axios from "axios";
import low from "lowdb";
import LocalStorage from "lowdb/adapters/LocalStorage";

const adapter = new LocalStorage("db");
const db = low(adapter);

export default {
  name: "app",
  data: function() {
    return {
      count: 0,
      loading: false,
      updates: [],
    };
  },
  mounted() {
    this.checkForUpdates();
     /* 
     Für späteres Aktualiseren
     this.$nextTick(function () {
            window.setInterval(() => {
                this.checkForUpdates();
            },20000);
        }) */
  },
  methods: {
    checkForUpdates: function() {
      axios
        .get("http://beugelbuddel.de/sound/info")
        .then(response => {
          var count = 0;
          // handle success
          console.log(response.data);
          this.updates = response.data;
          console.log(db.value());
          response.data.forEach(function(spieler) {
            if (
              db
                .get("spieler")
                .find({ username: spieler.username })
                .value() == null ||
              db
                .get("spieler")
                .find({ username: spieler.username })
                .value().version < spieler.version
            ) {
              console.log(
                "Download für " + spieler.username + " muss erfolgen!"
              );
              count++;
            } else {
              console.log(spieler.username + " ist up2date!");
            }
          });

          console.log(this.updates);


                 this.count = count;
        });
    },
    //Kann später gelöscht werden
    Sleep: function Sleep(milliseconds) {
   return new Promise(resolve => setTimeout(resolve, milliseconds));
},
    downloadAll: async function(){
      const updates = [...this.updates];
      var i = 0;
         for (let spieler of updates) {
          await this.downloadFile(spieler.username);
        //Löschen aus der Liste
        this.updates.splice(this.updates.findIndex(x => x.username === spieler.username), 1);
        console.log(i);
        i++;
        console.log(spieler.username);
        console.log(this.updates);
    }
    this.loading = false;
    },
    downloadFile: async function(username) {
      this.loading = true;
      await this.Sleep(1000);
      console.log(username);
      // let s = await axios.get("test.de");
    }
  }
};
</script>

<style>
a[class*="button"] {
  padding: 0;
}
</style>
