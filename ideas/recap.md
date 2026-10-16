# 🌍 Récapitulatif du Projet : Civ-Like Pygame

> **Genre** : Jeu de stratégie 4X au tour par tour (*Civilization-like*)  
> **Technologies** : Python 3, Pygame  
> **Auteur** : Ismaël Molia  
> **Dossier du projet** : `civ_like_pygame/`  

---

## 📌 1. Vue d'ensemble du jeu

Ce projet est un jeu de stratégie 4X (*eXplore, eXpand, eXploit, eXterminate*) développé en Python avec **Pygame**. Il reprend les principes fondamentaux de la saga **Civilization** sur une grille hexagonale en 2D vue du dessus :
- **Exploration** : Carte hexagonale générée procéduralement avec biomes variés et système de brouillard de guerre (*Fog of War*).
- **Expansion** : Déplacement de colons (*settlers*), fondation de cités et extension territoriale dynamique alimentée par la nourriture locale.
- **Exploitation** : Récolte de ressources (nourriture, bois, pierre, or), construction de bâtiments régionaux (fermes, moulins, scieries, carrières, marchés).
- **Extermination** : Déplacement d'unités avec coûts en points de mouvement (PM) selon le terrain, combats directs et prise de territoire.

---

## 🏗️ 2. Architecture du code & Organisation des modules

Le projet est structuré de façon modulaire sous le dossier `civ_like_pygame/scripts/` :

```text
civ_like_pygame/
├── assets/
│   ├── backgrounds/          # Fonds d'écran des différents menus
│   ├── buildings/            # Sprites des bâtiments (fermes, moulins, etc.)
│   ├── cursors/              # Curseurs personnalisés selon l'action en cours
│   ├── ressources/           # Icônes des ressources (pomme, bois, pierre, or...)
│   └── units/                # Sprites des unités (colon, château/ville, etc.)
└── scripts/
    ├── main.py               # Point d'entrée de l'application
    ├── game.py               # Moteur principal et boucle de jeu (Game Loop)
    ├── constants.py          # Constantes globales, équilibrage, chemins et données
    ├── map.py                # Grille hexagonale, génération procédurale, caméra et rendu
    ├── players.py            # Modèle joueur (humain & IA), inventaire, gestion des tours
    ├── units.py              # Classes Unit, Settler, City, Farm et règles de combat/déplacement
    ├── ui.py                 # Système d'interface utilisateur complet (boutons, pages, HUD)
    ├── saves_handler.py      # Persistance des données (sauvegardes et paramètres JSON)
    └── data/
        ├── settings.json     # Configuration de l'affichage et de l'interface
        └── saves/            # Fichiers de sauvegarde (save_1.json à save_10.json)
```

---

## 🧩 3. Description détaillée des scripts

### 🚀 `main.py`
- Point d'entrée minimaliste et propre.
- Initialise Pygame (`pygame.init()`), instancie la classe `Game` et démarre la boucle principale via `game.game_loop()`.

---

### ⚙️ `game.py`
- **Rôle** : Chef d'orchestre de l'application.
- **Boucle de jeu (`game_loop`)** :
  - Cadence stable à **60 FPS** (`Clock.tick(60)`).
  - Gestion centralisée des événements clavier et souris (`check_events`).
  - Déplacement fluide de la caméra avec maintien des touches (`Z, Q, S, D`).
  - Rendu et rafraîchissement écran (`ui.show()`, `pygame.display.update()`).
  - Sauvegarde automatique de la configuration et de l'état de la partie en quittant.
- **Curseurs personnalisés** : Adapte le curseur système selon le contexte (`default`, `build`, `city`, `unit`, `building`, `moving`).

---

### 🗺️ `map.py` & Mathématiques Hexagonales
Gère toute la logique spatiale, le monde et son affichage.

