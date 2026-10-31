<div align="center">

# 👁️ ARGUS-VID

**Video Inspector — analyse vidéo locale, image par image**

![Version](https://img.shields.io/badge/version-2.1.0--alpha-orange)
![Statut](https://img.shields.io/badge/statut-alpha-red)
![Python](https://img.shields.io/badge/python-3.10--3.12-blue)
![UI](https://img.shields.io/badge/UI-PyQt6-41cd52)
![API](https://img.shields.io/badge/API-FastAPI-009688)
![Docker](https://img.shields.io/badge/Docker-optionnel-2496ed)

👉 **Dépôt de l'outil : [data2391/argus-vid](https://github.com/data2391/argus-vid)**

</div>

---

## 🚧 Statut du projet

> **ARGUS-VID 2.1.0-alpha** est la **première version alpha publique (2026)**.
> Elle peut planter, évoluer sans préavis et n'a pas été testée sur toutes
> les configurations. À ne pas utiliser en production.
>
> 🔒 **Une version durcie (« hardened ») est prévue prochainement**, orientée
> **cybersécurité** : isolation renforcée, dépendances verrouillées,
> réduction de la surface d'attaque et durcissement de l'automatisation.
>
> 🏁 **La version finale et officielle est prévue pour 2027.**

## 🗓️ Calendrier

| Étape | Période | Contenu |
|---|---|---|
| **2.1.0-alpha** | **2026** | Première alpha publique : fonctions de base, tests et retours |
| **Version durcie (hardened)** | Entre l'alpha et la finale | Orientée cybersécurité : isolation, dépendances figées, robustesse |
| **Version finale officielle** | **2027** | Release stable, licence définie, documentation complète |

---

## 🎯 Présentation

ARGUS-VID est une application **locale** d'analyse vidéo pour Windows, écrite
en Python (PyQt6). Elle permet d'inspecter une vidéo image par image, d'en
extraire les frames utiles, de détecter des visages et d'organiser le tout
dans des projets structurés. Un serveur FastAPI local permet de l'intégrer à
d'autres outils.

## ✨ Fonctionnalités

- 🎞️ **Lecteur précis** : navigation image par image, sauts de ±30 ms / ±1 s, bornes IN/OUT, cadences 1 / 5 / 10 / full FPS
- 🙂 **Détection de visages** : InsightFace (CUDA si disponible), sinon cascade Haar sur CPU
- 📁 **Projets nommés** : `projects/<id>/project.argus.json`, export ZIP portable
- 🗂️ **Dossiers virtuels** : création, renommage, glisser-déposer, navigation
- 🌍 **GEO Mode IA** : analyse multi-images, extraction structurée lieu / ville / pays, capture Google Earth satellite et coordonnées du centre de vue
- 🔌 **API locale** (`127.0.0.1`) : `GET /status`, `POST /extract` (en-tête `X-Api-Key`)

## ⚙️ Comment ça fonctionne

ARGUS-VID s'exécute entièrement sur ta machine. Un projet suit ce cycle :

1. **Import et projet** : tu charges une vidéo dans un projet nommé, enregistré dans `projects/<id>/project.argus.json` et exportable en ZIP.
2. **Navigation** : le lecteur (OpenCV) avance image par image ou par sauts de ±30 ms / ±1 s. Tu poses des bornes **IN / OUT** sur le passage utile.
3. **Échantillonnage** : entre IN et OUT, le moteur vidéo calcule les indices d'images à extraire selon la cadence choisie (1, 5, 10 FPS ou toutes les images).
4. **Extraction** : chaque image retenue est sauvegardée en JPEG (qualité 95) dans `frames/`, avec son numéro d'image et son horodatage en ms.
5. **Détection de visages** (optionnelle) : sur chaque frame, le moteur détecte les visages et enregistre chaque recadrage (avec marge) dans `faces/`, avec un score de confiance. **InsightFace** (`buffalo_l`, CUDA si disponible) est utilisé en priorité ; la **cascade Haar OpenCV** (CPU) prend le relais automatiquement s'il est indisponible.
6. **Classement** : les résultats vont dans des **dossiers virtuels** pour organiser l'analyse.
7. **Export** : le projet s'exporte en ZIP portable.

### 🌍 GEO Mode IA

La fonction GEO identifie le lieu public visible dans les frames :

1. Playwright pilote Chromium avec un profil persistant (`.pw-profile/`) et ouvre le **Mode IA de Google**.
2. Jusqu'à 10 frames sont envoyées, avec une consigne demandant le lieu, la ville et le pays.
3. L'outil attend une réponse structurée, puis la valide. Si le lieu est « inconnu », le résultat est rejeté.
4. Il ouvre **Google Earth** (vue satellite) sur le lieu trouvé, récupère les coordonnées du centre de vue et fait une capture d'écran.
5. Tout est archivé dans un dossier horodaté : `answer.txt`, `result.json`, la capture Earth, plus un lien Google Maps.

### 🔌 API locale

Un serveur FastAPI écoute **uniquement sur `127.0.0.1`**. Il expose `GET /status` et `POST /extract`, protégés par un en-tête `X-Api-Key` (clé aléatoire générée au premier lancement, stockée dans `config.json`). La documentation OpenAPI est désactivée.

### 🧱 Architecture

| Module | Rôle |
|---|---|
| `app.py` | Interface PyQt6 |
| `core/video_engine.py` | Échantillonnage et extraction des frames |
| `core/face_engine.py` | Détection de visages (InsightFace / Haar) |
| `core/geolocator.py` | GEO via Playwright (Mode IA + Earth) |
| `core/project_manager.py` | Projets, dossiers virtuels, export ZIP |
| `core/thumbnailer.py` | Miniatures |
| `server/` | API FastAPI locale |
| `containers/` | Docker Compose |

## 🚀 Installation rapide (Windows)

1. Clone le dépôt de l'outil :
   ```sh
   git clone https://github.com/data2391/argus-vid.git
   cd argus-vid
   ```
2. Double-clique sur **`run.bat`**.

Le script détecte Python 3.10–3.12, crée le venv, installe les dépendances (dont la stack NVIDIA et Playwright/Chromium), génère `config.json` depuis `config.example.json`, puis lance l'application. Journal : `logs/setup-live.log`.

### 🐳 Docker (optionnel, isolation partielle)

```sh
docker build -t argus-vid:2.1 .
docker compose -f containers/docker-compose.yml up --build
```

Python, FFmpeg et Chromium sont séparés du système hôte, et aucun port externe n'est publié par défaut. L'isolation reste **partielle** : le socket X11/WSLg est monté et `ipc: host` est activé. Prérequis : Docker Desktop + WSL2 sous Windows, GUI Qt via WSLg (Windows 11) ou serveur X11, GPU via GPU-PV avec pilote NVIDIA WSL à jour. L'image Docker utilise la détection Haar (CPU) : InsightFace n'y est pas installé.

## 🗺️ Roadmap

- [x] Première version alpha (2.1.0-alpha, 2026)
- [ ] Retours utilisateurs et correctifs de stabilité
- [ ] **Version durcie orientée cybersécurité**
  - [ ] Dépendances figées (fichier de verrouillage)
  - [ ] Isolation Docker renforcée
  - [ ] Automatisation GEO plus robuste
  - [ ] Chemins relatifs dans les projets
  - [ ] Licence définie
- [ ] **Version finale officielle (2027)**

## ⚠️ Limites connues (alpha)

- La fonction GEO pilote l'interface web de Google avec Playwright : elle peut casser à chaque changement de page, déclencher des CAPTCHA et exposer le compte Google utilisé à des restrictions. Utilise un compte dédié.
- Le profil navigateur `.pw-profile/` contient des cookies de session : ne le partage jamais.
- Les projets enregistrent des chemins de fichiers absolus.
- Dépendances non figées (pas de fichier de verrouillage).
- Non vérifié : Qt sous Windows, GPU réel, session Google réelle, build Docker complet.

## 🔐 Sécurité

Voir le [SECURITY.md](https://github.com/data2391/argus-vid/blob/main/SECURITY.md) du dépôt de l'outil. Pour signaler une vulnérabilité, utilise **Security → Report a vulnerability** sur [argus-vid](https://github.com/data2391/argus-vid).

## 💬 Retours

Cette alpha sert à recueillir des retours. Ouvre une [issue](https://github.com/data2391/argus-vid/issues) pour un bug, une idée ou un test sur ta configuration.

## 📄 Licence

À définir. Tant qu'aucun fichier `LICENSE` n'est présent, tous les droits sont réservés. PyQt6 est sous GPL v3 ou licence commerciale, et les modèles pré-entraînés InsightFace peuvent avoir des conditions d'usage propres à vérifier.

---

<div align="center">

Développé par [data2391](https://github.com/data2391)

</div>
