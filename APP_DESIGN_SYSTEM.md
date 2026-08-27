# 7secs – Mobile App Design System & Style Guide

> **Zweck dieses Dokuments:**  
> Dieser Leitfaden definiert alle Design-Tokens, UI-Komponenten, Typografie, Farben und UX-Interaktionsmuster für die native **7secs Mobile App** (iOS / Android / Flutter / React Native). Ziel ist ein konsistentes, sportliches und blitzschnell bedienbares Erlebnis auf dem Spielfeld und am Kampfgericht.

---

## 1. Design-Philosophie & UX-Prinzipien

1. **Sporty, High-Contrast & Clean:**
   - Keine überladenen Glassmorphism-Effekte oder unruhigen Farbverläufe.
   - Klare, taktile Oberflächen mit scharfen 1.5pt Border-Linien und prägnantem Kontrast.
   - Perfekte Ablesbarkeit auch bei grellem Hallenlicht oder aus Distanz.

2. **One-Tap Soundboard (Null Reibung):**
   - Im Spielbetrieb zählt jede Sekunde: **1 Tap auf den Spielernamen $\rightarrow$ Song startet unverzüglich.**
   - Große, fehlerverzeihende Touch-Targets ($\ge 56\text{pt}$ Höhe).
   - Sofortiges visuelles und haptisches Feedback bei Berührung.

3. **7-Sekunden-Fokus:**
   - Der Countdown von 7 Sekunden wird prominent visualisiert (Pulse-Effekt, animierter Progress-Ring oder Waveform-Indikator).
   - Klare Farbkodierung: **Deep Blue / Surface** für Bereit, **Amber Gold** für Jubel/Laufender Sound, **Success Green** für aktive Spieler.

---

## 2. Farb-Palette & Semantic Tokens

### 2.1 Primäre Farb-Tokens (Light Mode / Hallenmodus)

```
┌────────────────────────────────────────────────────────────────────────┐
│  Brand Deep Blue   │  Accent Amber (Goal!) │  Success Green (Active)   │
│     #1E40AF        │       #F59E0B         │        #16A34A            │
└────────────────────────────────────────────────────────────────────────┘
```

| Token-Name | Hex-Wert | RGB | Verwendung |
|---|---|---|---|
| `color-brand` | `#1E40AF` | `rgb(30, 64, 175)` | Primäre Akzentfarbe, App-Bar, Team-ID-Border, Buttons |
| `color-brand-light` | `#DBEAFE` | `rgb(219, 234, 254)` | Team-ID Badge Hintergrund, Hover/Pressed-Flächen |
| `color-brand-mid` | `#3B82F6` | `rgb(59, 130, 246)` | Icons, Infoboxen, Ladeindikatoren |
| `color-accent` | `#F59E0B` | `rgb(245, 158, 11)` | **Goal / Active Sound:** Laufender Song, Jubel-Highlight |
| `color-accent-light` | `#FEF3C7` | `rgb(254, 243, 199)` | Hintergrund bei laufendem Sound / Warn-Badges |
| `color-success` | `#16A34A` | `rgb(22, 163, 74)` | Status "Aktiv im Kader", Play-Button, Upload bereit |
| `color-success-light` | `#DCFCE7` | `rgb(220, 252, 231)` | Status-Badge Hintergrund, Toggle aktiv |
| `color-danger` | `#DC2626` | `rgb(220, 38, 38)` | Stop-Aktion, Löschen, Fehler-Banners |
| `color-danger-light` | `#FEF2F2` | `rgb(254, 242, 242)` | Fehler-Boxen |

### 2.2 Oberflächen & Text (Neutrals)

