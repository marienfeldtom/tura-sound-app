<template>
  <f7-app>
    <f7-statusbar />

    <f7-view main>
      <f7-page :page-content="false" class="app-page">
        <!-- HEADER / NAVBAR (Sporty, High-Contrast Clean Design System) -->
        <f7-navbar class="app-navbar" no-shadow>
          <div class="navbar-inner-custom">
            <!-- Brand & App Identity -->
            <div class="navbar-brand-section">
              <div class="navbar-logo-icon">
                <i class="material-icons">sports_handball</i>
              </div>
              <div class="navbar-title-wrap">
                <span class="navbar-title">7secs</span>
                <span class="navbar-subtitle" v-if="currentActiveTeam">{{ currentActiveTeam.name }}</span>
                <span class="navbar-subtitle" v-else-if="activeTab === 'tab-manage'">Verwaltung</span>
                <span class="navbar-subtitle" v-else>Soundboard</span>
              </div>
            </div>

            <!-- Navbar Right Actions / Active Indicators -->
            <div class="navbar-right-section">
              <!-- When audio is playing: Live 7s countdown & animated waveform -->
              <div v-if="playingSpieler" class="navbar-playing-pill">
                <span class="live-pulse-dot"></span>
                <span class="live-time-text">{{ playingSecondsRemaining.toFixed(1) }}s</span>
                <div class="nav-waveform-bars">
                  <div class="nav-bar"></div>
                  <div class="nav-bar"></div>
                  <div class="nav-bar"></div>
                  <div class="nav-bar"></div>
                </div>
              </div>

              <!-- Idle: Official Team ID Badge from Design System -->
              <div v-else-if="currentActiveTeam" class="team-id-badge header-badge">
                {{ currentActiveTeam.id }}
              </div>

              <!-- Idle on Manage tab -->
              <div v-else-if="activeTab === 'tab-manage'" class="navbar-manage-badge">
                <i class="material-icons">tune</i>
                <span>Verwaltung</span>
              </div>
            </div>
          </div>
        </f7-navbar>

        <!-- BOTTOM TOOLBAR / TABBAR (Max 3 Teams + Manage Tab) -->
        <f7-toolbar tabbar bottom class="app-toolbar">
          <!-- Dynamic Team Tabs (Max 3) -->
          <f7-link
            v-for="(team, index) in teams"
            :key="team.id"
            :tab-link="`#tab-team-${index}`"
            :tab-link-active="activeTab === `tab-team-${index}`"
            @click="setActiveTab(`tab-team-${index}`)"
            class="toolbar-tab-link"
          >
            <i class="material-icons">sports_handball</i>
            <span class="tabbar-label">{{ team.name || team.id }}</span>
          </f7-link>

          <!-- Manage / Updates / Add Team Tab -->
          <f7-link
            tab-link="#tab-manage"
            :tab-link-active="activeTab === 'tab-manage'"
            @click="setActiveTab('tab-manage')"
            class="toolbar-tab-link"
          >
            <div class="tab-icon-wrapper">
              <i class="material-icons">tune</i>
              <!-- Red notification dot if updates are available -->
              <span v-if="updateCount > 0" class="tab-update-dot"></span>
            </div>
            <span class="tabbar-label">Teams &amp; Updates</span>
          </f7-link>
        </f7-toolbar>

        <!-- MAIN TABS CONTAINER -->
        <f7-tabs class="app-tabs">
          <!-- NO TEAMS ONBOARDING STATE (If 0 teams configured) -->
          <f7-tab
            v-if="teams.length === 0"
            id="tab-onboarding"
            class="page-content tab-content-padded"
            tab-active
          >
            <div class="onboarding-container">
              <div class="onboarding-card">
                <div class="onboarding-icon-box">
                  <i class="material-icons onboarding-icon">queue_music</i>
                </div>
                <h1 class="onboarding-title">7secs Soundboard</h1>
                <p class="onboarding-subtitle">
                  Willkommen! Gib deine 3-stellige Mannschafts-ID ein (z. B. <strong>#H4R</strong>), um deinen Spieltagskader und Torsongs zu laden.
                </p>

                <div class="team-input-group">
                  <span class="team-input-prefix">#</span>
                  <input
                    type="text"
                    v-model="newTeamIdInput"
                    placeholder="H4R"
                    maxlength="5"
                    class="team-id-input"
                    @keyup.enter="handleAddTeam"
                    autocapitalize="characters"
                  />
                </div>

                <div v-if="addTeamError" class="alert-box alert-danger">
                  <i class="material-icons">error_outline</i>
                  <span>{{ addTeamError }}</span>
                </div>

                <button
                  class="btn-primary btn-large btn-block"
                  :disabled="addTeamLoading || !newTeamIdInput.trim()"
                  @click="handleAddTeam"
                >
                  <span v-if="!addTeamLoading">Mannschaft hinzufügen</span>
                  <span v-else class="btn-loading">
                    <span class="spinner"></span> Lädt Kader...
                  </span>
                </button>

                <div class="onboarding-info">
                  <i class="material-icons">info</i>
                  <span>Du kannst später in der Verwaltung bis zu 3 Mannschaften anlegen.</span>
                </div>
              </div>
            </div>
          </f7-tab>

          <!-- DYNAMIC TEAM TABS (1, 2 or 3 teams) -->
          <f7-tab
            v-for="(team, index) in teams"
            :key="team.id"
            :id="`tab-team-${index}`"
            class="page-content tab-content-padded"
            :tab-active="activeTab === `tab-team-${index}`"
          >
            <!-- Team Sub-Header Info Banner (Ultra-Compact Bar) -->
            <div class="team-header-bar">
              <div class="team-header-left">
                <span class="team-id-badge compact">{{ team.id }}</span>
                <span class="team-name">{{ team.name }}</span>
              </div>
              <span class="team-player-count">
                {{ getPlayersForTeam(team.id).length }} Spieler
              </span>
            </div>

            <!-- SOUNDBOARD PLAYER GRID (Compact Buzzer-Style Pads) -->
            <div v-if="getPlayersForTeam(team.id).length > 0" class="soundboard-grid">
              <div
                v-for="spieler in getPlayersForTeam(team.id)"
                :key="spieler.id"
                class="soundboard-card"
                :class="{ 'is-playing': isSpielerPlaying(spieler) }"
                @click="togglePlay(spieler)"
                :title="spieler.anzeigename"
              >
                <!-- Initial Avatar Box / Stop Indicator on playback -->
                <div
                  class="avatar-box"
                  :class="{ 'avatar-playing': isSpielerPlaying(spieler) }"
                >
                  <i v-if="isSpielerPlaying(spieler)" class="material-icons card-live-icon">stop</i>
                  <span v-else>{{ getInitial(spieler.anzeigename) }}</span>
                </div>

                <!-- Player Name -->
                <div class="player-name">
                  {{ spieler.anzeigename }}
                </div>

                <!-- Active Countdown Pill -->
                <div v-if="isSpielerPlaying(spieler)" class="countdown-badge">
                  {{ playingSecondsRemaining.toFixed(1) }}s
                </div>

                <!-- Animated Progress Bar at bottom border -->
                <div v-if="isSpielerPlaying(spieler)" class="card-progress-track">
                  <div
                    class="card-progress-fill"
                    :style="{ width: playingProgressPercent + '%' }"
                  ></div>
                </div>
              </div>
            </div>

            <!-- EMPTY TEAM STATE -->
            <div v-else class="empty-state-box">
              <i class="material-icons empty-icon">sports_handball</i>
              <h3>Noch keine aktiven Spieler vorhanden</h3>
              <p>
                Prüfe die Internetverbindung oder synchronisiere die Daten im Tab "Teams &amp; Updates".
              </p>
              <button
                class="btn-secondary"
                @click="syncTeam(team.id)"
              >
                <i class="material-icons">refresh</i> Jetzt synchronisieren
              </button>
            </div>
          </f7-tab>

          <!-- MANAGE / UPDATES / SETTINGS TAB -->
          <f7-tab
            id="tab-manage"
            class="page-content tab-content-padded"
            :tab-active="activeTab === 'tab-manage'"
          >
            <div class="settings-container">
              <!-- TEAMS OVERVIEW SECTION -->
              <div class="section-card">
                <div class="section-header">
                  <div class="section-title-wrap">
                    <i class="material-icons section-icon">groups</i>
                    <h2 class="section-title">Meine Mannschaften ({{ teams.length }} / 3)</h2>
                  </div>
                </div>

                <div v-if="teams.length === 0" class="empty-list-notice">
                  Keine Mannschaften hinzugefügt.
                </div>

                <div v-else class="teams-list">
                  <div v-for="team in teams" :key="team.id" class="team-list-item">
                    <div class="team-item-left">
                      <div class="team-id-badge">{{ team.id }}</div>
                      <div class="team-item-details">
                        <div class="team-item-name">{{ team.name }}</div>
                        <div class="team-item-meta">
                          {{ getPlayersByTeam(team.id).length }} Spieler im Kader
                        </div>
                      </div>
                    </div>

                    <div class="team-item-actions">
                      <button
                        class="btn-icon-danger"
                        @click="confirmRemoveTeam(team)"
                        title="Mannschaft entfernen"
                      >
                        <i class="material-icons">delete_outline</i>
                      </button>
                    </div>
                  </div>
                </div>

                <!-- ADD TEAM FORM DIRECTLY IN TEAMS OVERVIEW SECTION (if < 3 teams) -->
                <div v-if="teams.length < 3" class="add-team-inline-box">
                  <div class="add-team-inline-label">
                    <i class="material-icons">add_circle_outline</i>
                    <span>Neue Mannschaft hinzufügen</span>
                  </div>

                  <div class="team-input-group">
                    <span class="team-input-prefix">#</span>
                    <input
                      type="text"
                      v-model="newTeamIdInput"
                      placeholder="H4R"
                      maxlength="5"
                      class="team-id-input"
                      @keyup.enter="handleAddTeam"
                      autocapitalize="characters"
                    />
                  </div>

                  <div v-if="addTeamError" class="alert-box alert-danger">
                    <i class="material-icons">error_outline</i>
                    <span>{{ addTeamError }}</span>
                  </div>

                  <button
                    class="btn-primary btn-block"
                    :disabled="addTeamLoading || !newTeamIdInput.trim()"
                    @click="handleAddTeam"
                  >
                    <span v-if="!addTeamLoading">
                      <i class="material-icons">add</i> Mannschaft hinzufügen
                    </span>
                    <span v-else class="btn-loading">
                      <span class="spinner"></span> Lädt Kader...
                    </span>
                  </button>
                </div>

                <div v-else class="limit-reached-badge">
                  <i class="material-icons">info</i>
                  <span>Maximale Anzahl von 3 Mannschaften erreicht.</span>
                </div>
              </div>

              <!-- UPDATES & OFFLINE-SYNC SECTION -->
              <div class="section-card">
                <div class="section-header">
                  <div class="section-title-wrap">
                    <i class="material-icons section-icon">cloud_download</i>
                    <h2 class="section-title">Offline-Synchronisation &amp; Updates</h2>
                  </div>
                </div>

                <p class="section-desc">
                  Lade alle Sound-Dateien lokal auf dein Gerät, um das Soundboard auch ohne Hallen-WLAN blitzschnell und offline nutzen zu können.
                </p>

                <!-- Update Status Box -->
                <div v-if="loadingUpdates" class="sync-status-box">
                  <div class="spinner"></div>
                  <span>Prüfe auf Updates...</span>
                </div>

                <div v-else-if="updates.length > 0" class="updates-available-box">
                  <div class="updates-header">
                    <i class="material-icons alert-icon">notifications_active</i>
                    <strong>{{ updates.length }} neue / aktualisierte Sounds verfügbar!</strong>
                  </div>

                  <div class="updates-list">
                    <div v-for="upd in updates" :key="upd.id" class="update-item">
                      <span class="update-name">{{ upd.anzeigename }}</span>
                      <span class="update-team">{{ upd.mannschaftId }} · v{{ upd.version }}</span>
                    </div>
                  </div>

                  <button
                    class="btn-success btn-large btn-block"
                    :disabled="downloadingUpdates"
                    @click="downloadAllUpdates"
                  >
                    <span v-if="!downloadingUpdates">
                      <i class="material-icons">file_download</i> Alle {{ updates.length }} Sounds herunterladen
                    </span>
                    <span v-else class="btn-loading">
                      <span class="spinner"></span> Lade herunter ({{ downloadProgress }} / {{ updates.length }})...
                    </span>
                  </button>
                </div>

                <div v-else class="all-synced-box">
                  <i class="material-icons success-icon">check_circle</i>
                  <span>Alle Kader und Torsongs sind auf dem neuesten Stand!</span>
                </div>

                <div class="section-btn-row">
                  <button
                    class="btn-secondary btn-block"
                    :disabled="loadingUpdates || downloadingUpdates"
                    @click="checkForUpdates"
                  >
                    <i class="material-icons">refresh</i> Auf Updates prüfen
                  </button>
                </div>
              </div>

              <!-- SETTINGS & SYSTEM SECTION -->
              <div class="section-card">
                <div class="section-header">
                  <div class="section-title-wrap">
                    <i class="material-icons section-icon">settings</i>
                    <h2 class="section-title">Server &amp; Einstellungen</h2>
                  </div>
                </div>

                <div class="settings-field">
                  <label class="field-label">API Server Base-URL</label>
                  <div class="url-input-group">
                    <input
                      type="text"
                      v-model="baseUrl"
                      @change="saveSettings"
                      :placeholder="defaultApiUrl"
                      class="text-input"
                    />
                  </div>
                  <span class="field-hint">Standard: <code>{{ defaultApiUrl }}</code></span>
                </div>

                <div class="danger-zone">
                  <h3 class="danger-title">App zurücksetzen</h3>
                  <p class="danger-desc">
                    Löscht alle gecachten Daten, Teams und gespeicherten Sounds aus der lokalen Datenbank.
                  </p>
                  <button class="btn-danger btn-block" @click="confirmResetApp">
                    <i class="material-icons">warning</i> APP ZURÜCKSETZEN
                  </button>
                </div>
              </div>
            </div>
          </f7-tab>
        </f7-tabs>

        <!-- STICKY BOTTOM PLAYER (Appears when sound is playing) -->
        <div v-if="playingSpieler" class="sticky-player-bar">
          <div class="sticky-progress-track">
            <div
              class="sticky-progress-fill"
              :style="{ width: playingProgressPercent + '%' }"
            ></div>
          </div>

          <div class="sticky-player-content">
            <div class="sticky-player-left">
              <i class="material-icons playing-icon">music_note</i>
              <div class="sticky-player-meta">
                <div class="sticky-player-name">{{ playingSpieler.anzeigename }}</div>
                <div class="sticky-player-sub">Torjubel läuft · {{ playingSpieler.mannschaftId }}</div>
              </div>
            </div>

            <div class="sticky-player-right">
              <div class="sticky-time-pill">{{ playingSecondsRemaining.toFixed(1) }}s</div>
              <button class="sticky-stop-btn" @click="stopSound()">
                <i class="material-icons">stop</i>
                <span>STOPP</span>
              </button>
            </div>
          </div>
        </div>

        <!-- ADD TEAM MODAL DIALOG -->
        <div v-if="showAddTeamModal" class="custom-modal-backdrop" @click.self="closeAddTeamModal">
          <div class="custom-modal-card">
            <div class="modal-header">
              <h3 class="modal-title">Mannschaft hinzufügen</h3>
              <button class="btn-icon-dark" @click="closeAddTeamModal" aria-label="Schließen">
                <i class="material-icons">close</i>
              </button>
            </div>

            <div class="modal-body">
              <p class="modal-instruction">
                Gib die 3-stellige Mannschafts-ID ein (z. B. <strong>#H4R</strong> oder <strong>H4R</strong>).
              </p>

              <div class="team-input-group">
                <span class="team-input-prefix">#</span>
                <input
                  type="text"
                  v-model="newTeamIdInput"
                  placeholder="H4R"
                  maxlength="5"
                  class="team-id-input"
                  @keyup.enter="handleAddTeam"
                  autocapitalize="characters"
                  autofocus
                />
              </div>

              <div v-if="addTeamError" class="alert-box alert-danger">
                <i class="material-icons">error_outline</i>
                <span>{{ addTeamError }}</span>
              </div>
            </div>

            <div class="modal-footer">
              <button class="btn-secondary" @click="closeAddTeamModal">Abbrechen</button>
              <button
                class="btn-primary"
                :disabled="addTeamLoading || !newTeamIdInput.trim()"
                @click="handleAddTeam"
              >
                <span v-if="!addTeamLoading">Hinzufügen</span>
                <span v-else class="btn-loading">
                  <span class="spinner"></span> Lädt...
                </span>
              </button>
            </div>
          </div>
        </div>
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

const isLocal = !!import.meta.env.DEV;

const DEFAULT_API_URL = isLocal
  ? "http://localhost:3000"
  : "https://sound.beugelbuddel.de";

export default {
  name: "App",
  data() {
    return {
      // Teams configuration (Max 3)
      teams: [],
      activeTab: "tab-team-0",
      players: [],

      // Audio Playback Engine (Full Length Playback)
      playingSpieler: null,
      playingDuration: 0,
      playingSecondsRemaining: 0.0,
      playingProgressPercent: 100,
      audioCountdownTimer: null,
      audioProgressTimer: null,
      currentMedia: null,
      htmlAudio: null,

      // Add Team Dialog
      showAddTeamModal: false,
      newTeamIdInput: "",
      addTeamLoading: false,
      addTeamError: "",

      // Updates & Settings
      baseUrl: DEFAULT_API_URL,
      updates: [],
      updateCount: 0,
      loadingUpdates: false,
      downloadingUpdates: false,
      downloadProgress: 0,
    };
  },
  computed: {
    defaultApiUrl() {
      return DEFAULT_API_URL;
    },
    isDev() {
      return isLocal;
    },
    currentActiveTeam() {
      if (this.activeTab && this.activeTab.startsWith("tab-team-")) {
        const index = parseInt(this.activeTab.replace("tab-team-", ""), 10);
        return this.teams[index] || null;
      }
      return null;
    },
  },
  created() {
    this.initDatabase();
    this.enableKeepAwake();
  },
  mounted() {
    if (this.teams.length > 0) {
      this.checkForUpdates();
    }
    window.addEventListener("online", this.onNetworkOnline);
    document.addEventListener("visibilitychange", this.onVisibilityChange);
  },
  beforeDestroy() {
    this.stopSound();
    window.removeEventListener("online", this.onNetworkOnline);
    document.removeEventListener("visibilitychange", this.onVisibilityChange);
  },
  methods: {
    // -------------------------------------------------------------
    // DATABASE & INITIALIZATION
    // -------------------------------------------------------------
    initDatabase() {
      db.defaults({
        teams: [],
        players: [],
        settings: { baseUrl: DEFAULT_API_URL },
      }).write();

      const savedSettings = db.get("settings").value();
      if (savedSettings && savedSettings.baseUrl) {
        // Automatically adapt default if switching between local dev and production
        if (isLocal && (savedSettings.baseUrl === "https://sound.beugelbuddel.de" || savedSettings.baseUrl === "https://beugelbuddel.de")) {
          this.baseUrl = "http://localhost:3000";
          db.set("settings.baseUrl", this.baseUrl).write();
        } else if (!isLocal && (savedSettings.baseUrl === "http://localhost:3000" || savedSettings.baseUrl === "http://127.0.0.1:3000" || savedSettings.baseUrl === "https://beugelbuddel.de")) {
          this.baseUrl = "https://sound.beugelbuddel.de";
          db.set("settings.baseUrl", this.baseUrl).write();
        } else {
          this.baseUrl = savedSettings.baseUrl.replace(/\/+$/, "");
        }
      } else {
        this.baseUrl = DEFAULT_API_URL;
        db.set("settings.baseUrl", this.baseUrl).write();
      }

      this.teams = db.get("teams").value() || [];
      const storedPlayers = db.get("players").value() || [];
      // Migration: Ensure all previously stored players have isDownloaded flag set to true
      let needsDbSave = false;
      this.players = storedPlayers.map((p) => {
        if (p.isDownloaded === undefined) {
          needsDbSave = true;
          return { ...p, isDownloaded: true };
        }
        return p;
      });
      if (needsDbSave) {
        db.set("players", this.players).write();
      }

      // Set initial active tab
      if (this.teams.length > 0) {
        this.activeTab = "tab-team-0";
      } else {
        this.activeTab = "tab-onboarding";
      }
    },

    saveSettings() {
      this.baseUrl = this.baseUrl.trim().replace(/\/+$/, "");
      db.set("settings.baseUrl", this.baseUrl).write();
    },

    enableKeepAwake() {
      if (window.plugins && window.plugins.insomnia && window.plugins.insomnia.keepAwake) {
        window.plugins.insomnia.keepAwake();
      }
    },

    // -------------------------------------------------------------
    // NAVIGATION
    // -------------------------------------------------------------
    setActiveTab(tabId) {
      this.activeTab = tabId;
    },

    getInitial(name) {
      if (!name) return "?";
      return name.trim().charAt(0).toUpperCase();
    },

    getPlayersByTeam(teamId) {
      const cleanTeamId = this.normalizeId(teamId);
      return this.players.filter(
        (p) => this.normalizeId(p.mannschaftId) === cleanTeamId && p.active && p.isDownloaded
      );
    },

    getPlayersForTeam(teamId) {
      const teamPlayers = [...this.getPlayersByTeam(teamId)];
      // Sort alphabetically by Anzeigename
      teamPlayers.sort((a, b) => (a.anzeigename || "").localeCompare(b.anzeigename || ""));
      return teamPlayers;
    },

    normalizeId(id) {
      if (!id) return "";
      return id.toString().trim().replace(/^#+/, "").toUpperCase().trim();
    },

    // -------------------------------------------------------------
    // ADD & REMOVE TEAMS (MAX 3 TEAMS)
    // -------------------------------------------------------------
    openAddTeamModal() {
      if (this.teams.length >= 3) {
        alert("Du kannst maximal 3 Mannschaften hinzufügen.");
        return;
      }
      this.newTeamIdInput = "";
      this.addTeamError = "";
      this.showAddTeamModal = true;
    },

    closeAddTeamModal() {
      this.showAddTeamModal = false;
      this.addTeamError = "";
      this.newTeamIdInput = "";
    },

    async handleAddTeam() {
      const input = this.newTeamIdInput.trim();
      if (!input) return;

      const rawId = this.normalizeId(input);
      if (!rawId) {
        this.addTeamError = "Bitte eine gültige Mannschafts-ID eingeben.";
        return;
      }

      const formattedId = `#${rawId}`;

      // Check for limit (Max 3)
      if (this.teams.length >= 3) {
        this.addTeamError = "Du kannst maximal 3 Mannschaften hinzufügen.";
        return;
      }

      // Check for duplicate
      if (this.teams.some((t) => this.normalizeId(t.id) === rawId)) {
        this.addTeamError = `Mannschaft ${formattedId} wurde bereits hinzugefügt.`;
        return;
      }

      this.addTeamLoading = true;
      this.addTeamError = "";

      try {
        // Fetch active players for this team from API
        const playersUrl = `${this.baseUrl}/api/info/${rawId}`;
        const playersResponse = await axios.get(playersUrl, { timeout: 10000 });

        if (!Array.isArray(playersResponse.data)) {
          throw new Error("Ungültiges Antwortformat vom Server.");
        }

        const teamPlayers = playersResponse.data;

        // Try to fetch official team name from /api/mannschaften
        let teamName = formattedId;
        try {
          const mannschaftenUrl = `${this.baseUrl}/api/mannschaften`;
          const mResponse = await axios.get(mannschaftenUrl, { timeout: 5000 });
          if (Array.isArray(mResponse.data)) {
            const foundTeam = mResponse.data.find(
              (m) => this.normalizeId(m.id) === rawId
            );
            if (foundTeam && foundTeam.name) {
              teamName = foundTeam.name;
            }
          }
        } catch (e) {
          console.warn("Could not fetch /api/mannschaften, using ID as name:", e);
        }

        const newTeam = {
          id: formattedId,
          rawId: rawId,
          name: teamName,
          addedAt: new Date().toISOString(),
        };

        // Save new team
        const currentTeams = db.get("teams").value() || [];
        currentTeams.push(newTeam);
        db.set("teams", currentTeams).write();
        this.teams = currentTeams;

        // Upsert players
        this.upsertPlayersForTeam(rawId, teamPlayers);

        // Switch to the newly added team tab
        const newIndex = this.teams.length - 1;
        this.activeTab = `tab-team-${newIndex}`;

        this.closeAddTeamModal();
        this.newTeamIdInput = "";
        this.addTeamError = "";

        // Check for updates / download sounds
        this.checkForUpdates();
      } catch (err) {
        console.error("Error loading team:", err);
        if (err.response && err.response.status === 404) {
          this.addTeamError = `Mannschaft '${formattedId}' wurde auf dem Server nicht gefunden.`;
        } else if (err.code === "ECONNABORTED" || !navigator.onLine) {
          this.addTeamError = "Keine Verbindung zum Server. Bitte Netzwerk prüfen.";
        } else {
          this.addTeamError = `Fehler beim Laden: ${err.message || "Unbekannter Fehler"}`;
        }
      } finally {
        this.addTeamLoading = false;
      }
    },

    upsertPlayersForTeam(rawId, newPlayersList) {
      const storedPlayers = db.get("players").value() || [];
      const cleanTeamId = this.normalizeId(rawId);

      // Keep players of other teams
      const otherPlayers = storedPlayers.filter(
        (p) => this.normalizeId(p.mannschaftId) !== cleanTeamId
      );

      // For new players, keep existing download status if already downloaded, else mark isDownloaded: false
      const mergedTeamPlayers = newPlayersList.map((np) => {
        const existing = storedPlayers.find((p) => p.username === np.username);
        return {
          ...np,
          isDownloaded: existing ? !!existing.isDownloaded : false,
          version: existing ? existing.version : 0,
        };
      });

      const updatedList = [...otherPlayers, ...mergedTeamPlayers];
      db.set("players", updatedList).write();
      this.players = updatedList;
    },

    confirmRemoveTeam(team) {
      if (confirm(`Möchtest du die Mannschaft "${team.name}" (${team.id}) wirklich entfernen?`)) {
        this.removeTeam(team.id);
      }
    },

    removeTeam(teamId) {
      const cleanId = this.normalizeId(teamId);
      const remainingTeams = this.teams.filter((t) => this.normalizeId(t.id) !== cleanId);
      db.set("teams", remainingTeams).write();
      this.teams = remainingTeams;

      // Remove players of that team
      const remainingPlayers = this.players.filter(
        (p) => this.normalizeId(p.mannschaftId) !== cleanId
      );
      db.set("players", remainingPlayers).write();
      this.players = remainingPlayers;

      // Update active tab
      if (this.teams.length > 0) {
        this.activeTab = "tab-team-0";
      } else {
        this.activeTab = "tab-onboarding";
      }

      this.checkForUpdates();
    },

    async syncTeam(teamId) {
      const rawId = this.normalizeId(teamId);
      try {
        const response = await axios.get(`${this.baseUrl}/api/info/${rawId}`, { timeout: 10000 });
        if (Array.isArray(response.data)) {
          this.upsertPlayersForTeam(rawId, response.data);
          this.checkForUpdates();
          alert(`Mannschaft ${teamId} erfolgreich synchronisiert (${response.data.length} Spieler).`);
        }
      } catch (err) {
        alert(`Fehler beim Synchronisieren von ${teamId}: ` + (err.message || err));
      }
    },

    // -------------------------------------------------------------
    // SOUNDBOARD AUDIO ENGINE (FULL LENGTH PLAYBACK)
    // -------------------------------------------------------------
    isSpielerPlaying(spieler) {
      return this.playingSpieler && this.playingSpieler.id === spieler.id;
    },

    togglePlay(spieler) {
      // Toggle: If this player is already playing -> STOP immediately
      if (this.isSpielerPlaying(spieler)) {
        this.stopSound();
        return;
      }

      // Otherwise: Start playing new player (stops any previous audio first)
      this.playSound(spieler);
    },

    async playSound(spieler) {
      await this.stopSound();

      this.playingSpieler = spieler;
      this.playingDuration = 0;
      this.playingSecondsRemaining = 0.0;
      this.playingProgressPercent = 100;

      // Haptic feedback if available
      if (navigator.vibrate) {
        navigator.vibrate(50);
      }

      const soundUrl = `${this.baseUrl}/uploads/${spieler.username}.mp3`;

      // Check if running in Cordova environment with Media plugin
      const hasCordovaMedia =
        typeof window.cordova !== "undefined" &&
        typeof window.Media !== "undefined" &&
        typeof cordova.file !== "undefined";

      if (hasCordovaMedia) {
        this.playCordovaAudio(spieler, soundUrl);
      } else {
        this.playHtmlAudio(spieler, soundUrl);
      }

      // Start progress timer to update remaining time and progress bar smoothly
      this.startPlaybackProgressTimer();
    },

    playCordovaAudio(spieler, fallbackUrl) {
      const localFilePath = cordova.file.dataDirectory + spieler.username + ".mp3";
      const audioSource = localFilePath || fallbackUrl;

      try {
        const onMediaSuccess = () => {
          console.log("Cordova Audio playback completed");
          this.stopSound();
        };

        const onMediaError = (err) => {
          console.warn("Cordova Audio Error with file, trying fallback stream:", err);
          if (audioSource !== fallbackUrl) {
            try {
              this.currentMedia = new window.Media(
                fallbackUrl,
                () => {
                  console.log("Cordova Fallback Stream completed");
                  this.stopSound();
                },
                (err2) => {
                  console.error("Cordova Stream Audio Error:", err2);
                  this.stopSound();
                }
              );
              this.currentMedia.play();
            } catch (e) {
              console.error("Cordova stream fallback failed:", e);
              this.stopSound();
            }
          } else {
            this.stopSound();
          }
        };

        this.currentMedia = new window.Media(audioSource, onMediaSuccess, onMediaError);
        this.currentMedia.play();
      } catch (e) {
        console.warn("Cordova media init failed, falling back to HTML5 audio:", e);
        this.playHtmlAudio(spieler, fallbackUrl);
      }
    },

    playHtmlAudio(spieler, soundUrl) {
      try {
        if (this.htmlAudio) {
          this.htmlAudio.pause();
          this.htmlAudio = null;
        }

        const audio = new Audio(soundUrl);
        this.htmlAudio = audio;
        this.htmlAudio.currentTime = 0;

        audio.addEventListener("loadedmetadata", () => {
          if (this.htmlAudio === audio && audio.duration && isFinite(audio.duration) && audio.duration > 0) {
            this.playingDuration = audio.duration;
            this.playingSecondsRemaining = Math.max(0, audio.duration - audio.currentTime);
          }
        });

        audio.addEventListener("ended", () => {
          if (this.htmlAudio === audio) {
            this.stopSound();
          }
        });

        audio.addEventListener("error", (err) => {
          console.error("HTML5 Audio playback error:", err);
          if (this.htmlAudio === audio) {
            this.stopSound();
          }
        });

        audio.play().catch((err) => {
          console.error("HTML5 Audio play failed:", err);
          if (this.htmlAudio === audio) {
            this.stopSound();
          }
        });
      } catch (e) {
        console.error("Audio error:", e);
        this.stopSound();
      }
    },

    startPlaybackProgressTimer() {
      const INTERVAL_MS = 50;
      const startTime = Date.now();

      this.audioProgressTimer = setInterval(() => {
        if (!this.playingSpieler) {
          clearInterval(this.audioProgressTimer);
          this.audioProgressTimer = null;
          return;
        }

        // 1. HTML5 Audio
        if (this.htmlAudio) {
          const audio = this.htmlAudio;
          if (audio.duration && isFinite(audio.duration) && audio.duration > 0) {
            this.playingDuration = audio.duration;
            const remaining = Math.max(0, audio.duration - audio.currentTime);
            this.playingSecondsRemaining = remaining;
            this.playingProgressPercent = Math.max(0, Math.min(100, (remaining / audio.duration) * 100));
          } else {
            const elapsed = (Date.now() - startTime) / 1000;
            this.playingSecondsRemaining = elapsed;
            this.playingProgressPercent = 100;
          }
          return;
        }

        // 2. Cordova Media
        if (this.currentMedia) {
          const media = this.currentMedia;
          try {
            const dur = media.getDuration();
            if (dur && dur > 0) {
              this.playingDuration = dur;
            }

            media.getCurrentPosition((pos) => {
              if (this.currentMedia !== media) return;
              if (pos >= 0) {
                if (this.playingDuration > 0) {
                  const remaining = Math.max(0, this.playingDuration - pos);
                  this.playingSecondsRemaining = remaining;
                  this.playingProgressPercent = Math.max(0, Math.min(100, (remaining / this.playingDuration) * 100));
                } else {
                  this.playingSecondsRemaining = pos;
                  this.playingProgressPercent = 100;
                }
              }
            }, (err) => {
              console.warn("Error getting Cordova media position:", err);
            });
          } catch (e) {
            console.warn("Error in Cordova progress update:", e);
          }
          return;
        }

        // Fallback before audio is ready
        const elapsed = (Date.now() - startTime) / 1000;
        this.playingSecondsRemaining = elapsed;
      }, INTERVAL_MS);
    },

    stopSound() {
      // Clear timers
      if (this.audioProgressTimer) {
        clearInterval(this.audioProgressTimer);
        this.audioProgressTimer = null;
      }
      if (this.audioCountdownTimer) {
        clearTimeout(this.audioCountdownTimer);
        this.audioCountdownTimer = null;
      }

      // Stop Cordova Media
      if (this.currentMedia) {
        try {
          const media = this.currentMedia;
          this.currentMedia = null;
          media.stop();
          media.release();
        } catch (e) {
          console.warn("Error releasing Cordova media:", e);
        }
      }

      // Stop HTML5 Audio
      if (this.htmlAudio) {
        try {
          const audio = this.htmlAudio;
          this.htmlAudio = null;
          audio.pause();
          audio.currentTime = 0;
          audio.onended = null;
          audio.onerror = null;
          audio.ontimeupdate = null;
          audio.onloadedmetadata = null;
        } catch (e) {
          console.warn("Error stopping HTML5 audio:", e);
        }
      }

      if (this.playingSpieler && navigator.vibrate) {
        navigator.vibrate(25);
      }

      this.playingSpieler = null;
      this.playingDuration = 0;
      this.playingSecondsRemaining = 0.0;
      this.playingProgressPercent = 100;
    },

    // -------------------------------------------------------------
    // OFFLINE CACHING & UPDATES
    // -------------------------------------------------------------
    onNetworkOnline() {
      if (this.teams.length > 0) {
        this.checkForUpdates();
      }
    },

    onVisibilityChange() {
      if (document.visibilityState === "visible" && this.teams.length > 0) {
        this.checkForUpdates();
      }
    },

    async checkForUpdates() {
      if (this.teams.length === 0) return;

      this.loadingUpdates = true;
      this.updates = [];

      try {
        let allPendingUpdates = [];

        // Check updates for each registered team WITHOUT modifying active roster
        for (const team of this.teams) {
          const rawId = this.normalizeId(team.id);
          try {
            const res = await axios.get(`${this.baseUrl}/api/info/${rawId}`, { timeout: 8000 });
            if (Array.isArray(res.data)) {
              res.data.forEach((serverSpieler) => {
                const localSpieler = this.players.find(
                  (p) => p.username === serverSpieler.username
                );
                const localVersion = localSpieler ? localSpieler.version || 0 : 0;
                const isDownloaded = localSpieler ? !!localSpieler.isDownloaded : false;

                // Needs download if not present, server version is newer or file not yet downloaded
                if (!localSpieler || serverSpieler.version > localVersion || !isDownloaded) {
                  allPendingUpdates.push(serverSpieler);
                }
              });
            }
          } catch (e) {
            console.warn(`Could not check updates for team ${team.id}:`, e);
          }
        }

        this.updates = allPendingUpdates;
        this.updateCount = allPendingUpdates.length;
      } catch (err) {
        console.error("Check for updates error:", err);
      } finally {
        this.loadingUpdates = false;
      }
    },

    async downloadAllUpdates() {
      if (this.updates.length === 0) return;

      this.downloadingUpdates = true;
      this.downloadProgress = 0;

      const updatesToProcess = [...this.updates];

      for (let i = 0; i < updatesToProcess.length; i++) {
        const spieler = updatesToProcess[i];
        await this.downloadSingleFile(spieler);
        this.downloadProgress = i + 1;
        this.updates = this.updates.filter((u) => u.username !== spieler.username);
        this.updateCount = this.updates.length;
      }

      this.downloadingUpdates = false;
      alert("Alle Torsongs wurden erfolgreich heruntergeladen und sind offline bereit!");
    },

    async downloadSingleFile(spieler) {
      const soundUrl = `${this.baseUrl}/uploads/${spieler.username}.mp3`;

      // If running on Cordova with cordova.file
      const isCordova = typeof window.cordova !== "undefined" && typeof cordova.file !== "undefined";

      if (isCordova) {
        try {
          if (typeof window.FileTransfer !== "undefined") {
            await new Promise((resolve, reject) => {
              const fileTransfer = new window.FileTransfer();
              const targetPath = cordova.file.dataDirectory + spieler.username + ".mp3";
              fileTransfer.download(encodeURI(soundUrl), targetPath, resolve, reject, true);
            });
          } else {
            // Modern Cordova: Fetch MP3 blob and save using cordova-plugin-file
            const response = await fetch(soundUrl);
            if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
            const blob = await response.blob();

            await new Promise((resolve, reject) => {
              window.resolveLocalFileSystemURL(
                cordova.file.dataDirectory,
                (dirEntry) => {
                  dirEntry.getFile(
                    spieler.username + ".mp3",
                    { create: true, exclusive: false },
                    (fileEntry) => {
                      fileEntry.createWriter((fileWriter) => {
                        fileWriter.onwriteend = () => resolve(true);
                        fileWriter.onerror = (e) => reject(e);
                        fileWriter.write(blob);
                      }, reject);
                    },
                    reject
                  );
                },
                reject
              );
            });
          }
        } catch (error) {
          console.warn("Cordova File Download Error:", error);
        }
      } else {
        // Browser environment: pre-fetch / cache in browser or mark updated
        try {
          if ("caches" in window) {
            const cache = await caches.open("7secs-sounds-cache");
            await cache.add(soundUrl);
          } else {
            await axios.get(soundUrl, { responseType: "blob" });
          }
        } catch (e) {
          console.warn("Browser download pre-fetch simulated:", e);
        }
      }

      // Update local version in db and activate player on Soundboard
      const currentPlayers = db.get("players").value() || [];
      const index = currentPlayers.findIndex((p) => p.username === spieler.username);
      if (index >= 0) {
        currentPlayers[index] = { ...currentPlayers[index], ...spieler, version: spieler.version, isDownloaded: true };
      } else {
        currentPlayers.push({ ...spieler, version: spieler.version, isDownloaded: true });
      }
      db.set("players", currentPlayers).write();
      this.players = currentPlayers;
      return true;
    },

    confirmResetApp() {
      if (
        confirm(
          "Möchtest du die App wirklich zurücksetzen? Alle Mannschaften und Einstellungen werden gelöscht."
        )
      ) {
        this.resetApp();
      }
    },

    resetApp() {
      this.stopSound();
      db.set("teams", []).write();
      db.set("players", []).write();
      this.teams = [];
      this.players = [];
      this.updates = [];
      this.updateCount = 0;
      this.activeTab = "tab-onboarding";
      alert("Die App wurde erfolgreich zurückgesetzt.");
    },
  },
};
</script>

<style>
/* ==========================================================================
   7SECS DESIGN SYSTEM TOKENS & GLOBAL STYLES (APP_DESIGN_SYSTEM.md)
   ========================================================================== */
:root {
  --color-brand: #1E40AF;
  --color-brand-light: #DBEAFE;
  --color-brand-mid: #3B82F6;
  --color-accent: #F59E0B;
  --color-accent-light: #FEF3C7;
  --color-success: #16A34A;
  --color-success-light: #DCFCE7;
  --color-danger: #DC2626;
  --color-danger-light: #FEF2F2;

  --color-bg: #F8FAFC;
  --color-surface: #FFFFFF;
  --color-border: #E2E8F0;
  --color-border-strong: #CBD5E1;

  --color-text-primary: #0F172A;
  --color-text-muted: #64748B;
  --color-text-light: #94A3B8;

  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 16px;
  --radius-full: 9999px;

  --shadow-subtle: 0 2px 6px rgba(15, 23, 42, 0.06);
  --shadow-active: 0 8px 20px -2px rgba(245, 158, 11, 0.35);
  --shadow-sticky: 0 -4px 20px rgba(15, 23, 42, 0.2);

  --f7-navbar-height: 56px;
}

body, html, #app {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'SF Pro Display', Roboto, sans-serif;
  color: var(--color-text-primary);
  background-color: var(--color-bg);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin: 0;
  padding: 0;
}

.app-page {
  background-color: var(--color-bg) !important;
}

/* ==========================================================================
   NAVBAR & HEADER (APP_DESIGN_SYSTEM.md Specification)
   ========================================================================== */
.app-navbar {
  background-color: var(--color-surface) !important;
  color: var(--color-text-primary) !important;
  border-bottom: 1.5px solid var(--color-border) !important;
  box-shadow: 0 2px 6px rgba(15, 23, 42, 0.04) !important;
  height: calc(56px + env(safe-area-inset-top, 0px)) !important;
  position: fixed !important;
  top: 0 !important;
  left: 0 !important;
  right: 0 !important;
  z-index: 600 !important;
}

.navbar-inner-custom {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  padding: env(safe-area-inset-top, 0px) 16px 0 16px;
  height: 56px;
  box-sizing: border-box;
}

.navbar-brand-section {
  display: flex;
  align-items: center;
  gap: 10px;
}

.navbar-logo-icon {
  width: 34px;
  height: 34px;
  background-color: var(--color-brand);
  color: #FFFFFF;
  border-radius: var(--radius-sm);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 4px rgba(30, 64, 175, 0.25);
  flex-shrink: 0;
}

.navbar-logo-icon i {
  font-size: 20px;
}

.navbar-title-wrap {
  display: flex;
  flex-direction: column;
}

.navbar-title {
  font-size: 19px;
  font-weight: 900;
  letter-spacing: -0.5px;
  color: var(--color-text-primary);
  line-height: 1.15;
}

.navbar-subtitle {
  font-size: 11px;
  font-weight: 600;
  color: var(--color-text-muted);
  letter-spacing: 0.02em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 160px;
}

.navbar-right-section {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* Team ID Badge (Section 5.1) */
.team-id-badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 4px 10px;
  background-color: var(--color-brand-light);
  color: var(--color-brand);
  border: 2px solid var(--color-brand);
  border-radius: var(--radius-md);
  font-family: 'JetBrains Mono', 'SF Mono', monospace;
  font-size: 14px;
  font-weight: 900;
  letter-spacing: 0.08em;
  box-sizing: border-box;
}

.team-id-badge.header-badge {
  font-size: 15px;
  padding: 4px 10px;
}

.team-id-badge.large {
  font-size: 20px;
  padding: 6px 14px;
  border-width: 2px;
}

.team-id-badge.compact {
  font-size: 11px;
  padding: 2px 6px;
  border-width: 1.5px;
  border-radius: 4px;
}

/* Live Playing Indicator (Section 2.1 & 5.2) */
.navbar-playing-pill {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background-color: var(--color-accent-light);
  color: #92400E;
  border: 1.5px solid var(--color-accent);
  padding: 4px 10px;
  border-radius: var(--radius-full);
  box-shadow: 0 2px 8px rgba(245, 158, 11, 0.25);
  animation: pulsePill 1.5s infinite;
}

@keyframes pulsePill {
  0% { transform: scale(1); }
  50% { transform: scale(1.03); }
  100% { transform: scale(1); }
}

.live-pulse-dot {
  width: 7px;
  height: 7px;
  background-color: var(--color-accent);
  border-radius: 50%;
  display: inline-block;
  animation: blink 0.8s infinite alternate ease-in-out;
}

@keyframes blink {
  0% { opacity: 0.3; }
  100% { opacity: 1; }
}

.live-time-text {
  font-family: 'JetBrains Mono', monospace;
  font-size: 13px;
  font-weight: 900;
  color: #92400E;
}

.nav-waveform-bars {
  display: flex;
  align-items: flex-end;
  gap: 2px;
  height: 14px;
}

.nav-bar {
  width: 2.5px;
  height: 4px;
  background-color: var(--color-accent);
  border-radius: 1px;
  animation: soundBars 0.6s infinite alternate ease-in-out;
}

.nav-bar:nth-child(1) { animation-duration: 0.45s; }
.nav-bar:nth-child(2) { animation-duration: 0.35s; }
.nav-bar:nth-child(3) { animation-duration: 0.55s; }
.nav-bar:nth-child(4) { animation-duration: 0.4s; }

.navbar-manage-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  background-color: var(--color-surface);
  color: var(--color-text-muted);
  border: 1px solid var(--color-border);
  padding: 4px 8px;
  border-radius: var(--radius-sm);
  font-size: 12px;
  font-weight: 600;
}

.navbar-manage-badge i {
  font-size: 16px;
  color: var(--color-brand);
}

@keyframes soundBars {
  0% { height: 4px; }
  100% { height: 14px; }
}

.btn-icon {
  background: transparent;
  border: none;
  color: #FFFFFF;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: var(--radius-full);
  cursor: pointer;
  outline: none;
  transition: background 0.15s;
}

.btn-icon:active {
  background: rgba(255, 255, 255, 0.2);
}

.btn-icon-dark {
  background: transparent;
  border: none;
  color: var(--color-text-primary);
  display: flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: var(--radius-full);
  cursor: pointer;
  outline: none;
  transition: background 0.15s;
}

.btn-icon-dark:active {
  background: var(--color-border);
}

/* ==========================================================================
   TOOLBAR & TABS (BOTTOM TABBAR)
   ========================================================================== */
.app-toolbar {
  background: #FFFFFF !important;
  border-top: 1.5px solid var(--color-border) !important;
  height: calc(58px + env(safe-area-inset-bottom, 0px)) !important;
  padding-bottom: env(safe-area-inset-bottom, 0px) !important;
  box-shadow: 0 -2px 8px rgba(0, 0, 0, 0.04);
}

.toolbar-tab-link {
  color: var(--color-text-muted) !important;
  font-weight: 600;
  font-size: 11px !important;
  transition: color 0.15s ease;
}

.toolbar-tab-link.tab-link-active {
  color: var(--color-brand) !important;
}

.toolbar-tab-link i.material-icons {
  font-size: 22px;
  margin-bottom: 2px;
}

.tab-icon-wrapper {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.tab-update-dot {
  position: absolute;
  top: -2px;
  right: -4px;
  width: 9px;
  height: 9px;
  background-color: var(--color-danger);
  border-radius: 50%;
  border: 1.5px solid #FFFFFF;
  box-shadow: 0 0 0 1px rgba(220, 38, 38, 0.4);
  animation: dotPulse 2s infinite ease-in-out;
}

@keyframes dotPulse {
  0% {
    transform: scale(1);
    box-shadow: 0 0 0 1px rgba(220, 38, 38, 0.4);
  }
  50% {
    transform: scale(1.2);
    box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.18);
  }
  100% {
    transform: scale(1);
    box-shadow: 0 0 0 1px rgba(220, 38, 38, 0.4);
  }
}

.tab-badge {
  position: absolute;
  top: -4px;
  right: -8px;
  background: var(--color-danger);
  color: #FFFFFF;
  font-size: 10px;
  font-weight: 800;
  padding: 1px 5px;
  border-radius: var(--radius-full);
}

.tab-content-padded {
  padding-top: calc(56px + env(safe-area-inset-top, 0px) + 8px) !important;
  padding-bottom: calc(58px + env(safe-area-inset-bottom, 0px) + 8px) !important;
  padding-left: calc(8px + env(safe-area-inset-left, 0px)) !important;
  padding-right: calc(8px + env(safe-area-inset-right, 0px)) !important;
  box-sizing: border-box;
}

/* ==========================================================================
   TEAM HEADER BAR (COMPACT STATUS LINE)
   ========================================================================== */
.team-header-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #FFFFFF;
  border: 1.5px solid var(--color-border);
  border-radius: var(--radius-md);
  padding: 6px 10px;
  margin-bottom: 8px;
  box-shadow: 0 1px 3px rgba(15, 23, 42, 0.04);
}

