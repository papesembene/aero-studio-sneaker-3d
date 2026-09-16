# Aero Studio — Sneaker 3D

Expérience produit immersive réalisée avec Vue 3 et Three.js. La scène utilise un modèle GLB, un éclairage HDRI, des animations au scroll, des interactions souris/tactiles et une mise en page responsive.

## Lancer le projet

```bash
npm install
npm run dev
```

## Préparer la production

```bash
npm run build
```

Le projet est compatible avec Vercel : importer le dépôt GitHub, laisser le framework **Vite** être détecté automatiquement, puis utiliser `npm run build` comme commande de build.

Le modèle 3D et les fichiers audio sont servis depuis `public/`. La licence du modèle est indiquée dans [public/ASSET-LICENSES.md](public/ASSET-LICENSES.md).