| Token-Name | Hex-Wert | RGB | Verwendung |
|---|---|---|---|
| `color-bg` | `#FFFFFF` | `rgb(255, 255, 255)` | Haupt-App-Hintergrund (Canvas) |
| `color-surface` | `#F8FAFC` | `rgb(248, 250, 252)` | Soundboard-Kacheln, Card-Hintergrund |
| `color-border` | `#E2E8F0` | `rgb(226, 232, 240)` | Standard-Border für Cards, Trennlinien |
| `color-border-strong`| `#CBD5E1` | `rgb(203, 213, 225)` | Fokus-Borders, Inaktiv-Dashed-Borders |
| `color-text-primary` | `#0F172A` | `rgb(15, 23, 42)` | Spielernamen, Überschriften (Slate 900) |
| `color-text-muted` | `#64748B` | `rgb(100, 116, 139)` | Untertitel, Beschreibungen, Zähler |
| `color-text-light` | `#94A3B8` | `rgb(148, 163, 184)` | Deaktivierte Texte, Placeholders |

### 2.3 Dark Mode Palette (Optional für Abendspiele)

| Token | Light Value | Dark Equivalent |
|---|---|---|
| `color-bg` | `#FFFFFF` | `#0B0F19` (Fast Schwarz / Tiefes Slate) |
| `color-surface` | `#F8FAFC` | `#151E2E` (Dunkles Navy) |
| `color-border` | `#E2E8F0` | `#26354A` |
| `color-text-primary` | `#0F172A` | `#F8FAFC` |
| `color-text-muted` | `#64748B` | `#94A3B8` |
| `color-brand-light` | `#DBEAFE` | `#1E293B` |

---

## 3. Typografie-System

- **iOS Standardschrift:** San Francisco (`SF Pro Display` / `SF Pro Text`)
- **Android Standardschrift:** `Roboto` oder `Inter`
- **Monospace für IDs/Dateinamen:** `SF Mono` / `Roboto Mono`

| Style | Size | Weight | Tracking | Line-Height | Verwendung |
|---|---|---|---|---|---|
| **Display (Team ID)** | 36pt | 900 (Black) | +0.12em | 1.0 | Große Team-Kennung (z. B. `#H4R`) |
| **Headline Large (H1)** | 28pt | 800 (Bold) | -0.02em | 1.15 | Screen-Title (z. B. Mannschaftsname) |
| **Headline Medium (H2)**| 20pt | 700 (Bold) | -0.01em | 1.25 | Sektions-Titel, Modals |
| **Player Name** | 18pt | 700 (Bold) | normal | 1.25 | Name auf Soundboard-Taste |
| **Body Base** | 16pt | 600 / 400 | normal | 1.5 | Standard-Texte, Button-Labels |
| **Subtext / Meta** | 13pt | 500 (Medium) | normal | 1.3 | Dateiname, Subtitles |
| **Badge / Micro** | 11pt | 700 (Bold) | +0.03em | 1.0 | Status-Pills (`Aktiv`, `v2`, `7s`) |

---

## 4. Spacing & Radien (Grid-System)

### 4.1 Abstände (4pt/8pt Grid)
- `spacing-xs`: `4pt`
- `spacing-sm`: `8pt`
- `spacing-md`: `16pt`
- `spacing-lg`: `24pt`
- `spacing-xl`: `32pt`
- `spacing-2xl`: `48pt`

### 4.2 Corner Radii (Abrundungen)
- `radius-sm`: `6pt` (Kompakte Badges, Mini-Buttons)
- `radius-md`: `10pt` (Eingabefelder, Standard-Buttons)
- `radius-lg`: `16pt` (Soundboard-Kacheln, Player-Cards, Bottom-Sheets)
- `radius-full`: `9999pt` (Pill-Buttons, Status-Pills, Countdown-Rings)

### 4.3 Borders & Elevation
- **Border-Stärke:** Konsistent `1.5pt solid var(--color-border)` für alle Cards und Kacheln.
- **Schatten:**
  - `shadow-subtle`: `0 2px 4px rgba(0, 0, 0, 0.05)` (Idle Kacheln)
  - `shadow-active`: `0 8px 16px -2px rgba(245, 158, 11, 0.25)` (Playing Glow)

---

## 5. UI-Komponenten Spezifikation

### 5.1 Team-ID Badge (`#H4R`)