.team-header-left {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 0;
}

.team-name {
  margin: 0;
  font-size: 13px;
  font-weight: 800;
  color: var(--color-text-primary);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.team-player-count {
  font-size: 12px;
  color: var(--color-text-muted);
  font-weight: 600;
  white-space: nowrap;
  margin-left: 8px;
}

/* ==========================================================================
   SOUNDBOARD GRID & COMPACT PLAYER BUZZER TILES
   ========================================================================== */
.soundboard-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 6px;
}

@media (min-width: 600px) {
  .soundboard-grid {
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
  }
}

@media (min-width: 900px) {
  .soundboard-grid {
    grid-template-columns: repeat(5, 1fr);
    gap: 12px;
  }
}

.soundboard-card {
  background: var(--color-surface);
  border: 1.5px solid var(--color-border);
  border-radius: var(--radius-md);
  padding: 6px 8px;
  box-shadow: var(--shadow-subtle);
  display: flex;
  align-items: center;
  gap: 6px;
  min-height: 48px;
  height: 48px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: transform 0.08s ease, border-color 0.15s ease, background-color 0.15s ease, box-shadow 0.15s ease;
  user-select: none;
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
}

.soundboard-card:active {
  transform: scale(0.96);
  background-color: var(--color-brand-light);
  border-color: var(--color-brand);
}

