# Simple Bien-être

Simple Bien-être est une petite application web mobile-first (Svelte + Vite) qui génère rapidement des séances (1/2/4 mouvements équilibrés push/pull/squat/hinge), propose des exercices de respiration et des méthodes de fumage, et permet de sauvegarder des séances localement via `localStorage`. Technologies : Svelte, Vite, CSS3.
# Simple Bien-Ãªtre ðŸ‹ï¸

Une application web mobile-first pour gÃ©nÃ©rer des sÃ©ances d'entraÃ®nement personnalisÃ©es, des exercices de respiration et des mÃ©thodes de consommation.

## ðŸŒŸ FonctionnalitÃ©s

### GÃ©nÃ©rateur d'Exercices
- **4 mouvements Ã©quilibrÃ©s** : Push, Pull, Squat, Hinge
- **Options flexibles** : 4 mouvements, 2 mouvements, ou 1 mouvement
- **SÃ©lection intelligente** : Aucune rÃ©pÃ©tition d'exercice consÃ©cutive
- **Ã‰pingles personnalisÃ©es** : Cliquez sur les cartes pour les verrouiller lors de la rÃ©gÃ©nÃ©ration
- **Sauvegarde de sÃ©ances** : Nommez et sauvegardez vos sÃ©ances prÃ©fÃ©rÃ©es

### Respiration & Conscience
- 6 exercices de respiration guidÃ©s
- Box Breathing, Wim Hof, 4-7-8 Breathing, et plus...

### MÃ©thodes de Fumage
- 6 mÃ©thodes diffÃ©rentes avec descriptions visuelles
- Guide complet des techniques

### Favoris
- Sauvegardez vos sÃ©ances d'entraÃ®nement prÃ©fÃ©rÃ©es
- AccÃ©dez rapidement Ã  vos favoris depuis le coin supÃ©rieur droit
- Persistance avec localStorage

## ðŸŽ¨ Design

- **ThÃ¨me Mauve** : Interface moderne et relaxante
- **Mobile-First** : OptimisÃ© pour tous les appareils
- **Mode Sombre** : Facile pour les yeux
- **Interface FranÃ§aise** : EntiÃ¨rement traduite

## ðŸš€ DÃ©marrage Rapide

### PrÃ©requis
- Node.js 16+ 
- npm ou yarn

### Installation

```bash
# Cloner le repository
git clone https://github.com/Aleksios22/SimpleBienEtre.git
cd SimpleBienEtre

# Installer les dÃ©pendances
npm install

# Lancer le serveur de dÃ©veloppement
npm run dev
```

Le site sera accessible Ã  `http://localhost:5174`

### Build pour la Production

```bash
npm run build
```

Les fichiers compilÃ©s seront dans le dossier `dist/`

## ðŸ“± Structure du Projet

```
src/
â”œâ”€â”€ App.svelte              # Composant principal
â”œâ”€â”€ lib/
â”‚   â”œâ”€â”€ exercises.js        # Base de donnÃ©es des 27 exercices
â”‚   â”œâ”€â”€ exercises-2-movements.js  # Base de donnÃ©es (2 mouvements)
â”‚   â”œâ”€â”€ exercises-1-movement.js   # Base de donnÃ©es (1 mouvement)
â”‚   â”œâ”€â”€ breathing.js        # Exercices de respiration
â”‚   â””â”€â”€ smoking.js          # MÃ©thodes de fumage
â”œâ”€â”€ assets/images/
â”‚   â”œâ”€â”€ exercises/          # Images des exercices
â”‚   â”œâ”€â”€ breathing/          # Images des respirations
â”‚   â””â”€â”€ smoking/            # Images des mÃ©thodes
â””â”€â”€ main.js                 # Point d'entrÃ©e

```

## ðŸ”§ Technologies

- **Svelte** : Framework rÃ©actif lÃ©ger
- **Vite** : Build tool ultra-rapide
- **localStorage API** : Persistance des donnÃ©es
- **CSS3** : Styling responsive

## ðŸ’¾ Persistence des DonnÃ©es

L'application utilise `localStorage` pour:
- Sauvegardez vos sÃ©ances d'entraÃ®nement prÃ©fÃ©rÃ©es
- Les donnÃ©es persistent aprÃ¨s fermeture du navigateur

## ðŸ“Š CatÃ©gories d'Exercices

### Push (10 exercices)
Push-ups, Mountain Climber, Shoulder Press, Chandelle, etc.

### Pull (4 exercices)
Tractions, Body Rows, Front Lever GroupÃ©, Rowing Barre

### Squat (8 exercices)
Squats, Lunges, Squats Explosifs, Squats Pistolet, etc.

### Hinge (6 exercices)
Deadlift, Crunch VÃ©lo, Kettlebell Swing, Power Clean, etc.

## ðŸŒ DÃ©ploiement

L'application peut Ãªtre dÃ©ployÃ©e facilement sur:
- **Netlify** : `npm run build` â†’ drag & drop le dossier `dist/`
- **Vercel** : Push vers GitHub et connectez le repo
- **GitHub Pages** : Configurez les Actions GitHub

## ðŸ¤ Contribution

Les contributions sont bienvenues! N'hÃ©sitez pas Ã :
1. Fork le projet
2. CrÃ©er une branche (`git checkout -b feature/AmazingFeature`)
3. Commit vos changements (`git commit -m 'Add some AmazingFeature'`)
4. Push vers la branche (`git push origin feature/AmazingFeature`)
5. Ouvrir une Pull Request

## ðŸ“ License

Ce projet est sous la License MIT. Voir le fichier `LICENSE` pour plus de dÃ©tails.

## ðŸ‘¨â€ðŸ’» Auteur

**Aleksios22** - [GitHub](https://github.com/Aleksios22)

## ðŸ™ Remerciements

- Svelte et Vite pour les outils exceptionnels
- Tous les contributeurs et utilisateurs


