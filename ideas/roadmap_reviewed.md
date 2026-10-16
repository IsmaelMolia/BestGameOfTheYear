# 🗺️ Feuille de Route : Civ-Like Pygame — Roadmap (Reviewed)

> **De l'idée initiale à un 4X complet et fonctionnel.**  
> Suivi des étapes clés, des jalons de développement et des leçons apprises.

---

## 📊 Avancement Global du Projet

```
[██████████████████████████████░░░░░░░░░░] ~70% Réalisé
```

| Phase | Intitulé | Statut | Progression |
| :---: | :--- | :---: | :---: |
| **0** | **Fondations & Architecture** | 🟢 Terminé | 100% |
| **1** | **Moteur Hexagonal & Caméra** | 🟢 Terminé | 100% |
| **2** | **Interface Utilisateur & Menus** | 🟢 Terminé | 100% |
| **3** | **Boucle de Jeu 4X Fondamentale** | 🟢 Terminé | 95% |
| **4** | **Gameplay Approfondi & Combat** | 🟡 En cours | 40% |
| **5** | **Pathfinding & Intelligence Artificielle** | 🔴 Planifié | 15% |
| **6** | **Optimisation, Polish & Rétrospective** | 🔴 Planifié | 20% |

---

## 📍 Jalons Détaillés (Milestones)

### 🧱 Phase 0 : Fondations & Architecture du Projet
*Objectif : Mettre en place un environnement sain, modulaire et évolutif.*

- [x] Créer l'environnement virtuel Python (`venv`) et configurer les dépendances.
- [ ] Mettre en place le versionnement **Git** avec un `.gitignore` propre (ignorer `__pycache__`, environnements, etc.).
- [x] Séparer rigoureusement les répertoires : `scripts/`, `assets/`, `ideas/`, `data/`.
- [x] Découper le projet en modules à responsabilité claire :
  - `game.py` : Chef d'orchestre et boucle principale (Game Loop à 60 FPS).
  - `constants.py` : Dictionnaire de couleurs, coûts, rendements et chemins.
  - `map.py`, `players.py`, `units.py`, `ui.py`, `saves_handler.py`.
- [x] Mettre en place un système de constantes globales partagées pour l'équilibrage.

---

### 📐 Phase 1 : Moteur Hexagonal, Rendu & Caméra
*Objectif : Donner vie au monde et maîtriser l'affichage spatial 2D.*