/* PLAYING STATE (7s COUNTDOWN & HIGHLIGHT) */
.soundboard-card.is-playing {
  background-color: var(--color-accent-light);
  border: 2px solid var(--color-accent);
  box-shadow: var(--shadow-active);
}

.soundboard-card.is-playing:active {
  transform: scale(0.96);
  background-color: #FDE68A;
}

.avatar-box {
  width: 28px;
  height: 28px;
  border-radius: var(--radius-sm);
  background-color: var(--color-brand-light);
  color: var(--color-brand);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  font-weight: 800;
  flex-shrink: 0;
  transition: all 0.15s ease;
}

.avatar-box.avatar-playing {
  background-color: var(--color-danger);
  color: #FFFFFF;
}

.card-live-icon {
  font-size: 16px;
}

.player-name {
  flex: 1;
  min-width: 0;
  font-size: 13px;
  font-weight: 700;
  color: var(--color-text-primary);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.2;
}

.countdown-badge {
  font-size: 10px;
  font-weight: 800;
  color: #92400E;
  background: #FFFFFF;
  padding: 2px 4px;
  border-radius: 4px;
  border: 1px solid var(--color-accent);
  font-family: 'JetBrains Mono', monospace;
  white-space: nowrap;
  flex-shrink: 0;
}

