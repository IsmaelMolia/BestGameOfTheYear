# 👑 Projet : Civ-Like Pygame — Document de Vision (Reviewed)

> **« Construire un empire à partir d'un simple colon, tour après tour. »**  
> Un jeu de stratégie 4X en 2D vue du dessus, inspiré de *Civilization VI*, développé de zéro en **Python & Pygame**.

---

## 🧭 1. Résumé Exécutif & Vision du Projet

L'objectif de **Civ-Like Pygame** est de concevoir un jeu de stratégie au tour par tour reposant sur les piliers du genre **4X** (*Explore, Expand, Exploit, Exterminate*) sur une **grille hexagonale**.

Le projet vise un double objectif :
1. **Un gameplay immersif et complet** : Offrir les sensations d'un vrai jeu de gestion d'empire (colonisation, économie, frontière dynamique, guerre et technologie) dans un univers 2D épuré et lisible.
2. **Un défi technique et formateur** : Maîtriser de bout en bout l'architecture d'un jeu vidéo complexe en Python sans moteur lourd (sans Unity/Godot), en créant son propre moteur de rendu hexagonal, son interface utilisateur modulaire et ses algorithmes procéduraux.

---

## 🏛️ 2. Les 4 Piliers du Gameplay (4X)

```
        ╔══════════════════════════════════════════════════════════════╗
        ║                       CIV-LIKE PYGAME                        ║
        ╚══════════════════════════════════════════════════════════════╝
                 │                     │                    │
        ┌────────┴────────┐   ┌────────┴────────┐  ┌────────┴────────┐
        ▼                 ▼   ▼                 ▼  ▼                 ▼
   🧭 eXplore        🏰 eXpand         🌾 eXploit        ⚔️ eXterminate
  Brouillard,      Cités, frontières  Ressources,       Déplacement,
  biomes variés,   dynamiques par la  fermes, mines,    coûts en PM,
  ruines & scouts   nourriture locale  économie d'empire combats directs
```