1. **Mathématiques de la grille hexagonale** :
   - Système de coordonnées décalées **odd-r** (décalage horizontal d'une demi-colonne sur les lignes impaires).
   - Calcul trigonométrique des 6 sommets (`get_points`) pour le rendu vectoriel des hexagones et de leurs bordures.
   - Conversion bidirectionnelle : coordonnées grille $\leftrightarrow$ position en pixels à l'écran (`coo_to_real_pos_centered` et `real_pos_to_coo_centered`).
   - Détection précise du survol souris par distance euclidienne par rapport au rayon de l'hexagone.

2. **Génération procédurale de la carte** :
   - **Couches concentriques (`layers`)** : Construction d'un monde circulaire par anneaux concentriques autour du centre.
   - **Croissance organique de biomes (`organic_tiles` / `seed_generation_v3`)** :
     - Calcul du nombre de graines (*seeds*) selon les pourcentages de fréquence configurés.
     - Croissance itérative des biomes vers les voisins libres jusqu'à atteindre les tailles limites prévues pour chaque type de terrain.
     - Remplissage automatique de secours (*plain*) si nécessaire pour éviter les trous.

3. **Système de caméra & Zoom** :
   - Zoom continu à la molette de la souris avec recalcul instantané des échelles et des rayons d'hexagones.
   - Déplacement libre (`Z, Q, S, D`) ou recentrage immédiat (`C`).
   - Recentrage automatique sur le colon du joueur au démarrage de la partie (`focus_camera`).

4. **Modes de vision** :
   - **Mode Empire (`E`)** : Colorie les bordures et fonds de tuiles selon la couleur du joueur propriétaire.
   - **Mode Ressources (`R`)** : Superpose les icônes de ressources naturelles sur les cases.
   - **Brouillard de guerre (`TAB`)** : Masque les cases non explorées par les unités et bâtiments du joueur.

---

### 👥 `players.py`
- **Classe `Player`** :
  - **Stock de ressources** : Suivi des réserves (nourriture, bois, pierre, or...).
  - **Économie au tour par tour** : Calcul des revenus nets par tour (`total_ressource_income`) en combinant le rendement des tuiles possédées, les gains des bâtiments et les coûts d'entretien des unités.
  - **Territoire** : Liste des cités et des tuiles rattachées à chacune (`tiles_owned`).
  - **Champ de vision** : Ensemble des tuiles découvertes (`tiles_seen`) mis à jour au fil des déplacements d'unités.
  - **Fin de tour (`finish_turn`)** : Réapprovisionne les stocks, réinitialise les points de mouvement des unités, calcule la croissance et l'expansion des villes.
- **Classe `AI_Player`** :
  - Hérite de `Player` pour implémenter des comportements automatisés pour les autres empires (ex. fondation de ville au tour 1).
- **Classe `PlayerHandler`** :
  - Gère de 1 à 4 joueurs avec attribution aléatoire de couleurs.
  - Génère le colon de départ pour chaque joueur sur une case favorable.
  - Fait tourner la boucle des tours de jeu (`next_turn`).

---

### ⚔️ `units.py`
Définit les entités mobiles et les structures du jeu.

1. **Classe de base `Unit`** :
   - Attributs : Points de vie (`hp`), points d'attaque (`damage`), points de mouvement (`movement`), coût d'entretien, sprite proportionnel à l'hexagone.
   - Déplacement case par case avec déduction de PM selon le coût du terrain (`TILES_MOVEMENT_COST`).
   - Interdiction de déplacement sur l'eau et les montagnes pour les unités terrestres.
   - **Système de combat** : Déplacer une unité sur une case ennemie déclenche un échange de dégâts réciproque jusqu'à la mort éventuelle d'une unité.
   - Indicateur visuel au-dessus de l'unité sélectionnée (flèche verte pour unité alliée, rouge pour ennemie).

2. **`Settler` (Colon)** :
   - Unité fondatrice capable d'effectuer une action spéciale (`A`).
   - Fonde une ville (`City`) sur sa case et disparaît.
   - Avec l'option `BIG_SPAWN = True`, annexe immédiatement la case centrale + les 6 cases adjacentes au territoire de la cité.

3. **`City` (Ville)** :
   - Considérée comme une unité stationnaire / bâtiment majeur (100 PV, 5 dégâts, produit de l'or).
   - **Croissance et expansion automatique** :
     - Accumule la nourriture locale des cases de son territoire chaque tour (`calculate_expansion`).
     - Quand le seuil requis (`food_expansion`) est atteint, la ville s'étend automatiquement en annexant une case frontalière non revendiquée.
     - Le seuil requis augmente à chaque nouvelle tuile absorbée (+15 nourriture).
   - Calcul de la production régionale cumulée (`regional_prod`).

4. **`Farm` (Ferme)** :
   - Bâtiment agricole constructible sur les plaines et collines apportant +3 de nourriture par tour.

---

### 🖥️ `ui.py`
Interface utilisateur complète codée sur-mesure sans bibliothèque externe :

1. **Composants d'interface** :
   - `Button` : Boutons interactifs avec gestion du survol, zoom, arrondis, clics gauche/droit et mode saisie textuelle/numérique (`inputable`).
   - `ButtonHandler` : Gestion des états, réconciliation des clics et exécution des actions.
   - `PageHanlder` : Gestionnaire des scènes et affichages d'écrans.

2. **Écrans du jeu (Pages 0 à 5)** :
   - **Page 0 (Start Menu)** : Démarrage, Options, Quitter.
   - **Page 1 (Options Menu)** : Réglage de la résolution (largeur/hauteur), taille/zoom des boutons, couleurs de thème.
   - **Page 2 (Save Menu)** : Nouvelle partie ou chargement.
   - **Page 3 (Saves Page)** : Gestion des 10 slots de sauvegarde (chargement par clic gauche, suppression par clic droit, bouton "Clear Saves").
   - **Page 4 (Game Configuration)** : Paramétrage avancé avant génération :
     - Taille de la carte (nombre de couches concentriques).
     - Fréquence de chaque biome (plaine, eau, montagne, colline, forêt) en % avec calcul dynamique du total.
     - Nombre de joueurs (1 à 4).
     - Activation / désactivation du brouillard de guerre.
   - **Page 5 (Game Page)** : La carte de jeu en plein écran avec l'océan animé en arrière-plan et le HUD complet.

3. **Panneaux latéraux contextuels (`Side Pages`)** :
   - **`UI` (Barre supérieure)** : Affichage du tour en cours, des stocks de chaque ressource et du solde net par tour en vert/blanc/rouge.
   - **`TILE`** : Panneau de détails de la case sélectionnée (coordonnées relatives, propriétaire, type de terrain).
   - **`CURRENT_UNIT`** : PV, points de mouvement restants et état du mode déplacement.
   - **`CURRENT_BUILDING`** : Type de bâtiment et PV.
   - **`CURRENT_CITY`** : Statistiques de la ville et production locale.
   - **`BUILD`** : Menu de construction (ferme, moulin, scierie, carrière, marché) avec mise en évidence sur la carte des tuiles valides en vert et interdites en rouge.
   - **`INSPECT`** : Panneau encyclopédique détaillant les rendements et caractéristiques de l'élément inspecté.
   - **`PAUSE` & `COMMANDS`** : Menu pause et aide des raccourcis clavier.

4. **Effet graphique procédural** :
   - Animation d'ondes marines (`animated_sea`) dessinant des cercles concentriques dynamiques sous la carte pour simuler l'océan.

---

### 💾 `saves_handler.py`
- Sauvegarde et charge les paramètres d'affichage dans `data/settings.json`.
- Gère jusqu'à 10 emplacements de sauvegarde de partie dans `data/saves/save_X.json` :
  - **État de la carte** : Coordonnées, types de tuiles, ressources stratégiques.
  - **État des empires** : Tour actuel, réserves de ressources, cités et territoires rattachés, unités actives avec leurs PV/PM, historique des tuiles découvertes.
- Fonctions utilitaires : détection automatique du premier slot libre, suppression d'une sauvegarde ciblée, nettoyage total.

---

## 🎮 4. Tableau des Commandes et Raccourcis

| Touche | Action | Description |
| :--- | :--- | :--- |
| **`Z, Q, S, D`** | Déplacer la caméra | Navigation fluide sur la carte |
| **`Molette Souris`**| Zoom Caméra | Agrandir ou rétrécir la vue de la carte |
| **`C`** | Recentrer la caméra | Réinitialise la caméra au centre initial |
| **`Clic Gauche`** | Sélectionner / Valider | Sélectionne une case, une unité ou clique sur un bouton de menu |
| **`Espace`** | Fin de tour | Passe au tour suivant et résout les actions des joueurs / IA |
| **`Échap`** | Pause / Menu | Ouvre ou ferme le panneau de pause |
| **`1`** | Sélectionner l'unité | Active le panneau de l'unité présente sur la case survolée |
| **`2`** | Sélectionner le bâtiment | Active le panneau du bâtiment présent sur la case |
| **`3`** | Mode déplacement / attaque | Prépare le déplacement de l'unité vers la case ciblée |
| **`A`** | Action spéciale | Fonde une ville si un Colon (*Settler*) est sélectionné |
| **`B`** | Menu de construction | Affiche les bâtiments constructibles et colore les tuiles éligibles |
| **`V`** | Vue de la ville | Affiche les informations détaillées de la cité |
| **`F`** | Inspecter | Ouvre le panneau d'inspection détaillé (stats, rendements) |
| **`E`** | Vision Empire | Met en évidence les frontières territoriales des empires |
| **`R`** | Vision Ressources | Affiche les icônes de ressources directement sur la carte |
| **`TAB`** | Brouillard de guerre | Active ou désactive le brouillard de guerre (*cheat / debug*) |

---

## 🌾 5. Économie, Biomes et Rendements

### Biomes et Rendements naturels
| Biome | Couleur | Rendement de base | Coût de déplacement | Bâtiments compatibles |
| :--- | :--- | :--- | :---: | :--- |
| **Plaine** (*Plain*) | Vert vif | 2x Nourriture (*Food*) | 1.0 PM | Ferme, Moulin, Marché |
| **Forêt** (*Forest*) | Vert forêt | 1x Nourriture, 1x Bois (*Wood*) | 1.5 PM | Bûcheron, Marché |
| **Colline** (*Hill*) | Brun doré | 1x Pierre (*Rock*) | 1.5 PM | Ferme, Moulin, Carrière, Marché |
| **Montagne** (*Mountain*) | Gris roche | 2x Pierre (*Rock*) | 2.0 PM | Carrière |
| **Eau** (*Water*) | Bleu lumineux | 1x Nourriture (*Food*) | 0.5 PM (Naval) | - |

### Économie des unités et bâtiments
- **Colon (`Settler`)** : 10 PV, 2 Dégâts, 3 PM. Coût d'entretien : 1 nourriture/tour.
- **Ville (`City`)** : 100 PV, 5 Dégâts, 5 Production. Revenu : +5 Or/tour.
- **Ferme (`Farm`)** : 30 PV, 3 Production. Revenu : +3 Nourriture/tour.
- **Expansion d'une ville** : Nécessite 15 nourritures cumulées de base + 15 supplémentaires par tuile déjà possédée (maximum 15 tuiles par ville).

---

## 💡 6. Points forts & Pistes d'évolution future

### ✅ Points forts déjà en place
1. **Moteur mathématique hexagonal robuste** avec calculs d'angles, gestion de caméra et zoom fluide sans désalignement.
2. **Génération procédurale organique** créant des continents et archipels cohérents plutôt que du bruit aléatoire basique.
3. **Boucle 4X complète** : Colonisation $\rightarrow$ Exploitation $\rightarrow$ Gestion des ressources $\rightarrow$ Expansion de territoire $\rightarrow$ Combat.
4. **Interface utilisateur 100% autonome** en Pygame (boutons avec zoom, inputs de texte pour paramétrer la partie, curseurs dynamiques).
5. **Système de sauvegarde complet en JSON** garantissant la persistance intégrale d'une session de jeu.

### 🔮 Pistes d'évolution (issues des notes de développement)
- **Pathfinding A\*** : Permettre aux unités de se déplacer sur plusieurs cases en calculant le chemin optimal en fonction des coûts de terrain.
- **Intelligence Artificielle renforcée** : Développer l'IA pour qu'elle construise des bâtiments, déplace ses troupes et étende son empire au-delà du tour 1.
- **Ressources stratégiques actives** : Exploitation effective des chevaux, du fer et des cristaux de mana pour débloquer des unités avancées.
- **Arbre technologique & Nouvelles unités** : Intégrer les sprites déjà présents dans les assets (`archer.png`, `knight.png`, `farmer.png`).
- **Optimisation du rendu** : Optimiser `seed_generation_v3()` et le rendu des tuiles pour permettre des cartes géantes avec des centaines de couches.