/* Progress track attached to bottom border of tile */
.card-progress-track {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: rgba(245, 158, 11, 0.3);
}

.card-progress-fill {
  height: 100%;
  background: var(--color-accent);
  transition: width 0.05s linear;
}

/* ==========================================================================
   STICKY BOTTOM PLAYER (GLOBAL 7-SECOND BAR)
   ========================================================================== */
.sticky-player-bar {
  position: fixed;
  bottom: calc(58px + env(safe-area-inset-bottom, 0px));
  left: 0;
  right: 0;
  background-color: var(--color-text-primary);
  color: #FFFFFF;
  z-index: 999;
  box-shadow: var(--shadow-sticky);
  border-top: 2px solid var(--color-accent);
  animation: slideUp 0.2s ease-out;
}

@keyframes slideUp {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
}

.sticky-progress-track {
  width: 100%;
  height: 3px;
  background: rgba(255, 255, 255, 0.2);
}

.sticky-progress-fill {
  height: 100%;
  background: var(--color-accent);
  transition: width 0.05s linear;
}

.sticky-player-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 16px;
}

.sticky-player-left {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 0;
}

.playing-icon {
  color: var(--color-accent);
  font-size: 24px;
  animation: pulseIcon 1s infinite alternate;
}

@keyframes pulseIcon {
  from { opacity: 0.6; }
  to { opacity: 1; }
}