| Pilier | Mécaniques en jeu | Statut |
| :--- | :--- | :---: |
| 🧭 **eXplore** *(Explorer)* | Carte générée procéduralement, brouillard de guerre (*Fog of War*), découverte progressive des ressources stratégiques et des empires rivaux. | 🟡 Partiel |
| 🏰 **eXpand** *(S'étendre)* | Déplacement du Colon (*Settler*), fondation de cités, extension automatique des frontières par accumulation de nourriture locale. | 🟢 Fonctionnel |
| 🌾 **eXploit** *(Exploiter)* | Récolte de matières premières (nourriture, bois, pierre, or), construction d'aménagements (fermes, moulins, scieries, marchés). | 🟢 Fonctionnel |
| ⚔️ **eXterminate** *(Combattre)* | Déplacement tactique case par case (coûts PM selon terrain), combats avec riposte réciproque, prise de cités adverses. | 🟡 Partiel |

---

## 🎮 3. Ce que le joueur peut faire (État des Fonctionnalités)

### 🖥️ Interface & Paramétrage
- [x] **Menu d'accueil interactif** : Transitions fluides, thème dynamique et curseurs sur-mesure.
- [x] **Gestion des options & graphismes** : Choix de la résolution, échelle de l'interface, zoom des boutons.
- [x] **Configuration de partie personnalisée** :
  - Taille de la carte (nombre de couches concentriques).
  - Répartition des biomes en pourcentage (plaines, forêts, collines, montagnes, eau).
  - Nombre de joueurs (1 à 4).
  - Activation ou désactivation du brouillard de guerre.
- [x] **Système de sauvegardes robuste** : 10 slots disponibles, sauvegarde/chargement au format JSON avec persistance intégrale de la carte et des joueurs.
- [x] **Menu Pause & Aide** : Affichage à tout instant des raccourcis clavier et de l'aide en jeu.

### 🗺️ Navigation & Découverte
- [x] **Contrôle caméra libre** : Déplacement au clavier (`Z, Q, S, D`), recentrage immédiat (`C`) et focus automatique sur la capitale.
- [x] **Zoom continu à la molette** : Recalcul dynamique de l'échelle des hexagones et des sprites.
- [x] **Modes de vue multiples** :
  - **Mode Empire (`E`)** : Frontières colorées par joueur.
  - **Mode Ressources (`R`)** : Icônes des ressources sur la carte.
  - **Brouillard de guerre (`TAB`)** : Découverte progressive du monde.

### 👑 Gestion & Stratégie
- [x] **Fondation de ville** : Le Colon s'établit et annexe les hexagones voisins.
- [x] **Cycle de tours** : Fin de tour (`Espace`), calcul des gains nets, régénération des points de mouvement (PM).
- [x] **Croissance territoriale** : La ville absorbe de nouvelles tuiles frontalières à mesure qu'elle stocke de la nourriture.
- [x] **Menu de Construction (`B`)** : Pose de bâtiments agricoles et économiques avec surbrillance des tuiles constructibles en vert/rouge.
- [x] **Panneau d'Inspection (`F`)** : Données détaillées sur la tuile, les rendements, les PV des unités et bâtiments.
- [ ] **Menu Cité Avancé** : File de production d'unités militaires (Guerrier, Archer, Cavalier) et de colons.
- [ ] **Exploitation des ressources rares** : Chevaux, fer, cristaux pour débloquer des unités spécialisées.
- [ ] **Arbre technologique** : Progression d'ères débloquant de nouveaux bâtiments et unités.
- [ ] **IA adverse dynamique** : Prise de décision autonome (déplacement de troupes, expansion, attaques).
- [ ] **Conditions de victoire / défaite** : Domination militaire, suprématie économique ou survie.

---

## 🎯 4. Pourquoi ce projet est passionnant (Objectifs Pédagogiques)

Ce projet représente un formidable terrain d'apprentissage pour tout développeur cherchant à dépasser les petits scripts pour concevoir un **système complexe et pérenne** :

```
┌────────────────────────────────────────────────────────────────────────┐
│                   DÉFIS D'INGÉNIERIE LOGICIELLE                       │
├────────────────────────────────────────────────────────────────────────┤
│ 📐 Mathématiques Hexagonales  │ Conversion pixel <-> coordonnées odd-r  │
│ 🎨 Moteur d'Interface        │ Système complet de GUI sans bibliothèque │
│ 🌍 Génération Procédurale     │ Croissance organique de continents cohérents│
│ 💾 Sérialisation & Données    │ Sauvegarde sans perte d'un graphe d'états│
│ 🧠 Intelligence Artificielle  │ Pathfinding A* et automates de décision  │
│ ⚡ Optimisation & Rendu       │ Culling hors-champ, gestion mémoire      │
└────────────────────────────────────────────────────────────────────────┘
```

### 1. Maîtriser la géométrie hexagonale
- Passer de la grille carrée classique à une **grille hexagonale (odd-r)** selon les principes de référence de *Red Blob Games*.
- Implémenter la détection de clic précise via distance euclidienne et calcul trigonométrique des sommets.

### 2. Concevoir un moteur d'interface (UI) maison
- Programmer de zéro des boutons réactifs, des champs d'entrée texte/chiffres, des menus contextuels rétractables et un HUD propre.
- Gérer l'adaptation de l'interface aux variations de résolution d'écran.

### 3. Génération procédurale de mondes
- Sortir du simple bruit aléatoire pour créer des continents réalistes avec des amas de forêts, des chaînes de montagnes protectrices et des bassins d'eau navigables.
- Ajuster les probabilités et la propagation des graines de biomes (*seed-based organic growth*).

### 4. Architecture logicielle & Clean Code
- Structurer un projet de plus de 3 000 lignes de code en modules à responsabilité unique (`game`, `map`, `players`, `units`, `ui`, `saves`).
- Rendre le code lisible, documenté et maintenable afin de pouvoir s'y replonger sans difficulté après plusieurs semaines de pause.

---

## 🔮 5. Ambitions & Idées d'Extensions Futures

| Domaine | Idée d'évolution | Impact sur le jeu |
| :--- | :--- | :--- |
| 🛡️ **Factions Asymétriques** | Proposer plusieurs civilisations avec des bonus exclusifs (ex: Nomades avec +1 PM, Bâtisseurs avec coût réduit en pierre). | Rejouabilité accrue |
| 🏹 **Combat Tactique Avancé** | Portée de tir pour les archers, bonus défensif sur les collines et forêts, système de contre (Piquier > Cavalier > Archer). | Profondeur stratégique |
| 📜 **Événements Aléatoires** | Catastrophes naturelles (sécheresse réduisant la nourriture), découvertes de ruines anciennes octroyant des bonus d'or ou de technologie. | Émergence narrative |
| 🎵 **Ambiance Sonore** | Musiques médiévales/acoustiques douces et effets sonores pour la sélection d'unités, les clics et le passage de tour. | Immersion totale |