Die Mannschafts-Kennung ist das zentrale Erkennungsmerkmal.

```
┌──────────────┐
│    #H4R      │
└──────────────┘
```
- **Hintergrund:** `color-brand-light` (`#DBEAFE`)
- **Text & Rahmen:** `color-brand` (`#1E40AF`), 2pt Randstärke
- **Typografie:** Display 24pt–32pt, Bold/Black, Buchstabenabstand gesperrt (`+0.12em`), Monospace/Tabular Numbers
- **Radius:** `radius-md` (`10pt`)

---

### 5.2 Soundboard-Player-Kachel (Hero-Komponente)

Das Soundboard wird als 3-spaltiges Grid (Smartphone) oder 4-5-spaltiges Grid (Tablet/Desktop) dargestellt. Alle Kacheln sind als kompakte, direkte Taster (Buzzer-Pads) konzipiert, sodass ein Spieltagskader von ca. 15–20 Spielern ohne Scrollen vollständig auf das Display passt.

#### Zustände der Kachel:

```
[ IDLE STATE ]                         [ PLAYING STATE (7s) ]
┌──────────────────────────────┐       ┌──────────────────────────────┐
│ [ T ]  Tom M.                │  Tap  │ [ ■ ]  Tom M.          4.2s  │
│                              │ ────> │ ━━━━━━━━━━━━━━━━━━━━━━░░░░░  │
└──────────────────────────────┘       └──────────────────────────────┘
```

1. **State: Idle (Bereit)**
   - Höhe: ca. 48px, kompakt & taktil
   - Hintergrund: `#F8FAFC`
   - Border: `1.5pt solid #E2E8F0`
   - Avatar: Runde/Quadratische Initiale (z. B. `T` für Tom) in `#DBEAFE` mit `#1E40AF` Text (28px)
   - Name: `13–14pt Bold` in `#0F172A`
   - Redundante Texte wie Version oder Songlänge entfallen für maximale Übersicht.

2. **State: Playing (Sound läuft – 7s Countdown)**
   - Hintergrund: `#FEF3C7` (Amber Light) mit `#F59E0B` Border
   - Avatar-Icon wechselt zu rotem Stop-Symbol `■`
   - Badge: Zeigt verbleibende Restzeit `7.0s ➔ 0.0s`
   - Progress-Leiste: 3px hoher animierter Fortschrittsbalken an der Unterkante der Kachel

3. **State: Inaktiv / Ausgeblendet**
   - Wird im **Live-Soundboard komplett ausgeblendet** (damit keine falschen Klicks im Spiel passieren).
   - In der Verwaltungsansicht: Gestrichelter Rahmen (`dashed #CBD5E1`), 50% Deckkraft.

---

### 5.3 Sticky Bottom-Player (Globale Sound-Leiste)

Wenn ein Sound abgespielt wird, erscheint am unteren Bildschirmrand eine persistente Leiste:

```
┌─────────────────────────────────────────────────────────────┐
│ 🎵 Tom M. · Torjubel läuft...       [ 4.8s ]   [ ■ STOPP ]  │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
└─────────────────────────────────────────────────────────────┘
```
- **Hintergrund:** `#0F172A` (Dunkles Slate) mit weißer Schrift
- **Progress-Bar:** `#F59E0B` (Amber), nimmt über 7,0s linear auf 0% ab
- **Schnell-Stopp:** Großer roter `■ STOPP`-Button (min. 48pt Touch-Target)

---

### 5.4 Schnelle Such- & Filterleiste

- Oben auf dem Soundboard fixiert.
- Live-Filterung bei Eingabe des Spielernamens.
- Filter-Chips: `Alle`, `Kader (Aktiv)`, `Zuletzt gespielt`.

---

## 6. Haptik & Audio UX-Richtlinien

1. **Haptisches Feedback:**
   - **Sound Start:** `UIImpactFeedbackGenerator(.medium)` (iOS) bzw. `HapticFeedbackConstants.VIRTUAL_KEY` (Android).
   - **Sound Auto-Stop:** Sanfter `UIImpactFeedbackGenerator(.light)` Impuls nach Ablauf der 7 Sekunden.
   - **Manueller Abbruch:** Sofortiger Feedback-Klick.