.sticky-player-meta {
  min-width: 0;
}

.sticky-player-name {
  font-size: 15px;
  font-weight: 700;
  color: #FFFFFF;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sticky-player-sub {
  font-size: 12px;
  color: var(--color-text-light);
}

.sticky-player-right {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-shrink: 0;
}

.sticky-time-pill {
  font-family: 'JetBrains Mono', monospace;
  font-size: 13px;
  font-weight: 800;
  color: var(--color-accent);
  background: rgba(245, 158, 11, 0.15);
  padding: 4px 8px;
  border-radius: var(--radius-sm);
  border: 1px solid rgba(245, 158, 11, 0.3);
}

.sticky-stop-btn {
  background: var(--color-danger);
  color: #FFFFFF;
  border: none;
  border-radius: var(--radius-md);
  padding: 6px 14px;
  font-size: 13px;
  font-weight: 800;
  display: flex;
  align-items: center;
  gap: 4px;
  cursor: pointer;
}

/* ==========================================================================
   ONBOARDING & EMPTY STATES
   ========================================================================== */
.onboarding-container {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: calc(100vh - 150px);
  padding: 16px;
}

.onboarding-card {
  background: #FFFFFF;
  border: 1.5px solid var(--color-border);
  border-radius: var(--radius-lg);
  padding: 32px 24px;
  max-width: 440px;
  width: 100%;
  text-align: center;
  box-shadow: var(--shadow-subtle);
}

