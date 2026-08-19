<p align="center">
  <img src="docs/screenshots/icon.png" alt="Unifyl" width="128" height="128">
</p>

<h1 align="center">Unifyl</h1>

<p align="center">
  <strong>Le gestionnaire de fichiers à double panneau propulsé par l'IA pour macOS.</strong><br>
  La profondeur de Total Commander. La finition de ForkLift. L'intelligence intégrée.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-14%2B-000?logo=apple&logoColor=white" alt="macOS 14+">
  <img src="https://img.shields.io/badge/Swift-6.0-F05138?logo=swift&logoColor=white" alt="Swift 6">
  <img src="https://img.shields.io/badge/Apple%20Silicon-Optimized-8E8E93" alt="Apple Silicon">
</p>

<p align="center">
  <a href="https://unifyl.app/fr/">Site web</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">Documentation</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/releases/latest">Télécharger</a>
</p>

<p align="center">
  <strong>Langue :</strong>
  <a href="README.md">English</a> &middot;
  <a href="README.ja.md">日本語</a> &middot;
  <a href="README.zh-Hans.md">简体中文</a> &middot;
  <a href="README.de.md">Deutsch</a> &middot;
  <a href="README.es.md">Español</a> &middot;
  <strong>Français</strong>
</p>

---

![Capture Unifyl](docs/screenshots/01-dualpane.png)

## Aperçu

Unifyl est le gestionnaire de fichiers à double panneau pour macOS qui combine des outils professionnels et une intelligence artificielle locale. Vos fichiers ne quittent jamais votre Mac.

- **Double panneau** — Mises en page simple, double, triple et libre, onglets, espaces de travail, terminal intégré dans chaque panneau
- **Propulsé par l'IA** — Recherche sémantique, renommage par IA, étiquetage intelligent, détection de doublons — le tout local sur Apple Neural Engine
- **Distant et cloud** — FTP, SFTP, WebDAV, SMB, S3, Google Drive, Dropbox, OneDrive, Docker, Kubernetes
- **Plus de 50 outils professionnels** — Renommer par lot, comparaison texte/binaire/image, synchronisation de dossiers, fusion à 3 voies, édition EXIF par lot
- **Archives en dossiers virtuels** — ZIP, 7z, TAR, GZ, BZ2, XZ, ZSTD, RAR (avec détection automatique de l'encodage des ZIP CJK hérités)
- **Clavier d'abord** — Palette de commandes, navigation à la Vim, plus de 120 raccourcis réassignables
- **15 langues** — Anglais, coréen, japonais, chinois (simplifié et traditionnel), allemand, français, espagnol, portugais, italien, russe, turc, arabe, thaï, vietnamien
- **Renforcement CJK** (1.3.0) : extraction d'archives `.alz` (`unalz` intégré), aperçu en ligne de `.hwpx`, recherche multilingue de noms de fichiers (Pinyin / Romaji / Hangul-Hanja)
- **Renforcement CJK étendu** (1.3.1) : signets automatiques pour dossiers de messagerie/cloud (KakaoTalk · LINE · WeChat · iCloud Drive — 12 chemins), repli plein/demi-largeur dans la recherche, conversion d'encodage EUC-KR/Shift-JIS/GBK → UTF-8 (avec annulation), détection heuristique des formulaires gouvernementaux coréens

---

## Installation

Obtenez la dernière version sur [unifyl.app/download](https://github.com/goodbug89/Unifyl.app/releases/latest).

Également disponible sur le [Mac App Store](https://apps.apple.com/app/unifyl/id6773977314?mt=12) (édition sandbox — voir la FAQ pour les différences de fonctionnalités).

1. Ouvrez le fichier `.dmg`
2. Glissez Unifyl dans votre dossier Applications
3. Lancez l'application et accordez l'accès aux fichiers lorsque demandé

---

## Tarifs

| | **Free** | **Pro — $24.99** |
|---|---|---|
| Mises en page | Double, triple, division libre (toutes) | Simple, double, triple, libre |
| Recherche | Basique + Spotlight | Regex complet + recherche de contenu |
| Outils IA | — | Toutes les fonctions IA |
| Connexions distantes | FTP, SFTP | Tous les protocoles + stockage cloud |
| Archives | ZIP | ZIP, 7z, TAR, GZ, BZ2, XZ, ZSTD, RAR, **ALZ** |

> **Note sur les éditions.** La version du Mac App Store fonctionne en bac à sable et n'inclut pas le panneau Git, Docker/Kubernetes, les tunnels SSH, le terminal intégré, les visualiseurs de ports et de processus, ni la conversion multimédia basée sur ffmpeg — macOS ne l'autorise pas là. Tout le reste est identique, et Pro coûte le même prix unique des deux côtés. Choisissez le téléchargement direct si vous avez besoin de l'un de ces éléments.

Achat unique, sans abonnement. Téléchargement gratuit sur le Mac App Store (Pro via achat intégré, lié à votre identifiant Apple avec le partage familial) ou en téléchargement direct (Pro via LemonSqueezy, utilisable sur 3 appareils). Essai Pro gratuit de 14 jours au premier lancement.

---

## Raccourcis clavier principaux

| Raccourci | Action |
|---|---|
| `⌘ + ⇧ + P` | Palette de commandes |
| `Tab` | Changer de panneau actif |
| `Return` | Ouvrir le fichier / entrer dans le dossier |
| `⇧ + Return` | Renommer |
| `Space` | Coup d'œil |
| `⌘ + Delete` | Placer dans la corbeille |
| `⌃ + G` | Grille de vignettes |
| `⌘ + T` | Nouvel onglet |
| `⌘ + 2` | Double panneau |
| `⌘ + K` | Terminal |

Les plus de 120 raccourcis sont personnalisables dans « Réglages > Raccourcis clavier ».

---

## Compiler depuis les sources

```bash
git clone https://github.com/goodbug89/Unifyl.app.git
cd unifyl
make doctor    # Vérifier la chaîne d'outils
make gen       # Générer le projet Xcode
make build     # Compiler tous les paquets en séquence
```

Liste complète des fonctionnalités, guide de contribution et développement de plug-ins dans le [README en anglais](README.md).

---

## Licence

Propriétaire. Copyright 2024–2026 Unifyl. Tous droits réservés.

<p align="center">
  <a href="https://unifyl.app/fr/">Site web</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">Documentation</a>
</p>