2. **Audio-Session Konfiguration:**
   - Kategorie: `AVAudioSessionCategoryPlayback` (spielt auch bei aktivierter Stummschaltung/Lautlos-Schalter ab!).
   - Audio Focus: Hintergrundmusik (z. B. Hallen-Playlist) unterbrechen oder ducking anwenden.

3. **Exklusivität:**
   - Immer nur **1 Sound zeitgleich**. Neuer Klick $\rightarrow$ alter Sound stoppt sofort ohne Überlappung.

---

## 7. Native Code-Snippets für App-Entwickler

### 7.1 SwiftUI (iOS) Theme & Components

```swift
import SwiftUI

// MARK: - Color Extension
extension Color {
    static let appBrand = Color(red: 0.12, green: 0.25, blue: 0.69)       // #1E40AF
    static let appBrandLight = Color(red: 0.86, green: 0.92, blue: 0.99)  // #DBEAFE
    static let appAccent = Color(red: 0.96, green: 0.62, blue: 0.04)      // #F59E0B
    static let appAccentLight = Color(red: 1.00, green: 0.95, blue: 0.78) // #FEF3C7
    static let appSuccess = Color(red: 0.09, green: 0.64, blue: 0.29)     // #16A34A
    static let appSurface = Color(red: 0.97, green: 0.98, blue: 0.99)     // #F8FAFC
    static let appBorder = Color(red: 0.89, green: 0.91, blue: 0.94)      // #E2E8F0
    static let appText = Color(red: 0.06, green: 0.09, blue: 0.16)        // #0F172A
    static let appTextMuted = Color(red: 0.39, green: 0.45, blue: 0.55)   // #64748B
}

// MARK: - Team ID Badge
struct TeamIDBadge: View {
    let id: String
    
    var body: some View {
        Text(id)
            .font(.system(size: 26, weight: .black, design: .monospaced))
            .foregroundColor(.appBrand)
            .padding(.horizontal, 16)
            .padding(.vertical, 8)
            .background(Color.appBrandLight)
            .clipShape(RoundedRectangle(cornerRadius: 10))
            .overlay(
                RoundedRectangle(cornerRadius: 10)
                    .stroke(Color.appBrand, lineWidth: 2)
            )
    }
}

// MARK: - Soundboard Player Tile
struct SoundboardTile: View {
    let name: String
    let version: Int
    let isPlaying: Bool
    let onToggle: () -> Void
    
    var body: some View {
        Button(action: onToggle) {
            VStack(alignment: .leading, spacing: 12) {
                HStack(spacing: 10) {
                    // Initial Avatar
                    Text(String(name.prefix(1)))
                        .font(.system(size: 18, weight: .heavy))
                        .foregroundColor(isPlaying ? .appText : .appBrand)
                        .frame(width: 42, height: 42)
                        .background(isPlaying ? Color.white : Color.appBrandLight)
                        .clipShape(RoundedRectangle(cornerRadius: 8))
                    
                    VStack(alignment: .leading, spacing: 2) {
                        Text(name)
                            .font(.system(size: 18, weight: .bold))
                            .foregroundColor(.appText)
                            .lineLimit(1)
                        
                        Text("v\(version) · 7s Torsong")
                            .font(.system(size: 12, weight: .medium))
                            .foregroundColor(.appTextMuted)
                    }
                    Spacer()
                }
                
                // Bottom Action Bar
                HStack {
                    Image(systemName: isPlaying ? "stop.fill" : "play.fill")
                    Text(isPlaying ? "STOPP" : "Sound")
                        .font(.system(size: 15, weight: .bold))
                }
                .frame(maxWidth: .infinity)
                .frame(height: 40)
                .background(isPlaying ? Color.red : Color.appSuccess)
                .foregroundColor(.white)
                .clipShape(RoundedRectangle(cornerRadius: 8))
            }
            .padding(16)
            .background(isPlaying ? Color.appAccentLight : Color.white)
            .clipShape(RoundedRectangle(cornerRadius: 16))
            .overlay(
                RoundedRectangle(cornerRadius: 16)
                    .stroke(isPlaying ? Color.appAccent : Color.appBorder, lineWidth: isPlaying ? 2.5 : 1.5)
            )
            .shadow(color: isPlaying ? Color.appAccent.opacity(0.3) : Color.black.opacity(0.04),
                    radius: isPlaying ? 8 : 4, y: 2)
        }
        .buttonStyle(.plain)
    }
}
```