.onboarding-icon-box {
  width: 64px;
  height: 64px;
  background: var(--color-brand-light);
  color: var(--color-brand);
  border-radius: var(--radius-full);
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 16px auto;
}

.onboarding-icon {
  font-size: 32px;
}

.onboarding-title {
  font-size: 24px;
  font-weight: 900;
  color: var(--color-text-primary);
  margin: 0 0 8px 0;
}

.onboarding-subtitle {
  font-size: 15px;
  color: var(--color-text-muted);
  line-height: 1.5;
  margin: 0 0 24px 0;
}

.team-input-group {
  display: flex;
  align-items: center;
  background: var(--color-surface);
  border: 2px solid var(--color-brand);
  border-radius: var(--radius-md);
  padding: 4px 12px;
  margin-bottom: 16px;
}

.team-input-prefix {
  font-family: 'JetBrains Mono', monospace;
  font-size: 24px;
  font-weight: 900;
  color: var(--color-brand);
  margin-right: 6px;
}

.team-id-input {
  flex: 1;
  border: none;
  outline: none;
  background: transparent;
  font-family: 'JetBrains Mono', monospace;
  font-size: 24px;
  font-weight: 900;
  color: var(--color-brand);
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.onboarding-info {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  color: var(--color-text-muted);
  font-size: 13px;
  margin-top: 16px;
}

.onboarding-info i {
  font-size: 16px;
}

.empty-state-box {
  background: #FFFFFF;
  border: 1.5px dashed var(--color-border-strong);
  border-radius: var(--radius-lg);
  padding: 40px 20px;
  text-align: center;
  margin-top: 20px;
}

.empty-icon {
  font-size: 48px;
  color: var(--color-text-light);
  margin-bottom: 12px;
}

.empty-state-box h3 {
  font-size: 18px;
  font-weight: 700;
  margin: 0 0 6px 0;
  color: var(--color-text-primary);
}

.empty-state-box p {
  font-size: 14px;
  color: var(--color-text-muted);
  margin: 0 0 16px 0;
}

/* ==========================================================================
   SETTINGS & MANAGEMENT SECTIONS
   ========================================================================== */
.settings-container {
  display: flex;
  flex-direction: column;
  gap: 16px;
  max-width: 640px;
  margin: 0 auto;
}

.section-card {
  background: #FFFFFF;
  border: 1.5px solid var(--color-border);
  border-radius: var(--radius-lg);
  padding: 20px;
  box-shadow: var(--shadow-subtle);
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 14px;
}

.section-title-wrap {
  display: flex;
  align-items: center;
  gap: 8px;
}

.section-icon {
  color: var(--color-brand);
  font-size: 22px;
}

.section-title {
  margin: 0;
  font-size: 17px;
  font-weight: 800;
  color: var(--color-text-primary);
}

.section-desc {
  font-size: 14px;
  color: var(--color-text-muted);
  margin: 0 0 16px 0;
  line-height: 1.4;
}

.teams-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.team-list-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 12px;
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
}

