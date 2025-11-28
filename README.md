# Portfolio DevOps - Kevin Monvoisin

Portfolio personnel développé avec Vue.js 3, TypeScript et Tailwind CSS. Le contenu est géré via des fichiers Markdown pour une maintenance facile.

## 🚀 Technologies

- **Vue.js 3** - Framework JavaScript progressif
- **TypeScript** - Typage statique
- **Vite** - Build tool rapide
- **Tailwind CSS** - Framework CSS utility-first
- **Markdown-it** - Parser Markdown

## 📁 Structure du projet

```
monvoisin-kevin.fr/
├── public/
│   ├── content/          # Fichiers Markdown
│   │   ├── about.md
│   │   ├── experience.md
│   │   ├── education.md
│   │   ├── certifications.md
│   │   └── projects.md
│   └── profile.jpg       # Photo de profil
├── src/
│   ├── components/       # Composants Vue
│   │   ├── Header.vue
│   │   ├── Section.vue
│   │   └── Footer.vue
│   ├── composables/      # Composables Vue
│   │   └── useMarkdown.ts
│   ├── App.vue
│   ├── main.ts
│   └── style.css
├── package.json
├── tailwind.config.js
├── tsconfig.json
└── vite.config.ts
```

## 🎨 Personnalisation

### Modifier le contenu

Tous les contenus sont dans `public/content/` au format Markdown. Éditez simplement les fichiers `.md` pour mettre à jour votre portfolio.

### Ajouter votre photo

Placez votre photo de profil dans `public/profile.jpg`

### Modifier les couleurs

Les couleurs DevOps sont définies dans `tailwind.config.js`:

```javascript
colors: {
  devops: {
    blue: '#0066CC',
    darkblue: '#003366',
    green: '#00B894',
    darkgreen: '#00856A',
    gray: '#2D3436',
    lightgray: '#636E72',
  },
}
```

### Modifier les liens sociaux

Éditez le composant `Footer.vue` pour changer les liens vers vos profils sociaux.

## 🛠️ Installation

```bash
# Installer les dépendances
npm install

# Lancer le serveur de développement
npm run dev

# Build pour la production
npm run build

# Preview du build de production
npm run preview
```

## 📝 Ajouter une nouvelle section

1. Créez un fichier Markdown dans `public/content/`
2. Ajoutez la section dans `App.vue`:

```vue
<Section 
  id="nouvelle-section"
  title="Titre de la section"
  markdown-path="/content/nouvelle-section.md"
  bg-color="bg-white"
/>
```

3. Ajoutez le lien dans le menu du `Header.vue`:

```javascript
const sections = [
  // ... autres sections
  { id: 'nouvelle-section', label: 'Nouvelle Section' }
]
```

## 🚢 Déploiement

### Netlify / Vercel

1. Connectez votre repository GitHub
2. Configuration de build:
   - Build command: `npm run build`
   - Publish directory: `dist`

### GitHub Pages

```bash
npm run build
# Déployez le contenu du dossier dist/
```

### Docker

```dockerfile
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## 📄 Licence

MIT

## 👤 Auteur

**Kevin Monvoisin**
- GitHub: [@kevinmonvoisin](https://github.com/kevinmonvoisin)
- LinkedIn: [kevin-monvoisin](https://linkedin.com/in/kevin-monvoisin)
