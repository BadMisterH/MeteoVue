# 🌦️ MeteoVue

Une application météo moderne et élégante construite avec Vue.js 3, offrant des prévisions météorologiques en temps réel avec une interface utilisateur intuitive et responsive.

## ✨ Fonctionnalités

- 🔍 **Recherche de ville** : Recherchez la météo de n'importe quelle ville dans le monde
- 🌡️ **Météo actuelle** : Température, ressenti, humidité, vitesse du vent
- ⏰ **Prévisions horaires** : Prévisions détaillées pour aujourd'hui (toutes les 3h)
- 📅 **Prévisions 5 jours** : Aperçu météorologique sur 5 jours
- 🎨 **Interface moderne** : Design glassmorphism avec animations fluides
- 📱 **Responsive** : Optimisé pour desktop, tablette et mobile
- 🌍 **Multilingue** : Interface et données en français
- 🔄 **Gestion d'erreurs** : Messages d'erreur informatifs et retry automatique

## 🛠️ Technologies utilisées

- **Vue.js 3** avec Composition API
- **Vite** pour le bundling ultra-rapide
- **Tailwind CSS** pour le styling
- **OpenWeatherMap API** pour les données météo
- **JavaScript ES6+**

## 📋 Prérequis

- **Node.js** version 18.x ou 20.x (⚠️ **Pas compatible avec Node.js 22.x**)
- **npm** ou **yarn**
- Une clé API OpenWeatherMap (gratuite)

## 🚀 Installation

1. **Clonez le repository**
   ```bash
   git clone [url-du-repo]
   cd meteoVue
   ```

2. **Vérifiez votre version de Node.js**
   ```bash
   node --version
   ```
   Si vous utilisez Node.js 22.x, passez à une version compatible :
   ```bash
   nvm use 20  # ou nvm use 18
   ```

3. **Installez les dépendances**
   ```bash
   npm install
   ```

4. **Configuration de l'API**
   - Créez un compte sur [OpenWeatherMap](https://openweathermap.org/api)
   - Obtenez votre clé API gratuite
   - Créez un fichier `.env` à la racine du projet :
   ```env
   VITE_WEATHER_API_KEY=votre_cle_api_ici
   ```

5. **Lancez le serveur de développement**
   ```bash
   npm run dev
   ```

6. **Ouvrez votre navigateur** sur `http://localhost:5173`

## 🏗️ Structure du projet

```
meteoVue/
├── src/
│   ├── components/
│   │   ├── Search.vue          # Composant de recherche
│   │   └── Today.vue           # Composant prévisions du jour
│   ├── assets/                 # Images et assets
│   ├── App.vue                 # Composant principal
│   ├── main.js                 # Point d'entrée
│   └── style.css              # Styles globaux
├── public/
│   └── vite.svg               # Favicon
├── .env                       # Variables d'environnement
├── package.json
├── vite.config.js
└── README.md
```

## 🔧 Scripts disponibles

```bash
npm run dev          # Serveur de développement
npm run build        # Build de production
npm run preview      # Aperçu du build de production
```

## 🌍 APIs utilisées

### OpenWeatherMap API
- **Current Weather API** : `api.openweathermap.org/data/2.5/weather`
- **5-Day Forecast API** : `api.openweathermap.org/data/2.5/forecast`

**Paramètres utilisés** :
- `units=metric` (Températures en Celsius)
- `lang=fr` (Descriptions en français)
- Icônes météo haute définition (`@4x.png`)

## 🎨 Design System

### Palette de couleurs
- **Arrière-plan** : Dégradé bleu (`from-blue-400 via-blue-500 to-blue-600`)
- **Cartes** : Glassmorphism (`bg-white/15 backdrop-blur-md`)
- **Texte** : Blanc avec différents niveaux d'opacité
- **Erreurs** : Rouge avec transparence (`bg-red-500/20`)

### Animations
- **Fade-in** : Apparition en fondu des éléments
- **Bounce-slow** : Animation flottante pour les icônes météo
- **Hover effects** : Transformations au survol (`scale`, `blur`)
- **Loading spinner** : Animation de chargement

### Typography
- **Titres principaux** : 4xl-6xl, font-bold
- **Températures** : 5xl-6xl, font-bold
- **Descriptions** : Texte capitalize, opacité réduite

## 📱 Interface utilisateur

### États de l'application
1. **Chargement** : Spinner animé avec message
2. **Contenu normal** : Données météo complètes
3. **Erreur** : Message d'erreur avec bouton de retry
4. **Pas de données** : État vide informatif

### Sections principales
1. **Barre de recherche** : Recherche en temps réel
2. **Météo actuelle** : Carte principale avec toutes les infos
3. **Prévisions horaires** : Scroll horizontal avec toutes les prévisions du jour
4. **Prévisions 5 jours** : Vue compacte verticale

## 🔍 Fonctionnalités avancées

### Gestion des erreurs
- Validation de la clé API
- Gestion des villes introuvables (404)
- Retry automatique avec Paris par défaut
- Messages d'erreur localisés

### Performance
- Appels API parallèles (`Promise.all`)
- Computed properties optimisées
- Évitement des appels redondants
- Images optimisées et lazy loading

### Accessibilité
- Alt text pour toutes les images
- Contrastes respectés
- Navigation au clavier
- Messages d'état pour les lecteurs d'écran

## 🚨 Problèmes connus et solutions

### Node.js 22.x incompatible
**Erreur** : `SyntaxError: The requested module './dep-lCKrEJQm.js' does not provide an export named '__commonJS'`

**Solution** :
```bash
nvm use 20  # ou nvm use 18
npm run dev
```

### Clé API manquante
**Erreur** : "Clé API OpenWeatherMap manquante"

**Solution** : Vérifiez votre fichier `.env` et que la variable commence par `VITE_`

## 🤝 Contribution

1. Fork le projet
2. Créez une branche feature (`git checkout -b feature/AmazingFeature`)
3. Commit vos changements (`git commit -m 'Add some AmazingFeature'`)
4. Push vers la branche (`git push origin feature/AmazingFeature`)
5. Ouvrez une Pull Request

## 📄 Licence

Ce projet est sous licence MIT. Voir le fichier `LICENSE` pour plus de détails.

## 👨‍💻 Auteur

**Badro** - Développeur Frontend passionné

## 🙏 Remerciements

- [OpenWeatherMap](https://openweathermap.org/) pour l'API météo gratuite
- [Vue.js](https://vuejs.org/) pour le framework réactif
- [Tailwind CSS](https://tailwindcss.com/) pour les utilitaires CSS
- [Vite](https://vitejs.dev/) pour l'excellent tooling de développement

---

⭐ N'hésitez pas à donner une étoile si ce projet vous a plu !
