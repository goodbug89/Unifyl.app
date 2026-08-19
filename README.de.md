<p align="center">
  <img src="docs/screenshots/icon.png" alt="Unifyl" width="128" height="128">
</p>

<h1 align="center">Unifyl</h1>

<p align="center">
  <strong>Der KI-gestützte Zweipanel-Dateimanager für macOS.</strong><br>
  Die Tiefe von Total Commander. Die Eleganz von ForkLift. Eingebaute Intelligenz.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-14%2B-000?logo=apple&logoColor=white" alt="macOS 14+">
  <img src="https://img.shields.io/badge/Swift-6.0-F05138?logo=swift&logoColor=white" alt="Swift 6">
  <img src="https://img.shields.io/badge/Apple%20Silicon-Optimized-8E8E93" alt="Apple Silicon">
</p>

<p align="center">
  <a href="https://unifyl.app/de/">Website</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">Dokumentation</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/releases/latest">Download</a>
</p>

<p align="center">
  <strong>Sprache:</strong>
  <a href="README.md">English</a> &middot;
  <a href="README.ja.md">日本語</a> &middot;
  <a href="README.zh-Hans.md">简体中文</a> &middot;
  <strong>Deutsch</strong> &middot;
  <a href="README.es.md">Español</a> &middot;
  <a href="README.fr.md">Français</a>
</p>

---

![Unifyl Screenshot](docs/screenshots/01-dualpane.png)

## Überblick

Unifyl ist der Zweipanel-Dateimanager für macOS, der professionelle Power-Tools mit lokaler KI-Intelligenz verbindet. Ihre Dateien verlassen Ihren Mac nie.

- **Zweipanel** — Einzel-, Zwei-, Dreipanel- und freie Split-Layouts, Tabs, Arbeitsbereiche, Inline-Terminal in jedem Panel
- **KI-gestützt** — Semantische Suche, KI-Umbenennung, intelligente Tags, Duplikat-Erkennung — alles lokal auf Apple Neural Engine
- **Remote & Cloud** — FTP, SFTP, WebDAV, SMB, S3, Google Drive, Dropbox, OneDrive, Docker, Kubernetes
- **Über 50 professionelle Werkzeuge** — Mehrfach-Umbenennen, Text/Binär/Bildvergleich, Verzeichnissynchronisation, 3-Wege-Merge, EXIF-Stapeleditor
- **Archive als virtuelle Ordner** — ZIP, 7z, TAR, GZ, BZ2, XZ, ZSTD, RAR (inkl. automatischer Kodierungserkennung für CJK-Legacy-ZIPs)
- **Tastatur zuerst** — Befehlspalette, Vim-artige Navigation, über 120 frei belegbare Kurzbefehle
- **15 Sprachen** — Englisch, Koreanisch, Japanisch, Chinesisch (vereinfacht & traditionell), Deutsch, Französisch, Spanisch, Portugiesisch, Italienisch, Russisch, Türkisch, Arabisch, Thailändisch, Vietnamesisch
- **CJK-Verstärkung** (1.3.0): `.alz`-Archive entpacken (gebündeltes `unalz`), `.hwpx`-Inline-Vorschau, mehrsprachige Dateinamen-Suche (Pinyin / Romaji / Hangul-Hanja)
- **CJK-Verstärkung erweitert** (1.3.1): Auto-Lesezeichen für Messenger-/Cloud-Ordner (KakaoTalk · LINE · WeChat · iCloud Drive — 12 Pfade), Voll-/Halbbreite-Suchfaltung, EUC-KR/Shift-JIS/GBK → UTF-8 Kodierungskonvertierung (mit Rückgängig), Heuristische Erkennung koreanischer Behördenformulare

---

## Installation

Holen Sie die neueste Version unter [unifyl.app/download](https://github.com/goodbug89/Unifyl.app/releases/latest).

Auch im [Mac App Store](https://apps.apple.com/app/unifyl/id6773977314?mt=12) erhältlich (Sandbox-Edition — Funktionsunterschiede siehe FAQ).

1. Öffnen Sie die `.dmg`-Datei
2. Ziehen Sie Unifyl in den Ordner „Programme"
3. Starten Sie die App und gewähren Sie den Dateizugriff, wenn Sie dazu aufgefordert werden

---

## Preise

| | **Free** | **Pro — $24.99** |
|---|---|---|
| Panel-Layouts | Zwei/drei/frei geteilt (alle) | Einzel, Zwei, Drei, freier Split |
| Suche | Basis + Spotlight | Voller Regex + Inhaltssuche |
| KI-Werkzeuge | — | Alle KI-Funktionen |
| Remote-Verbindungen | FTP, SFTP | Alle Protokolle + Cloud-Speicher |
| Archive | ZIP | ZIP, 7z, TAR, GZ, BZ2, XZ, ZSTD, RAR, **ALZ** |

> **Hinweis zur Edition.** Die Mac-App-Store-Version läuft in der Sandbox und enthält kein Git-Panel, kein Docker/Kubernetes, keine SSH-Tunnel, kein Inline-Terminal, keine Port- und Prozess-Viewer und keine ffmpeg-basierte Medienkonvertierung — macOS erlaubt das dort nicht. Alles andere ist identisch, und Pro kostet auf beiden Wegen dasselbe einmalig. Wählen Sie den Direkt-Download, wenn Sie eines davon brauchen.

Einmaliger Kauf, kein Abo. Kostenloser Download im Mac App Store (Pro per In-App-Kauf, an Ihre Apple-ID gebunden mit Familienfreigabe) oder als Direkt-Download (Pro über LemonSqueezy, nutzbar auf bis zu 3 Geräten). 14 Tage kostenlose Pro-Testphase beim ersten Start.

---

## Wichtige Tastaturkurzbefehle

| Kurzbefehl | Aktion |
|---|---|
| `⌘ + ⇧ + P` | Befehlspalette |
| `Tab` | Aktives Panel wechseln |
| `Return` | Datei öffnen / Ordner betreten |
| `⇧ + Return` | Umbenennen |
| `Space` | Übersicht (Quick Look) |
| `⌘ + Delete` | In den Papierkorb |
| `⌃ + G` | Miniaturraster |
| `⌘ + T` | Neuer Tab |
| `⌘ + 2` | Zweipanel |
| `⌘ + K` | Terminal |

Alle über 120 Kurzbefehle sind unter „Einstellungen > Tastaturkurzbefehle" anpassbar.

---

## Aus Quellcode bauen

```bash
git clone https://github.com/goodbug89/Unifyl.app.git
cd unifyl
make doctor    # Toolchain prüfen
make gen       # Xcode-Projekt erzeugen
make build     # Alle Pakete sequenziell bauen
```

Vollständige Funktionsliste, Beitragsrichtlinien und Plug-in-Entwicklung im [englischen README](README.md).

---

## Lizenz

Proprietär. Copyright 2024–2026 Unifyl. Alle Rechte vorbehalten.

<p align="center">
  <a href="https://unifyl.app/de/">Website</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">Dokumentation</a>
</p>