.team-item-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.team-item-details {
  display: flex;
  flex-direction: column;
}

.team-item-name {
  font-size: 15px;
  font-weight: 700;
  color: var(--color-text-primary);
}

.team-item-meta {
  font-size: 12px;
  color: var(--color-text-muted);
}

.btn-icon-danger {
  background: transparent;
  border: none;
  color: var(--color-danger);
  cursor: pointer;
  padding: 6px;
  border-radius: var(--radius-sm);
  display: flex;
  align-items: center;
}

.btn-icon-danger:active {
  background: var(--color-danger-light);
}

.add-team-inline-box {
  margin-top: 14px;
  padding: 16px;
  background: var(--color-bg);
  border: 1.5px dashed var(--color-border-strong);
  border-radius: var(--radius-md);
}

.add-team-inline-label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  font-weight: 700;
  color: var(--color-brand);
  margin-bottom: 12px;
}

.add-team-inline-label i {
  font-size: 18px;
}

.limit-reached-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  background: var(--color-brand-light);
  color: var(--color-brand);
  font-size: 13px;
  font-weight: 600;
  padding: 8px 12px;
  border-radius: var(--radius-md);
  margin-top: 10px;
}

.updates-available-box {
  background: var(--color-accent-light);
  border: 1.5px solid var(--color-accent);
  border-radius: var(--radius-md);
  padding: 14px;
  margin-bottom: 14px;
}

