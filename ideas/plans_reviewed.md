# 📐 Plans Techniques & Fonctionnels Détaillés : Civ-Like Pygame (Reviewed)

> **Spécifications d'Architecture et Conception Fonctionnelle**  
> Ce document détaille le fonctionnement interne de chaque système du jeu, de la géométrie hexagonale à l'économie, pour servir de référence technique pérenne.

---

## 📑 Sommaire
1. [Architecture Globale & Boucle Principale](#1-architecture-globale--boucle-principale)
2. [Moteur de Grille Hexagonale & Rendu](#2-moteur-de-grille-hexagonale--rendu)
3. [Génération Procédurale du Monde](#3-génération-procédurale-du-monde)
4. [Moteur d'Interface Utilisateur (UI Custom)](#4-moteur-dinterface-utilisateur-ui-custom)
5. [Système Économique, Cités & Territoire](#5-système-économique-cités--territoire)
6. [Système d'Unités, Déplacement & Combat](#6-système-dunités-déplacement--combat)
7. [Persistance des Données & Sauvegardes JSON](#7-persistance-des-données--sauvegardes-json)
8. [Spécifications Cibles pour l'Intelligence Artificielle](#8-spécifications-cibles-pour-lintelligence-artificielle)
9. [Arbre Technologique & Extensions Prévues](#9-arbre-technologique--extensions-prévues)

---

## 1. Architecture Globale & Boucle Principale

### 🔄 Diagramme de Flux de la Boucle de Jeu (`Game Loop`)
Le fichier `main.py` initialise Pygame et délègue l'exécution à l'instance de `Game` (`game.py`) :

```
                ┌───────────────────────────────────┐
                │          pygame.init()            │
                │          game = Game()            │
                │         game.game_loop()          │
                └─────────────────┬─────────────────┘
                                  │
                                  ▼
                     ┌─────────────────────────┐
       ┌────────────►│   Horloge : 60 FPS      │
       │             │    clock.tick(60)       │
       │             └────────────┬────────────┘
       │                          │
       │                          ▼
       │             ┌─────────────────────────┐
       │             │   Gestion Événements    │◄── Clavier, Souris, Molette
       │             │   check_events(events)  │
       │             └────────────┬────────────┘
       │                          │
       │                          ▼
       │             ┌─────────────────────────┐
       │             │  Mise à Jour Caméra /   │◄── Déplacement Z, Q, S, D
       │             │      Curseurs UI        │
       │             └────────────┬────────────┘
       │                          │
       │                          ▼
       │             ┌─────────────────────────┐
       │             │     Rendu Graphique     │◄── Océan, Hexagones, Unités,
       │             │      ui.show(...)       │    HUD & Panneaux contextuels
       │             └────────────┬────────────┘
       │                          │
       │                          ▼
       │             ┌─────────────────────────┐
       │             │  pygame.display.update()│
       │             └────────────┬────────────┘
       │                          │
       └──────────────────────────┘
```

### 🧩 Responsabilités des Modules
- **`game.py`** : Point central connectant la carte (`Map`), les joueurs (`PlayerHandler`), l'interface (`UI`) et le gestionnaire de persistance (`SaveHandler`).
- **`constants.py`** : Source unique de vérité pour les paramètres du jeu (couleurs RVB, rendements de tuiles, caractéristiques d'unités, dimensions par défaut).
- **`map.py`** : Représentation spatiale pure, calculs géométriques et rendu des polygones.
- **`players.py`** : Données des empires (ressources, territoires, vision, gestion du passage de tour).
- **`units.py`** : Entités mobiles et statiques (stats, actions, collisions, combats).
- **`ui.py`** : Système de fenêtrage, hiérarchie de boutons et saisies utilisateur.
- **`saves_handler.py`** : I/O disque, sérialisation et désérialisation JSON.

---

## 2. Moteur de Grille Hexagonale & Rendu

### 📐 Géométrie & Coordonnées « odd-r »
Le jeu utilise une grille hexagonale en orientation **pointy-topped** (sommet pointé vers le haut) avec un décalage horizontal d'une demi-colonne sur les lignes impaires (*odd-row horizontal layout*).

```
        Col 0       Col 1       Col 2
       /     \     /     \     /     \
      | (0,0) |   | (1,0) |   | (2,0) |   ... (Ligne 0 - paire)
       \     / \ / \     / \ / \     /
        -----   |   -----   |   -----
               / \         / \
              | (0,1) |   | (1,1) |       ... (Ligne 1 - décalée d'un demi-rayon)
               \     /     \     /
```

- **Calcul des sommets d'un hexagone** :  
  Pour un centre $(X_c, Y_c)$ et un rayon $R$, les 6 sommets à un angle $\theta_i = 60^\circ \times i - 30^\circ$ sont calculés via :
  $$X_i = X_c + R \cdot \cos(\theta_i)$$
  $$Y_i = Y_c + R \cdot \sin(\theta_i)$$
- **Espacement entre tuiles** :
  - Distance horizontale entre colonnes : $\Delta X = \sqrt{3} \cdot R$
  - Distance verticale entre lignes : $\Delta Y = \frac{3}{2} \cdot R$
  - Décalage horizontal pour lignes impaires : $+\frac{\sqrt{3}}{2} \cdot R$

### 🎥 Caméra & Zoom Dynamique
- **Position caméra** : Vecteur $(Cam_X, Cam_Y)$ appliqué comme translation globale à toutes les coordonnées réelles avant dessin.
- **Facteur d'échelle (`ratio_zoom`)** :
  - La molette modifie la taille de référence $R$.
  - Les positions et sommets sont recalculés à la volée.
- **Recentrage (`C` / `focus_camera`)** : Aligne immédiatement la caméra sur le centre géométrique de l'unité active ou de la capitale du joueur.

---

## 3. Génération Procédurale du Monde

### 🌍 Algorithme de Génération Organique (`seed_generation_v3`)
Plutôt qu'un simple bruit de Perlin pur, la carte est générée selon une croissance biologique par graines :

```
1. Définition du monde
   └── Génération de N couches concentriques ('layers') d'hexagones vides.

2. Répartition des graines (Seeds)
   └── Placement aléatoire de graines selon les pourcentages configurés en Page 4 :
       Plaines (%), Eau (%), Montagnes (%), Collines (%), Forêts (%).

3. Croissance itérative (Organic expansion)
   ├── Pour chaque graine :
   │   ├── Sélection des voisins libres immédiats.
   │   ├── Propagation du biome jusqu'à épuisement du quota de tuiles calculé.
   └── Les biomes s'agglomèrent naturellement en continents cohérents.

4. Passe de nettoyage & Remplissage
   └── Les cases orphelines restantes sont complétées par le biome par défaut ('plain').
```

### 🗺️ Tableau des Biomes & Propriétés
| Biome | Rendement Naturel | Coût PM | Bâtiments Autorisés | Franchissable Terrestre |
| :--- | :--- | :---: | :--- | :---: |
| **Plaine** | +2 Nourriture | 1.0 PM | Ferme, Moulin, Marché | ✅ Oui |
| **Forêt** | +1 Nourriture, +1 Bois | 1.5 PM | Bûcheron, Marché | ✅ Oui |
| **Colline** | +1 Pierre | 1.5 PM | Ferme, Moulin, Carrière | ✅ Oui |
| **Montagne** | +2 Pierre | 2.0 PM | Carrière | ❌ Non (Barrière naturelle) |
| **Eau** | +1 Nourriture | 0.5 PM (Naval)| Aucun | ❌ Unités terrestres |

---

## 4. Moteur d'Interface Utilisateur (UI Custom)

### 🧱 Composants GUI
L'interface est entièrement codée en Pygame natif sans dépendance externe :
- **Classe `Button`** :
  - États : Repos, Survolé (*Hover* avec scale visuel), Cliqué.
  - Rendu avec bordures lissées et coins arrondis (`border_radius`).
  - Mode `inputable` : Permet la saisie textuelle ou numérique directe (gestion de `BACKSPACE`, caractères ASCII).
- **Classe `ButtonHandler`** :
  - Centralise la détection des clics gauche et droit.
  - Déclenche les fonctions de rappel (*callbacks*) correspondantes.
- **Classe `PageHandler`** :
  - Gère la machine à états des écrans (Pages 0 à 5).

### 🖥️ Arborescence des Écrans
```
Page 0 : Start Menu
├── Page 1 : Settings (Résolution, Zoom UI, Couleurs)
└── Page 2 : Save Selection
    ├── Page 3 : Saves Manager (10 slots JSON, Supprimer, Charger)
    └── Page 4 : Game Setup (Configuration de la nouvelle partie)
        └── Page 5 : Game Page (Partie active + HUD + Panneaux latéraux)
```

### 📑 Panneaux Latéraux Contextuels en Jeu (`Side Panels`)
- **HUD Supérieur** : Compteur de tours, stock des 4 ressources, solde net par tour coloré (Vert = positif, Rouge = négatif).
- **Panneau `TILE`** : Coordonnées, biome, ressources et faction propriétaire de l'hexagone survolé.
- **Panneau `CURRENT_UNIT`** : Barre de vie, points de mouvement restants, commande de déplacement (`3`).
- **Panneau `CURRENT_CITY`** : Production locale cumulée, seuil de nourriture pour la prochaine extension de frontière.
- **Panneau `BUILD` (`B`)** : Grille des constructions possibles avec coloration dynamique de la carte (vert = constructible, rouge = interdit).

---

## 5. Système Économique, Cités & Territoire

### 🏛️ Cycle de Vie d'une Cité
```
[Colon arrive sur case] ──(Touche 'A')──► [Fondation de la Ville]
                                                   │
                                                   ├─► Annexe la case centrale + 6 voisines
                                                   └─► Devient le centre économique local
```

### 📈 Formule d'Expansion Territoriale
Chaque tour, la ville consomme et accumule la nourriture produite par toutes les tuiles de son territoire :
$$\text{Food}_{\text{accumulée}} = \sum_{t \in \text{Territoire}} \text{RendementNourriture}(t)$$

Dès que $\text{Food}_{\text{accumulée}} \ge \text{SeuilExpansion}$, la ville choisit automatiquement une case frontalière neutre adjacente et l'annexe :
$$\text{SeuilExpansion} = 15 + (15 \times \text{NombreDeTuilesDéjàPossédées})$$

---

## 6. Système d'Unités, Déplacement & Combat

### ⚔️ Entités & Caractéristiques
```
                       ┌──────────────────────┐
                       │     Classe Unit      │
                       │ hp, damage, movement │
                       └──────────┬───────────┘
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
┌──────────────────┐                              ┌──────────────────┐
│     Settler      │                              │       City       │
│ 10 PV | 3 PM     │                              │ 100 PV | 0 PM    │
│ Action : Fonder  │                              │ Production & Or  │
└──────────────────┘                              └──────────────────┘
```

### 🚶 Déplacement & Coût de Terrain
1. Une unité sélectionnée active le mode déplacement (`3`).
2. Le joueur clique sur une tuile voisine.
3. Vérification des conditions :
   - La tuile n'est ni de l'eau ni une montagne (sauf unités spécifiques).
   - Les points de mouvement restants de l'unité sont supérieurs ou égaux au coût de la tuile cible :
     $$\text{PM}_{\text{restants}} \ge \text{Coût}(Biome)$$
4. En cas de succès : Déduction des PM et mise à jour des coordonnées de l'unité.

### 💥 Mécanique de Combat Réciproque
Si la tuile ciblée est occupée par une unité ennemie :
1. **Attaque** : L'attaquant inflige ses dégâts au défenseur :
   $$\text{HP}_{\text{défenseur}} \leftarrow \text{HP}_{\text{défenseur}} - \text{Damage}_{\text{attaquant}}$$
2. **Riposte** : Si le défenseur survit, il riposte immédiatement :
   $$\text{HP}_{\text{attaquant}} \leftarrow \text{HP}_{\text{attaquant}} - \text{Damage}_{\text{défenseur}}$$
3. **Résolution** : Si l'une des unités atteint $\le 0$ PV, elle est supprimée du jeu. Si le défenseur périt, l'attaquant victorieux s'installe sur la case.

---

## 7. Persistance des Données & Sauvegardes JSON

### 💾 Schéma de Données (`save_X.json`)
```json
{
  "turn": 42,
  "map_settings": {
    "layers": 12,
    "fog_of_war": true
  },
  "tiles": [
    {
      "coords": [0, 0],
      "biome": "plain",
      "owner": 0,
      "building": "farm"
    }
  ],
  "players": [
    {
      "id": 0,
      "is_ai": false,
      "color": [255, 60, 60],
      "inventory": { "food": 140, "wood": 65, "rock": 30, "gold": 120 },
      "cities": [
        {
          "name": "Capitale",
          "coords": [0, 0],
          "stored_food": 25,
          "tiles_owned": [[0, 0], [0, 1], [1, 0]]
        }
      ],
      "units": [
        {
          "type": "settler",
          "coords": [2, 3],
          "hp": 10,
          "movement": 3
        }
      ],
      "tiles_seen": [[0, 0], [0, 1], [1, 0], [2, 3]]
    }
  ]
}
```

---

## 8. Spécifications Cibles pour l'Intelligence Artificielle

L'architecture actuelle dispose d'un stub `AI_Player` dans `players.py`. Voici le plan d'implémentation de l'automate à états finis (**FSM**) pour les tours adverses :

```
                  ┌────────────────────────┐
                  │      DÉBUT DU TOUR     │
                  └───────────┬────────────┘
                              │
                              ▼
            [Le Colon existe-t-il toujours ?]
             ├── OUI ──► Chercher case optimale (plaine avec nourriture) ──► Fonder ville
             └── NON ──► Passer à la gestion de la ville
                              │
                              ▼
            [Économie : Ressources disponibles ?]
             ├── Assez d'or/bois/pierre ──► Construire bâtiment prioritaire (Ferme > Scierie)
             └── Sinon ─────────────────► Économiser
                              │
                              ▼
            [Unités militaires actives ?]
             ├── Ennemi en vue ─────────► Attaquer si ratio PV favorable / Se replier si faible
             └── Aucun ennemi ──────────► Patrouiller à la frontière du territoire
                              │
                              ▼
                  ┌────────────────────────┐
                  │       FIN DU TOUR      │
                  └────────────────────────┘
```

---

## 9. Arbre Technologique & Extensions Prévues

### 🔬 Arbre Technologique en 3 Ères

```
        ÈRE 1 : ANTIQUITÉ              ÈRE 2 : ÂGE CLASSIQUE            ÈRE 3 : MOYEN ÂGE
    ┌──────────────────────┐        ┌──────────────────────┐        ┌──────────────────────┐
    │     Agriculture      │───────►│      Irrigation      │───────►│    Rotation des      │
    │ Débloque : Ferme     │        │ Débloque : Moulin    │        │      Cultures        │
    └──────────┬───────────┘        └──────────────────────┘        └──────────────────────┘
               │
    ┌──────────▼───────────┐        ┌──────────────────────┐        ┌──────────────────────┐
    │     Travail du Bois  │───────►│    Métallurgie       │───────►│     Équitation       │
    │ Débloque : Bûcheron  │        │ Débloque : Carrière, │        │ Débloque : Écurie,   │
    │          & Guerrier  │        │          Archer      │        │          Chevalier   │
    └──────────────────────┘        └──────────────────────┘        └──────────────────────┘
```

### 🎯 Prochaines Fonctionnalités à Implémenter
1. **Pathfinding A\*** sur grille hexagonale avec calcul de la distance Manhattan adaptée :
   $$D = \frac{|\Delta x| + |\Delta y| + |\Delta z|}{2}$$
2. **File de production de ville** : Production en $N$ tours basée sur la statistique $\text{Production}$ de la cité.
3. **Victoires scénarisées** : Écran de fin de partie avec statistiques graphiques récapitulatives.
