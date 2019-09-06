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
        <f7-navbar title="Sound App">
          <div id="bars">
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
            <div class="bar"></div>
          </div>
        </f7-navbar>
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

          <f7-link v-if="currentFile" @click="stop()" class="bg-color-red">
            <f7-button id="stop" class="bg-color-red" round>
              <f7-icon style="line-height:35px" material="stop" class="color-white"></f7-icon>
            </f7-button>
          </f7-link>
        </f7-toolbar>
        <f7-tabs>
          <f7-tab id="tab-1" class="page-content" tab-active>
            <f7-block-title>Herren</f7-block-title>
            <f7-block>
              <f7-row class="reihe" v-for="i in Math.ceil(herren.length / 4)">
                <f7-col v-for="item in herren.slice((i - 1) * 4, i * 4)">
                  <f7-button large @click="play(item.username)" fill>{{item.anzeigename}}</f7-button>
                </f7-col>
              </f7-row>
            </f7-block>
          </f7-tab>
          <f7-tab id="tab-2" class="page-content">
            <f7-block-title>Damen</f7-block-title>
            <f7-block>
              <f7-row class="reihe" v-for="i in Math.ceil(damen.length / 4)">
                <f7-col v-for="item in damen.slice((i - 1) * 4, i * 4)">
                  <f7-button large @click="play(item.username)" fill>{{item.anzeigename}}</f7-button>
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
                <f7-list-item v-for="spieler in updates" :title="spieler.anzeigename"></f7-list-item>
              </f7-list>
              <f7-button v-if="updates.length>0" large fill @click="downloadAll">Alle downloaden</f7-button>
            </f7-block>

            <f7-block>
              <f7-button large fill color="blue" @click="checkForUpdates">Auf Updates prüfen</f7-button>
            </f7-block>

            <f7-block>
              <f7-block-title v-if="updates.length>0">Probleme mit der App?</f7-block-title>Bitte nicht während des Spiels drücken.
              <f7-button fill color="red" large @click="resetApp">RESET</f7-button>
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
      currentFile: null,
      loading: false,
      updates: [],
      herren: [],
      spieler: [],
      damen: [],
      timer: "",
      timerDur: "",
      timeout: "",
      thevolume: 1
    };
  },
  created() {
    this.getSpieler();
    window.plugins.insomnia.keepAwake();
    this.checkForUpdates();
  },
  mounted() {
    /*
    this.$nextTick(function() {
      window.setInterval(() => {
        this.checkForUpdates();
      }, 20000);
    });
    */
  },
  computed: {
    sortedSpieler: function() {
      function compare(a, b) {
        if (a.anzeigename < b.anzeigename) return -1;
        if (a.anzeigename > b.anzeigename) return 1;
        return 0;
      }
      return this.spieler.sort(compare);
    }
  },
  methods: {
    resetApp: function() {
      db.set("spieler", []).write();
      this.spieler = [];
      this.herren = [];
      this.damen = [];
      this.getSpieler();
      this.checkForUpdates();
    },
    getSpieler: function() {
      db.defaults({ spieler: [] }).write();
      this.herren = [];
      this.damen = [];
      this.spieler = db.get("spieler").value();

      this.sortedSpieler.forEach(element => {
        if (element.mannschaft == "herren") {
          this.herren.push(element);
        } else {
          this.damen.push(element);
        }
      });
    },
    play: async function(name) {
      await this.stop();
      var fileDeviceDir = cordova.file.dataDirectory;
      this.currentFile = new Media(
        fileDeviceDir + name + ".mp3",
        // success callback
        () => {
          console.log("playAudio():Audio Success");
          this.currentFile = null;
        },
        // error callback
        function(err) {
          console.log("playAudio():Audio Error: " + JSON.stringify(err));
        },
        (status) => {
          console.log(status);
        }
      );
      this.currentFile.play();
      //Start Fadeout after 70% Playtime
      var counter = 0;
      var self = this;
      this.timerDur = setInterval(function() {
        counter = counter + 100;
        if (counter > 2000) {
          clearInterval(self.timerDur);
        }
        var dur = self.currentFile.getDuration();
        if (dur > 0) {
          clearInterval(self.timerDur);
         self.timeout = setTimeout(() => {
            self.fadeOut();
          }, self.currentFile.getDuration() * 0.7 * 1000);
        }
      }, 100);
    },
    fadeOut() {
      if (this.currentFile) {
        var fadeseconds = this.currentFile.getDuration() * 0.3;
        this.thevolume = 1;
        var fadeStep = 1 / (fadeseconds * 10);
        var self = this;
        this.timer = setInterval(() => {
          if (self.thevolume > 0 && self.currentFile != null) {
            console.log(self.thevolume);
            self.thevolume = self.thevolume - fadeStep;
            self.currentFile.setVolume(self.thevolume); // media is your audio object
          } else {
            clearInterval(self.timer);
          }
        }, 100);
      }
    },
    stop() {
      if (this.currentFile) {
        this.currentFile.stop();
        this.currentFile.release();
        this.thevolume = 1;
      }
      clearInterval(this.timerDur);
      clearInterval(this.timer);
      clearTimeout(this.timeout);
    },
    checkForUpdates: function() {
      this.updates = [];
      axios.get("https://beugelbuddel.de/sound/info/").then(response => {
        var count = 0;
        response.data.forEach(spieler => {
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
            console.log("Download für " + spieler.username + " muss erfolgen!");
            this.updates.push(spieler);
            count++;
          } else {
            console.log(spieler.username + " ist up2date!");
          }
        });
        this.count = count;
      });
    },
    downloadAll: async function() {
      const updates = [...this.updates];
      var i = 0;
      for (let spieler of updates) {
        await this.downloadFile(spieler);
        this.updates.splice(
          this.updates.findIndex(x => x.username === spieler.username),
          1
        );
        i++;
      }
      this.loading = false;
      this.$forceUpdate();
    },
    downloadFile: async function(spieler) {
      var cordovaFileTransfer = new FileTransfer();
      var fileDeviceDir = cordova.file.dataDirectory;
      var targetPath = fileDeviceDir + spieler.username + ".mp3";
      var uri = encodeURI(
        "https://beugelbuddel.de/sound/uploads/" + spieler.username + ".mp3"
      );
      this.loading = true;

      await cordovaFileTransfer.download(
        uri,
        targetPath,
        r => {
          if (
            !db
              .get("spieler")
              .find({ username: spieler.username })
              .value()
          ) {
            db.get("spieler")
              .push(spieler)
              .write();
          } else {
            var currentspieler = db
              .get("spieler")
              .find({ username: spieler.username })
              .value();
            db.find({ username: spieler.username })
              .assign({ version: currentspieler.version++ })
              .write();
          }

          this.getSpieler();
        },
        this.downloadFail,
        true
      );
      this.count--;
    },
    downloadFail: function(error) {
      alert("An error has occurred: Code = " + error.code);
      console.log("upload error source " + error.source);
      console.log("upload error target " + error.target);
    }
  }
};
</script>