.updates-header {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #92400E;
  font-size: 14px;
  margin-bottom: 10px;
}

.alert-icon {
  color: var(--color-accent);
  font-size: 20px;
}

.updates-list {
  display: flex;
  flex-direction: column;
  gap: 4px;
  max-height: 120px;
  overflow-y: auto;
  margin-bottom: 12px;
  background: #FFFFFF;
  padding: 8px;
  border-radius: var(--radius-sm);
  border: 1px solid rgba(245, 158, 11, 0.3);
}

.update-item {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
}

.update-name {
  font-weight: 600;
  color: var(--color-text-primary);
}

.update-team {
  color: var(--color-text-muted);
  font-size: 12px;
}

.all-synced-box {
  display: flex;
  align-items: center;
  gap: 8px;
  background: var(--color-success-light);
  color: #166534;
  font-weight: 600;
  font-size: 14px;
  padding: 12px;
  border-radius: var(--radius-md);
  margin-bottom: 14px;
}

.success-icon {
  color: var(--color-success);
  font-size: 20px;
}

.sync-status-box {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px;
  background: var(--color-surface);
  border-radius: var(--radius-md);
  margin-bottom: 14px;
  font-size: 14px;
  color: var(--color-text-muted);
}

.settings-field {
  margin-bottom: 20px;
}

.field-label {
  display: block;
  font-size: 14px;
  font-weight: 700;
  color: var(--color-text-primary);
  margin-bottom: 6px;
}

.text-input {
  width: 100%;
  padding: 10px 12px;
  background: var(--color-surface);
  border: 1.5px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: 14px;
  box-sizing: border-box;
  font-family: inherit;
  outline: none;
}

.text-input:focus {
  border-color: var(--color-brand);
}

.field-hint {
  display: block;
  font-size: 12px;
  color: var(--color-text-muted);
  margin-top: 4px;
}

.danger-zone {
  border-top: 1.5px solid var(--color-border);
  padding-top: 16px;
  margin-top: 16px;
}

.danger-title {
  margin: 0 0 6px 0;
  font-size: 15px;
  font-weight: 800;
  color: var(--color-danger);
}

.danger-desc {
  font-size: 13px;
  color: var(--color-text-muted);
  margin: 0 0 12px 0;
}

/* ==========================================================================
   BUTTONS & ALERTS
   ========================================================================== */
.btn-primary {
  background-color: var(--color-brand);
  color: #FFFFFF;
  border: none;
  border-radius: var(--radius-md);
  font-weight: 700;
  font-size: 15px;
  padding: 10px 16px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.btn-primary:active {
  background-color: #1E3A8A;
}

.btn-secondary {
  background-color: #FFFFFF;
  color: var(--color-text-primary);
  border: 1.5px solid var(--color-border);
  border-radius: var(--radius-md);
  font-weight: 700;
  font-size: 14px;
  padding: 10px 16px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.btn-success {
  background-color: var(--color-success);
  color: #FFFFFF;
  border: none;
  border-radius: var(--radius-md);
  font-weight: 700;
  font-size: 15px;
  padding: 10px 16px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.btn-danger {
  background-color: var(--color-danger);
  color: #FFFFFF;
  border: none;
  border-radius: var(--radius-md);
  font-weight: 700;
  font-size: 14px;
  padding: 10px 16px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.btn-large {
  padding: 14px 20px;
  font-size: 16px;
}

.btn-sm {
  padding: 6px 12px;
  font-size: 13px;
}

.btn-block {
  width: 100%;
}

.alert-box {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  border-radius: var(--radius-md);
  font-size: 13px;
  margin-bottom: 12px;
  text-align: left;
}

.alert-danger {
  background-color: var(--color-danger-light);
  color: var(--color-danger);
  border: 1px solid #FECACA;
}

.spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: #FFFFFF;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.btn-loading {
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

/* ==========================================================================
   MODAL BACKDROP & DIALOG
   ========================================================================== */
.custom-modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(4px);
  z-index: 10000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
}

.custom-modal-card {
  background: #FFFFFF;
  border-radius: var(--radius-lg);
  max-width: 400px;
  width: 100%;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.2);
  overflow: hidden;
  animation: modalPop 0.2s ease-out;
}

@keyframes modalPop {
  from { transform: scale(0.95); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1.5px solid var(--color-border);
}

.modal-title {
  margin: 0;
  font-size: 18px;
  font-weight: 800;
  color: var(--color-text-primary);
}

.modal-body {
  padding: 20px;
}

.modal-instruction {
  font-size: 14px;
  color: var(--color-text-muted);
  margin: 0 0 16px 0;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 14px 20px;
  background: var(--color-surface);
  border-top: 1.5px solid var(--color-border);
}
</style>