---

### 7.2 Jetpack Compose (Android) Theme & Components

```kotlin
package de.tura.sevensecs.ui.theme

import androidx.compose.foundation.background
import androidx.compose.foundation.border
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

// MARK: - Color Definitions
val AppBrand = Color(0xFF1E40AF)
val AppBrandLight = Color(0xFFDBEAFE)
val AppAccent = Color(0xFFF59E0B)
val AppAccentLight = Color(0xFFFEF3C7)
val AppSuccess = Color(0xFF16A34A)
val AppBorder = Color(0xFFE2E8F0)
val AppText = Color(0xFF0F172A)
val AppTextMuted = Color(0xFF64748B)

// MARK: - Team ID Component
@Composable
fun TeamIdBadge(id: String) {
    Box(
        modifier = Modifier
            .background(AppBrandLight, RoundedCornerShape(10.dp))
            .border(2.dp, AppBrand, RoundedCornerShape(10.dp))
            .padding(horizontal = 16.dp, vertical = 8.dp),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = id,
            color = AppBrand,
            fontSize = 26.sp,
            fontWeight = FontWeight.Black,
            fontFamily = FontFamily.Monospace
        )
    }
}

// MARK: - Soundboard Player Tile
@Composable
fun SoundboardTile(
    name: String,
    version: Int,
    isPlaying: Boolean,
    onToggle: () -> Unit
) {
    Card(
        shape = RoundedCornerShape(16.dp),
        colors = CardDefaults.cardColors(
            containerColor = if (isPlaying) AppAccentLight else Color.White
        ),
        modifier = Modifier
            .fillMaxWidth()
            .border(
                width = if (isPlaying) 2.5.dp else 1.5.dp,
                color = if (isPlaying) AppAccent else AppBorder,
                shape = RoundedCornerShape(16.dp)
            )
            .clickable { onToggle() }
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Row(verticalAlignment = Alignment.CenterVertically) {
                Box(
                    modifier = Modifier
                        .size(42.dp)
                        .background(if (isPlaying) Color.White else AppBrandLight, RoundedCornerShape(8.dp)),
                    contentAlignment = Alignment.Center
                ) {
                    Text(
                        text = name.take(1),
                        fontSize = 18.sp,
                        fontWeight = FontWeight.ExtraBold,
                        color = if (isPlaying) AppText else AppBrand
                    )
                }
                Spacer(modifier = Modifier.width(12.dp))
                Column {
                    Text(
                        text = name,
                        fontSize = 18.sp,
                        fontWeight = FontWeight.Bold,
                        color = AppText
                    )
                    Text(
                        text = "v$version · 7s Torsong",
                        fontSize = 12.sp,
                        color = AppTextMuted
                    )
                }
            }

            Spacer(modifier = Modifier.height(14.dp))

            Button(
                onClick = onToggle,
                shape = RoundedCornerShape(8.dp),
                colors = ButtonDefaults.buttonColors(
                    containerColor = if (isPlaying) Color(0xFFDC2626) else AppSuccess
                ),
                modifier = Modifier.fillMaxWidth().height(40.dp)
            ) {
                Text(
                    text = if (isPlaying) "■ STOPP" else "▶ Sound",
                    fontWeight = FontWeight.Bold,
                    fontSize = 15.sp,
                    color = Color.White
                )
            }
        }
    }
}
```