- [x] Comprendre et implémenter le système de coordonnées hexagonales **odd-r** (ressource : [Red Blob Games](https://www.redblobgames.com/grids/hexagons/)).
- [x] Implémenter la conversion bidirectionnelle : Grille $\leftrightarrow$ Pixels écran.
- [x] Détection précise du survol souris par calcul de distance euclidienne.
- [x] Génération de monde en cercles concentriques (`layers`) autour d'une case centrale.
- [x] Algorithme de génération organique par graines (*seed-based biome clustering*).
- [x] Caméra fluide : Déplacement libre (`Z, Q, S, D`), recentrage (`C`), focus automatique sur la capitale.
- [x] Système de zoom continu à la molette avec recalcul proportionnel des tailles d'hexagones et sprites.
- [x] Modes d'affichage de la carte :
  - Mode Empire (`E`) : Frontières territoriales.
  - Mode Ressources (`R`) : Pictogrammes des ressources naturelles.
  - Brouillard de guerre (`TAB`).

---

### 🎨 Phase 2 : Interface Utilisateur & Écrans (UI Maison)
*Objectif : Créer un framework GUI interactif sans aucune bibliothèque externe.*

- [x] Créer la classe `Button` : Détection du survol, effet de grossissement, clics gauche/droit, coins arrondis.
- [x] Gérer les champs de saisie interactifs (`inputable`) pour taper texte et chiffres.
- [x] Gérer les transitions de pages via `PageHandler` :
  - **Page 0** : Écran d'accueil (*Start Menu*).
  - **Page 1** : Options & Affichage (*Settings* : résolution, échelle UI).
  - **Page 2** : Choix Nouvelle partie / Continuer.
  - **Page 3** : Gestionnaire des 10 slots de sauvegarde avec suppression ciblée.
  - **Page 4** : Paramétrage avancé de la partie (sliders/pourcentages de biomes, joueurs, brouillard).
  - **Page 5** : Écran de jeu principal (*Game View*) avec fond marin animé.
- [x] Concevoir le système de panneaux contextuels rétractables (*Side Panels*) :
  - `TILE` : Informations sur le biome et propriétaire.
  - `CURRENT_UNIT` & `CURRENT_CITY` : Fiches d'état.
  - `BUILD` : Grille de sélection des bâtiments avec masquage des cases interdites.
  - `INSPECT` : Détails encyclopédiques.
  - `PAUSE` & `COMMANDS` : Aide et réglages en partie.
- [x] Intégrer des curseurs personnalisés adaptés au contexte d'action.

---

### 🌾 Phase 3 : Boucle de Jeu 4X Fondamentale
*Objectif : Relier la carte, les unités, l'économie et la persistance des données.*

- [x] Implémenter l'unité de départ : **Le Colon (`Settler`)**.
- [x] Fondation de ville : Le colon s'établit, fonde la cité et annexe les 6 tuiles adjacentes.
- [x] Système d'économie territorialisé :
  - Rendements de base selon le biome (nourriture, bois, pierre, or).
  - Bâtiments productifs (ex: Ferme `+3` nourriture/tour).
- [x] Croissance et expansion territoriale dynamique :
  - Accumulation de nourriture au fil des tours.
  - Annexion automatique d'une nouvelle case frontalière dès que le seuil est franchi.
- [x] Mécanique de fin de tour (`Espace`) :
  - Récolte des ressources dans l'inventaire de l'empire.
  - Réinitialisation des points de mouvement (PM) des unités.
- [x] Persistance JSON complète via `saves_handler.py` : Sauvegarde et rechargement exact de l'état de la carte, des joueurs et des unités.

---

### ⚔️ Phase 4 : Gameplay Approfondi & Stratégie Militaire
*Objectif : Transformer le prototype en un jeu tactique stimulant.*

- [ ] **Menu Cité & Production d'unités** :
  - Interface dédiée pour recruter des unités avec un coût en ressources et un nombre de tours requis.
  - Production de nouveaux colons, guerriers, archers, etc.
- [ ] **Diversité des unités** :
  - Guerrier (corps-à-corps résistant).
  - Archer (attaque à distance sur 2 cases, fragile).
  - Cavalier (vitesse élevée, idéal pour l'exploration).
- [ ] **Améliorations des combats** :
  - Prise en compte du terrain (bonus défensif en forêt ou sur colline).
  - Points de vie visibles sous forme de jauge au-dessus des unités.
- [ ] **Conditions de victoire & défaite** :
  - Victoire militaire : Éliminer tous les empires adverses.
  - Victoire économique/territoriale : Atteindre un seuil critique d'expansion ou de richesse.
  - Écran de défaite si la capitale est conquise.

---

### 🧠 Phase 5 : Pathfinding & Intelligence Artificielle
*Objectif : Donner vie aux empires concurrents.*

- [ ] **Algorithme de Pathfinding A\*** adapté à la grille hexagonale :
  - Calcul du chemin le plus court prenant en compte les coûts de terrain (`TILES_MOVEMENT_COST`).
  - Prévisualisation du tracé lors du déplacement d'une unité.
- [ ] **IA de base (`AI_Player`)** :
  - Décision d'établissement du premier colon sur la meilleure tuile disponible.
  - Gestion économique : Dépenser ses ressources pour poser des fermes/bâtiments dès que possible.
  - Recrutement d'unités de défense et patrouille autour du territoire.
  - Déclaration d'hostilités et attaque des unités ennemies à portée.

---

### 🚀 Phase 6 : Optimisation, Polish & Rétrospective
*Objectif : Assurer la fluidité, la robustesse et capitaliser sur l'expérience.*

- [ ] **Optimisation du rendu (Culling)** : Ne dessiner que les hexagones actuellement visibles dans le champ de la caméra pour supporter de très grandes cartes sans baisse de FPS.
- [ ] **Optimisation de la génération de carte** : Accélérer `seed_generation_v3()` pour réduire le temps de chargement des mondes géants.
- [ ] **Chasse aux bugs & Cas limites** :
  - Tester les superpositions d'unités, les divisions par zéro sur les jauges, etc.
- [ ] **Habillage sonore & Visuel** :
  - Intégration de bruitages (pose de bâtiment, combat, clic).
  - Micro-animations de bannières et de sélection.
- [ ] **Documentation & Archivage** :
  - Finaliser les schémas d'architecture et les commentaires de code.
  - Rédiger un bilan de fin de projet pour consolider les acquis.

---

## 💡 Leçons Apprises & Retours d'Expérience (Post-Mortem)

> *« Faire de grosses erreurs d'organisation et se rendre compte après plusieurs jours qu'il faut tout recommencer depuis le début, au moins 3 fois... »* — Note authentique du carnet de bord.

1. **La conception de l'UI avant le code** : Recommencer l'interface trois fois a démontré l'importance de séparer la logique événementielle (clics, états) de l'affichage pur. La création de classes dédiées (`Button`, `ButtonHandler`, `PageHandler`) a été la solution salvatrice.
2. **La règle d'or des coordonnées hexagonales** : Travailler avec des coordonnées écran sans passer par un système abstrait solide (axial ou odd-r) est un piège. S'appuyer sur la théorie éprouvée de *Red Blob Games* a tout débloqué.
3. **L'importance de la documentation continue** : Oublier comment fonctionne son propre code après une interruption est le problème n°1 de tout projet ambitieux. Documenter les structures au fur et à mesure (comme ce fichier et `plans_reviewed.md`) garantit la survie du projet.