<style>
a[class*="button"] {
  padding: 0;
}

.page-content {
  background-size: cover;
}
.reihe {
  margin-top: 20px !important;
}

#bars {
  height: 60px;
  left: 100px;
  margin: 0px 0 0 -40px;
  position: relative;
  width: 40px;
}

.bar {
  background: green;
  bottom: 1px;
  height: 3px;
  position: absolute;
  width: 3px;
  animation: sound 0ms -800ms linear infinite alternate;
}

@keyframes sound {
  0% {
    opacity: 0.35;
    height: 3px;
  }
  100% {
    opacity: 1;
    height: 28px;
  }
}

.bar:nth-child(1) {
  margin-left: 1px;
  animation-duration: 474ms;
}
.bar:nth-child(2) {
  margin-left: 5px;
  animation-duration: 433ms;
}
.bar:nth-child(3) {
  margin-left: 9px;
  animation-duration: 407ms;
}
.bar:nth-child(4) {
  margin-left: 13px;
  animation-duration: 458ms;
}
.bar:nth-child(5) {
  margin-left: 17px;
  animation-duration: 400ms;
}
.bar:nth-child(6) {
  margin-left: 21px;
  animation-duration: 427ms;
}
.bar:nth-child(7) {
  margin-left: 25px;
  animation-duration: 441ms;
}
.bar:nth-child(8) {
  margin-left: 29px;
  animation-duration: 419ms;
}
.bar:nth-child(9) {
  margin-left: 33px;
  animation-duration: 487ms;
}
.bar:nth-child(10) {
  margin-left: 47px;
  animation-duration: 555ms;
}
</style>
