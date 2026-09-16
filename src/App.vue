<template>
  <main
    id="gamme"
    class="experience"
    :class="{
      'profil-ouvert': profilActif,
      'mode-gamme-complete': modeGammeComplete,
    }"
    :style="{ '--couleur-saveur': varianteActive.couleur }"
  >
    <header class="entete-site" aria-label="Navigation principale">
      <div class="logo-ciao" aria-label="Aero Studio">AERO<br /><span>STUDIO</span></div>
      <div class="actions-site">
        <span class="menu-site"><i></i><i></i><i></i><i></i> MENU</span>
        <a href="#contact" class="contact-site">CONTACT</a>
      </div>
    </header>
    <div
      ref="canvasContainer"
      class="canvas-container"
      role="img"
      :aria-label="`Collection de sneakers : ${varianteActive.nom} sélectionnée. Cliquez sur la sneaker centrale pour voir les détails.`"
    ></div>
    <div class="hud-superieur" aria-hidden="true">
      <span class="hud-ligne">
        <i :style="{ left: `${progressionHud}%` }"></i>
      </span>
    </div>
    <div v-if="!profilActif" class="halo-gamme" aria-hidden="true"></div>
    <div v-if="chargement && !erreur" class="chargement-premium" role="status" aria-live="polite">
      <span class="chargement-rond" aria-hidden="true"></span>
      <span class="chargement-titre">AERO STUDIO</span>
      <span class="chargement-label">PRÉPARATION DU SHOWROOM 3D</span>
      <span class="chargement-barre" aria-hidden="true"><i></i></span>
    </div>
    <p v-if="erreur" class="message-chargement" role="alert">{{ erreur }}</p>
    <button
      class="bouton-son"
      type="button"
      :aria-pressed="sonActif"
      :aria-label="sonActif ? 'Couper les sons' : 'Activer les sons'"
      @click="basculerSon"
    >
      <span>{{ sonActif ? "ON" : "OFF" }}</span>
      <span class="barres-son" aria-hidden="true">
        <i v-for="hauteur in [5, 9, 7, 4]" :key="hauteur" :style="{ height: `${hauteur}px` }"></i>
      </span>
    </button>
    <section
      v-if="!chargement && !erreur"
      class="commandes-gamme"
      aria-label="Choisir une sneaker"
      @keydown.left.prevent="changerCanette(-1)"
      @keydown.right.prevent="changerCanette(1)"
    >
      <button
        class="zone-produit"
        type="button"
        :aria-label="`Voir les détails de la sneaker ${varianteActive.nom}`"
        @click.stop="ouvrirProfil"
      ></button>
      <button
        v-for="sens in [-1, 1]"
        :key="sens"
        type="button"
        class="fleche-gamme"
        :class="sens === -1 ? 'fleche-gauche' : 'fleche-droite'"
        :aria-label="sens === -1 ? 'Sneaker précédente' : 'Sneaker suivante'"
        @click="changerCanette(sens)"
      >
        <svg viewBox="0 0 20 36" aria-hidden="true" :class="{ 'vers-droite': sens === 1 }">
          <circle cx="14" cy="4" r="2" opacity="0.35" />
          <circle cx="8" cy="11" r="2" opacity="0.65" />
          <circle cx="2" cy="18" r="2" />
          <circle cx="8" cy="25" r="2" opacity="0.65" />
          <circle cx="14" cy="32" r="2" opacity="0.35" />
        </svg>
      </button>
      <h1 class="nom-produit" aria-live="polite" aria-atomic="true">
        <span v-for="ligne in varianteActive.titre" :key="ligne">{{ ligne }}</span>
      </h1>
      <p class="indice-clic" aria-hidden="true">CLIQUER SUR LA SNEAKER POUR DÉCOUVRIR</p>
    </section>
    <div v-if="!chargement && !erreur && !profilActif" class="pagination-gamme" aria-hidden="true">
      <div class="pagination-ligne">
        <span
          class="pagination-point"
          :style="{ left: `${(indexActif / (variantes.length - 1)) * 100}%` }"
        ></span>
      </div>
      <span class="scroll-discover">SCROLLER POUR DÉCOUVRIR</span>
    </div>
    <section v-if="profilActif && !modeGammeComplete" class="profil-produit" aria-live="polite">
      <div class="profil-carte">
        <transition name="profil-texte" mode="out-in">
          <div :key="profilEtape" class="profil-texte">
            <div v-if="contenuProfil.surtitre" class="profil-subhead">
              <span class="profil-subhead-icon">×</span>
              <span class="profil-subhead-label">{{ contenuProfil.surtitre }}</span>
            </div>
            <h2>
              <span v-for="ligne in contenuProfil.titre" :key="ligne">{{ ligne }}</span>
            </h2>
            <p>{{ contenuProfil.description }}</p>
          </div>
        </transition>
        <div v-if="profilEtape === 0" class="profil-infos">
          <span v-for="info in contenuProfil.infos" :key="info">{{ info }}</span>
        </div>
        <p class="profil-scroll">
          {{ profilEtape === profilEtapes.length - 1 ? "FIN DE LA FICHE · ÉCHAP POUR REVENIR" : "SCROLLER / GLISSER POUR LIRE LA SUITE" }}
        </p>
        <button type="button" class="bouton-fermer" @click="fermerProfil">
          <span aria-hidden="true">×</span> RETOUR À LA GAMME
        </button>
      </div>
      <nav class="profil-navigation" aria-label="Lire les informations de la sneaker">
        <button
          v-for="(icone, index) in iconesBenefices"
          :key="icone.nom"
          type="button"
          :class="{ actif: profilEtape === index + 1 }"
          :aria-label="icone.nom"
          :aria-current="profilEtape === index + 1 ? 'step' : undefined"
          @click="allerProfilEtape(index + 1)"
        >
          <svg viewBox="0 0 32 32" aria-hidden="true">
            <path :d="icone.path" />
          </svg>
        </button>
      </nav>
      <div v-if="profilEtape > 0" class="points-chaussure" aria-label="Points d’information">
        <button
          v-for="(point, index) in pointsSneaker"
          :key="point.nom"
          type="button"
          class="point-chaussure"
          :class="{ actif: profilEtape === index + 1 }"
          :style="{ left: point.left, top: point.top }"
          :aria-label="point.nom"
          @click="allerProfilEtape(index + 1)"
        >{{ index + 1 }}</button>
      </div>
    </section>
    <p v-if="modeGammeComplete" class="fin-gamme" aria-live="polite">
      TOUTE LA COLLECTION · SCROLLER VERS LE HAUT POUR REVENIR
    </p>
  </main>
</template>

<script setup>
import { computed, ref, onMounted, onBeforeUnmount } from "vue";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { HDRLoader } from "three/examples/jsm/loaders/HDRLoader.js";
const canvasContainer = ref(null);
const chargement = ref(true);
const erreur = ref("");
const sonActif = ref(true);
const profilActif = ref(false);

const fichiersSon = {
  change: "/audio/change.mp3",
  enter: "/audio/enter.mp3",
  benefits: "/audio/benefits.mp3",
  click: "/audio/click.mp3",
};
let contexteAudio = null;
let sons = {};
let sonDeverrouille = false;
let prechargementSons = null;
const volumeSon = 0.5;

function initialiserAudio() {
  if (!contexteAudio) {
    const AudioContextClasse = window.AudioContext || window.webkitAudioContext;
    if (!AudioContextClasse) return null;
    contexteAudio = new AudioContextClasse();
  }
  return contexteAudio;
}

async function prechargerSons() {
  const contexte = initialiserAudio();
  if (!contexte) return;

  const entrées = Object.entries(fichiersSon);
  await Promise.all(entrées.map(async ([nom, chemin]) => {
    try {
      const réponse = await fetch(chemin);
      if (!réponse.ok) throw new Error(`${réponse.status} ${chemin}`);
      const données = await réponse.arrayBuffer();
      sons[nom] = await contexte.decodeAudioData(données);
    } catch (cause) {
      console.warn(`[ciaoSound] échec chargement ${nom}`, cause);
    }
  }));
}

function deverrouillerAudio() {
  const contexte = initialiserAudio();
  if (!contexte) return;
  sonDeverrouille = true;
  if (contexte.state === "suspended") void contexte.resume();
}

function jouerSon(nom, niveau = 1) {
  if (!sonActif.value || !sonDeverrouille || !contexteAudio || !sons[nom]) return;
  const source = contexteAudio.createBufferSource();
  const gain = contexteAudio.createGain();
  source.buffer = sons[nom];
  gain.gain.value = volumeSon * niveau;
  source.connect(gain).connect(contexteAudio.destination);
  source.start(0);
}

function basculerSon() {
  sonActif.value = !sonActif.value;
  if (sonActif.value) {
    deverrouillerAudio();
    jouerSon("click", 0.6);
  }
}

// Variantes de la collection : le même modèle 3D reçoit des teintes différentes.
const variantes = [
  { nom: "Porcelain", titre: ["PORCELAIN", "WHITE"], couleur: "#e5e0d5", description: "Une silhouette ivoire lumineuse, sobre et facile à associer." },
  { nom: "Graphite", titre: ["GRAPHITE", "BLACK"], couleur: "#34363b", description: "Un noir graphite mat, relevé par les détails techniques de la chaussure." },
  { nom: "Terracotta", titre: ["TERRA", "COTTA"], couleur: "#a95e4b", description: "Une teinte argile chaleureuse qui donne du caractère à la silhouette." },
  { nom: "Moss", titre: ["MOSS", "GREEN"], couleur: "#68725f", description: "Un vert mousse naturel, inspiré des matières et des chemins en plein air." },
  { nom: "Midnight", titre: ["MIDNIGHT", "BLUE"], couleur: "#27384d", description: "Un bleu nuit profond, discret à l’ombre et intense sous la lumière." },
  { nom: "Plum", titre: ["PLUM", "DUSK"], couleur: "#704f65", description: "Un prune doux et contemporain, équilibré par les textures claires." },
];

// La cible change au clic ; la position réelle la rejoint progressivement.
let positionCible = 0;
let positionCarrousel = 0;
const indexActif = ref(0);
const varianteActive = computed(() => variantes[indexActif.value]);
const profilEtape = ref(0);
const profilEtapes = [
  null,
  {
    titre: ["Matière", "respirante"],
    surtitre: "TEXTILE TECHNIQUE",
    description: "Une maille légère favorise la circulation de l’air et garde le pied confortable pendant l’effort.",
    infos: ["MAILLE 3D", "RESPIRABILITÉ"],
  },
  {
    titre: ["Amorti", "progressif"],
    surtitre: "MOUSSE CONFORT",
    description: "La semelle intermédiaire absorbe les chocs à chaque pas tout en conservant un retour d’énergie naturel.",
    infos: ["ABSORPTION", "RETOUR D’ÉNERGIE"],
  },
  {
    titre: ["Semelle", "adhérente"],
    surtitre: "CAOUTCHOUC GRIP",
    description: "Le dessin de la semelle améliore l’adhérence sur les surfaces urbaines et accompagne les changements de direction.",
    infos: ["GRIP URBAIN", "FLEXIBILITÉ"],
  },
  {
    titre: ["Conçue", "durablement"],
    surtitre: "FABRICATION SOIGNÉE",
    description: "Chaque panneau est assemblé pour limiter les coutures inutiles et prolonger la durée de vie de la sneaker.",
    infos: ["PIÈCES RENFORCÉES", "USAGE QUOTIDIEN"],
  },
];
const iconesBenefices = [
  { nom: "Lire la matière", path: "M16 29c-1-5 0-9 4-12 3-2 5-6 4-11-5 0-9 2-11 6-2 4 0 9 3 11-3 0-6 2-8 6m7-9c-2-3-3-6-2-9" },
  { nom: "Lire l’amorti", path: "M16 2C14 7 8 11 8 18a8 8 0 0 0 16 0c0-7-6-11-8-16Zm0 23a6 6 0 0 1-6-6c0-3 2-6 6-10 4 4 6 7 6 10a6 6 0 0 1-6 6Z" },
  { nom: "Lire la semelle", path: "M10 6c-5 4-6 11-2 17 4 5 11 4 15-1 4-6 1-14-5-17-3-1-5-1-8 1Zm2 2c2-1 4-1 6 0-4 2-6 6-6 12-3-4-3-9 0-12Z" },
  { nom: "Lire la fabrication", path: "M15 29c1-8 0-14-4-19m4 8c3-4 7-5 11-5-2 5-5 8-10 9m-2-6C10 13 6 9 2 10c1 5 5 8 11 9m2-4c0-5 2-9 6-12 1 5-1 10-5 13" },
];
const pointsSneaker = [
  { nom: "Matière respirante", left: "56%", top: "38%" },
  { nom: "Amorti progressif", left: "63%", top: "55%" },
  { nom: "Semelle adhérente", left: "49%", top: "73%" },
  { nom: "Fabrication soignée", left: "42%", top: "46%" },
];
const contenuProfil = computed(() => profilEtape.value === 0
  ? {
      titre: varianteActive.value.titre,
      description: varianteActive.value.description,
      surtitre: "",
      infos: ["GLB PBR", "SEMELLE GRIP", "CONFORT QUOTIDIEN"],
    }
  : profilEtapes[profilEtape.value]);
const progressionHud = computed(() => profilActif.value
  ? Math.min(94, 8 + profilEtape.value * 18 + (modeGammeComplete.value ? 20 : 0))
  : 0);
const modeGammeComplete = ref(false);
let progressionProfil = 0;
let cibleProfil = 0;
let progressionEtape = 0;
let progressionGammeComplete = 0;
let cliquerCanette;
let fermerAvecEchap;
let debutTouchProfil = null;
let debutTouchGamme = null;
let verrouillageEtape = false;
let minuteurEtape;

function ouvrirProfil() {
  if (chargement.value || erreur.value || profilActif.value) return;
  deverrouillerAudio();
  jouerSon("enter", 0.9);
  profilEtape.value = 0;
  progressionEtape = 0;
  modeGammeComplete.value = false;
  progressionGammeComplete = 0;
  rotationManuelle = 0;
  verrouillageEtape = false;
  cibleProfil = 1;
  profilActif.value = true;
}

function fermerProfil() {
  deverrouillerAudio();
  jouerSon("click", 0.7);
  cibleProfil = 0;
  profilEtape.value = 0;
  progressionEtape = 0;
  modeGammeComplete.value = false;
  progressionGammeComplete = 0;
  rotationManuelle = 0;
  verrouillageEtape = false;
  profilActif.value = false;
}

function defilerProfil(evenement) {
  if (!profilActif.value || chargement.value || erreur.value) return;
  if (Math.abs(evenement.deltaY) < 4) return;
  evenement.preventDefault();
  if (verrouillageEtape) return;
  const sens = evenement.deltaY > 0 ? 1 : -1;
  if (modeGammeComplete.value) {
    if (sens < 0) {
      modeGammeComplete.value = false;
      progressionGammeComplete = 0;
      jouerSon("benefits", 0.85);
    } else {
      // Après l'écran « Toute la gamme », un dernier scroll termine la
      // boucle et revient à l'accueil, comme demandé pour cette version.
      fermerProfil();
    }
    return;
  }
  if (sens > 0 && profilEtape.value === profilEtapes.length - 1) {
    modeGammeComplete.value = true;
    progressionGammeComplete = 0;
    jouerSon("benefits", 0.85);
    return;
  }
  if (sens < 0 && profilEtape.value === 0) {
    // Une fois revenu au premier écran du profil, continuer vers le haut
    // ramène à la présentation de la gamme, comme sur le site officiel.
    fermerProfil();
    return;
  }
  const prochaineEtape = Math.max(0, Math.min(profilEtapes.length - 1, profilEtape.value + sens));
  if (prochaineEtape === profilEtape.value) return;
  profilEtape.value = prochaineEtape;
  jouerSon("benefits", 0.85);
  verrouillageEtape = true;
  clearTimeout(minuteurEtape);
  // Une étape reste affichée : cela évite que plusieurs crans de molette
  // fassent sauter le texte avant que la sneaker ait fini sa rotation.
  minuteurEtape = setTimeout(() => { verrouillageEtape = false; }, 1300);
}

function allerProfilEtape(etape) {
  if (!profilActif.value || modeGammeComplete.value || etape < 1 || etape >= profilEtapes.length) return;
  if (etape === profilEtape.value) return;
  profilEtape.value = etape;
  jouerSon("benefits", 0.85);
  verrouillageEtape = true;
  clearTimeout(minuteurEtape);
  minuteurEtape = setTimeout(() => { verrouillageEtape = false; }, 650);
}

function commencerTouchProfil(evenement) {
  if (profilActif.value) debutTouchProfil = evenement.touches[0]?.clientY ?? null;
}

function terminerTouchProfil(evenement) {
  if (debutTouchProfil === null || !profilActif.value) return;
  const fin = evenement.changedTouches[0]?.clientY ?? debutTouchProfil;
  const delta = debutTouchProfil - fin;
  debutTouchProfil = null;
  if (Math.abs(delta) > 24) defilerProfil({ deltaY: delta, preventDefault() {} });
}

function commencerTouchGamme(evenement) {
  if (!profilActif.value) debutTouchGamme = evenement.touches[0]?.clientX ?? null;
}

function terminerTouchGamme(evenement) {
  if (debutTouchGamme === null || profilActif.value) return;
  const fin = evenement.changedTouches[0]?.clientX ?? debutTouchGamme;
  const delta = debutTouchGamme - fin;
  debutTouchGamme = null;
  if (Math.abs(delta) > 32) changerCanette(delta > 0 ? 1 : -1);
}

function changerCanette(sens) {
  if (chargement.value || erreur.value) return;
  deverrouillerAudio();
  positionCible += sens;
  jouerSon("change");
}

// Contrairement à %, cette fonction boucle aussi correctement vers la gauche.
function modulo(valeur, taille) {
  return ((valeur % taille) + taille) % taille;
}

let renderer;
let animationId;
let redimensionner;
let suivreSouris;
let commencerGlisse;
let poursuivreGlisse;
let terminerGlisse;
let sneakerGlissee = false;
let dernierePositionGlisse = 0;
let rotationManuelle = 0;
let estDemonte = false;
const ressources = new Set();

// Garder les ressources pour libérer leur mémoire quand on quitte la page.
function conserver(ressource) {
  if (estDemonte) ressource.dispose();
  else ressources.add(ressource);
  return ressource;
}

async function chargerModele(chemin) {
  const gltf = await new GLTFLoader().loadAsync(chemin);
  gltf.scene.traverse((objet) => {
    if (!objet.isMesh) return;
    conserver(objet.geometry);
    const materiaux = Array.isArray(objet.material) ? objet.material : [objet.material];
    materiaux.forEach(conserver);
  });
  return gltf.scene;
}

function dessinerMotifManga(contexte, variante, dos) {
  if (variante.motif === "aucun") return;
  const accent = variante.accent;
  contexte.save();
  contexte.strokeStyle = accent;
  contexte.fillStyle = accent;
  contexte.lineWidth = 18;
  contexte.lineCap = "round";
  contexte.lineJoin = "round";
  if (variante.motif === "soleil") {
    contexte.beginPath();
    contexte.arc(512, 445, 125, 0, Math.PI * 2);
    contexte.stroke();
    for (let angle = 0; angle < Math.PI * 2; angle += Math.PI / 8) {
      contexte.beginPath();
      contexte.moveTo(512 + Math.cos(angle) * 165, 445 + Math.sin(angle) * 165);
      contexte.lineTo(512 + Math.cos(angle) * 225, 445 + Math.sin(angle) * 225);
      contexte.stroke();
    }
  } else if (variante.motif === "vagues") {
    for (let ligne = 0; ligne < 3; ligne += 1) {
      contexte.beginPath();
      for (let x = 170; x <= 854; x += 12) {
        const y = 360 + ligne * 90 + Math.sin(x / 72) * 32;
        if (x === 170) contexte.moveTo(x, y); else contexte.lineTo(x, y);
      }
      contexte.stroke();
    }
  } else if (variante.motif === "sakura") {
    for (const [x, y, taille] of [[360, 390, 1], [650, 470, 0.8], [520, 610, 0.65]]) {
      for (let petale = 0; petale < 5; petale += 1) {
        const angle = petale * Math.PI * 2 / 5;
        contexte.beginPath();
        contexte.ellipse(x + Math.cos(angle) * 55 * taille, y + Math.sin(angle) * 55 * taille, 28 * taille, 55 * taille, angle, 0, Math.PI * 2);
        contexte.stroke();
      }
      contexte.beginPath();
      contexte.arc(x, y, 12 * taille, 0, Math.PI * 2);
      contexte.fill();
    }
  } else if (variante.motif === "eclair") {
    contexte.beginPath();
    contexte.moveTo(580, 270); contexte.lineTo(425, 495); contexte.lineTo(525, 495);
    contexte.lineTo(445, 735); contexte.lineTo(655, 475); contexte.lineTo(535, 475);
    contexte.stroke();
  } else if (variante.motif === "renard") {
    contexte.beginPath();
    contexte.moveTo(355, 390); contexte.lineTo(405, 275); contexte.lineTo(505, 355);
    contexte.lineTo(615, 275); contexte.lineTo(670, 390); contexte.lineTo(620, 620);
    contexte.lineTo(512, 700); contexte.lineTo(405, 620); contexte.closePath();
    contexte.stroke();
    contexte.beginPath(); contexte.arc(455, 490, 10, 0, Math.PI * 2); contexte.fill();
    contexte.beginPath(); contexte.arc(570, 490, 10, 0, Math.PI * 2); contexte.fill();
  } else {
    contexte.beginPath();
    contexte.moveTo(360, 315); contexte.quadraticCurveTo(512, 235, 664, 315);
    contexte.lineTo(620, 640); contexte.quadraticCurveTo(512, 730, 404, 640); contexte.closePath();
    contexte.stroke();
    contexte.beginPath(); contexte.arc(455, 475, 15, 0, Math.PI * 2); contexte.fill();
    contexte.beginPath(); contexte.arc(570, 475, 15, 0, Math.PI * 2); contexte.fill();
  }
  if (dos) {
    contexte.globalAlpha = 0.45;
    contexte.font = "900 28px Arial";
    contexte.textAlign = "center";
    contexte.fillText("MANGA WAVE / ORIGINAL ART", 512, 660);
  }
  contexte.restore();
}

// Modèle de tee-shirt slim fit créé directement avec Three.js. Chaque variante
// possède une illustration manga originale, sans personnage ni logo protégé.
function creerTextureTshirt(variante, dos = false) {
  const canvas = document.createElement("canvas");
  canvas.width = 1024;
  canvas.height = 1024;
  const contexte = canvas.getContext("2d");
  // Texture transparente : elle sert uniquement d'impression sur le tissu
  // réel, elle ne dessine plus un faux rectangle de tee-shirt.
  contexte.clearRect(0, 0, canvas.width, canvas.height);
  dessinerMotifManga(contexte, variante, dos);
  if (variante.motif === "aucun") return conserver(new THREE.CanvasTexture(canvas));
  contexte.fillStyle = "#fff";
  contexte.textAlign = "center";
  contexte.font = "900 92px Arial";
  contexte.fillText("NOVA", 512, 210);
  contexte.font = "700 30px Arial";
  contexte.letterSpacing = "12px";
  contexte.fillText("WEAR", 512, 258);
  contexte.globalAlpha = 0.8;
  contexte.font = "600 24px Arial";
  contexte.fillText(dos ? "DESIGNED TO MOVE" : "SLIM FIT / 2026", 512, 710);
  contexte.globalAlpha = 1;
  contexte.font = "italic 700 22px Arial";
  const lignes = dos
    ? ["COTON BIO 95%", "ELASTHANNE 5%", "FABRIQUÉ AVEC SOIN", "LAVAGE À 30°C"]
    : [variante.nom.toUpperCase(), "ESSENTIAL COLLECTION"];
  lignes.forEach((ligne, index) => contexte.fillText(ligne, 512, 770 + index * 42));
  return conserver(new THREE.CanvasTexture(canvas));
}

// Prépare un vrai modèle GLB de tee-shirt (géométrie, col, manches et cintre).
// Les deux plans transparents sont seulement les impressions manga originales.
function preparerModeleTshirt(modele) {
  modele.scale.setScalar(4.4);
  // Le fichier de base est un vrai vêtement. On lui ajoute les éléments
  // caractéristiques du polo de la référence : col, patte de boutonnage et
  // petite poche. Ces pièces suivent naturellement toutes les rotations.
  const creerPiecePolo = (points, profondeur, position) => {
    const forme = new THREE.Shape();
    points.forEach(([x, y], index) => {
      if (index === 0) forme.moveTo(x, y);
      else forme.lineTo(x, y);
    });
    forme.closePath();
    const geometrie = conserver(new THREE.ExtrudeGeometry(forme, {
      depth: profondeur,
      bevelEnabled: true,
      bevelSegments: 2,
      bevelSize: 0.008,
      bevelThickness: 0.006,
    }));
    geometrie.translate(0, 0, -profondeur / 2);
    const piece = new THREE.Mesh(geometrie);
    piece.position.set(...position);
    piece.userData.estPoloDetail = true;
    return piece;
  };
  const colGauche = creerPiecePolo([
    [-0.2, -0.08], [-0.015, -0.04], [-0.06, -0.2], [-0.23, -0.14],
  ], 0.035, [0, 0, 0.23]);
  const colDroit = creerPiecePolo([
    [0.2, -0.08], [0.015, -0.04], [0.06, -0.2], [0.23, -0.14],
  ], 0.035, [0, 0, 0.23]);
  const patte = new THREE.Mesh(conserver(new THREE.BoxGeometry(0.075, 0.19, 0.035)));
  patte.position.set(0, -0.135, 0.23);
  patte.userData.estPoloDetail = true;
  const poche = new THREE.Mesh(conserver(new THREE.BoxGeometry(0.2, 0.1, 0.025)));
  poche.position.set(0.18, -0.32, 0.23);
  poche.userData.estPoloDetail = true;
  modele.add(colGauche, colDroit, patte, poche);
  const impression = conserver(new THREE.PlaneGeometry(0.45, 0.46));
  const avant = new THREE.Mesh(impression);
  avant.name = "ImpressionAvant";
  avant.userData.faceDos = false;
  avant.position.set(0, -0.4, 0.225);
  const dos = new THREE.Mesh(impression);
  dos.name = "ImpressionDos";
  dos.userData.faceDos = true;
  dos.position.set(0, -0.4, -0.225);
  dos.rotation.y = Math.PI;
  modele.add(avant, dos);
  modele.traverse((objet) => {
    if (!objet.isMesh) return;
    const materiau = Array.isArray(objet.material) ? objet.material[0] : objet.material;
    objet.userData.estCintre = materiau?.name === "metal";
    objet.userData.estTissu = !objet.userData.estCintre;
    if (objet.userData.estCintre) objet.visible = false;
  });
  return modele;
}

async function chargerEtiquette(chemin) {
  const texture = await new THREE.TextureLoader().loadAsync(chemin);
  texture.colorSpace = THREE.SRGBColorSpace;
  // Conservé pour les futurs produits avec une texture externe.
  // Garder le retournement vertical par défaut évite un texte à l'envers.
  texture.flipY = true;
  return conserver(texture);
}

onMounted(async () => {
  const gestesAudio = ["pointerdown", "touchstart", "keydown", "wheel"];
  gestesAudio.forEach((type) => window.addEventListener(type, deverrouillerAudio, { passive: true }));
  fermerAvecEchap = (evenement) => {
    if (evenement.key === "Escape" && profilActif.value) fermerProfil();
  };
  window.addEventListener("keydown", fermerAvecEchap);
  window.addEventListener("wheel", defilerProfil, { passive: false });
  window.addEventListener("touchstart", commencerTouchProfil, { passive: true });
  window.addEventListener("touchend", terminerTouchProfil, { passive: true });
  window.addEventListener("touchstart", commencerTouchGamme, { passive: true });
  window.addEventListener("touchend", terminerTouchGamme, { passive: true });
  prechargementSons = prechargerSons();
  // Même point d'entrée que le site officiel pour les futures sections.
  window.ciaoSound = { play: jouerSon, unlock: deverrouillerAudio };

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(20, 1, 0.1, 100);
  const raycaster = new THREE.Raycaster();
  const pointer = new THREE.Vector2();
  let parallaxeCibleX = 0;
  let parallaxeCibleY = 0;
  let parallaxeX = 0;
  let parallaxeY = 0;

  // 1. Un canvas transparent laisse voir le dégradé CSS derrière la 3D.
  try {
    renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
  } catch (cause) {
    erreur.value = "La 3D n'est pas disponible dans ce navigateur. Vérifiez l'accélération graphique.";
    chargement.value = false;
    console.error("Création de la scène 3D :", cause);
    return;
  }
  const affichageMobile = window.innerWidth < 600;
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, affichageMobile ? 1.2 : 1.5));
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.domElement.className = "webgl-canvas";
  canvasContainer.value.appendChild(renderer.domElement);

  redimensionner = () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    // Cadrer le produit central, pas toute la rangée. Les voisines peuvent dépasser.
    const distanceHero = Math.max(14.5, 7.5 / camera.aspect);
    // Laisser une marge autour de la sneaker lorsqu'on ouvre sa fiche.
    camera.position.z = THREE.MathUtils.lerp(distanceHero, 6.4, progressionProfil);
    camera.fov = THREE.MathUtils.lerp(20, 38, progressionProfil);
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.render(scene, camera);
  };
  redimensionner();
  window.addEventListener("resize", redimensionner);
  suivreSouris = (evenement) => {
    parallaxeCibleX = (evenement.clientX / window.innerWidth - 0.5) * 2;
    parallaxeCibleY = (evenement.clientY / window.innerHeight - 0.5) * 2;
  };
  window.addEventListener("pointermove", suivreSouris, { passive: true });
  commencerGlisse = (evenement) => {
    if (!profilActif.value || modeGammeComplete.value) return;
    sneakerGlissee = true;
    dernierePositionGlisse = evenement.clientX;
    renderer.domElement.setPointerCapture?.(evenement.pointerId);
  };
  poursuivreGlisse = (evenement) => {
    if (!sneakerGlissee) return;
    rotationManuelle += (evenement.clientX - dernierePositionGlisse) * 0.012;
    dernierePositionGlisse = evenement.clientX;
  };
  terminerGlisse = () => {
    sneakerGlissee = false;
  };
  renderer.domElement.addEventListener("pointerdown", commencerGlisse);
  renderer.domElement.addEventListener("pointermove", poursuivreGlisse);
  renderer.domElement.addEventListener("pointerup", terminerGlisse);
  renderer.domElement.addEventListener("pointercancel", terminerGlisse);

  // 2. Reprendre les projecteurs de la référence à l'échelle de notre scène (×0.5).
  // Leur atténuation est 0.1 sur le site, pas la valeur 2 utilisée par défaut.
  const intensiteProjecteurs = 50 * Math.pow(0.5, 0.1);
  const lumiereHaute = new THREE.SpotLight(
    "#ffffff", intensiteProjecteurs, 4, Math.PI / 4, 1, 0.1,
  );
  lumiereHaute.position.set(0, 1.75, 0);
  lumiereHaute.castShadow = true;
  lumiereHaute.shadow.mapSize.set(affichageMobile ? 512 : 1024, affichageMobile ? 512 : 1024);
  lumiereHaute.shadow.bias = -0.0002;
  lumiereHaute.target.position.set(0, 0, 0.5);
  scene.add(lumiereHaute, lumiereHaute.target);

  const lumiereBasse = new THREE.SpotLight(
    "#ffffff", intensiteProjecteurs, 4, Math.PI / 4, 1, 0.1,
  );
  lumiereBasse.position.set(0, -1.5, 1);
  lumiereBasse.castShadow = true;
  lumiereBasse.shadow.mapSize.set(affichageMobile ? 512 : 1024, affichageMobile ? 512 : 1024);
  lumiereBasse.target.position.set(0, 0, 0.9);
  scene.add(lumiereBasse, lumiereBasse.target);

  // Projecteur mobile : il suit le bloc de texte actif de la sneaker.
  // C'est ce qui rend réellement lisibles « moins de sucre », « arômes »
  // et « stevia », au lieu de seulement faire pivoter le modèle.
  const lumiereInformation = new THREE.SpotLight(
    "#ffffff", 34, 5, Math.PI / 14, 0.35, 0.5,
  );
  lumiereInformation.position.set(0, 0.8, 3.2);
  lumiereInformation.target.position.set(0, 0.1, 0);
  scene.add(lumiereInformation, lumiereInformation.target);

  try {
    // 3. Charger la sneaker réaliste et son éclairage HDR.
    const [modele, environnement] = await Promise.all([
      chargerModele("/sneaker.glb"),
      new HDRLoader().loadAsync("/hdri2.hdr").then(conserver),
    ]);
    // Les sons se chargent en arrière-plan : ils ne bloquent pas la scène 3D.
    if (estDemonte) return;

    // Le HDR sert aux reflets du métal ; ce n'est pas l'image de fond.
    environnement.mapping = THREE.EquirectangularReflectionMapping;
    scene.environment = environnement;
    scene.environmentIntensity = 1;

    // 4. Petit socle procédural pour garder la sneaker bien lisible.
    const socle = new THREE.Mesh(
      conserver(new THREE.CylinderGeometry(1.9, 1.9, 0.12, 96)),
      conserver(new THREE.MeshPhysicalMaterial({
        color: "#171a20",
        metalness: 0.75,
        roughness: 0.25,
        envMap: environnement,
        envMapIntensity: 0.7,
      })),
    );
    socle.position.y = -1.4;
    socle.receiveShadow = true;
    scene.add(socle);
    const solReflet = new THREE.Mesh(
      conserver(new THREE.PlaneGeometry(12, 12)),
      conserver(new THREE.MeshPhysicalMaterial({
        color: "#0e1013",
        metalness: 0.55,
        roughness: 0.2,
        transparent: true,
        opacity: 0.42,
        envMap: environnement,
        envMapIntensity: 0.5,
      })),
    );
    solReflet.rotation.x = -Math.PI / 2;
    solReflet.position.y = -1.346;
    solReflet.receiveShadow = true;
    scene.add(solReflet);

    /*
     * Les textures PBR sont intégrées dans sneaker.glb. Chaque copie clone
     * ses matériaux pour pouvoir changer la couleur sans modifier les autres.
     */
    // 5. Répéter les six modèles pour que le carrousel reste continu.
    const produits = [];
    // Deux passages suffisent sur mobile et sollicitent moins le GPU.
    const nombreCopies = variantes.length * (affichageMobile ? 2 : 4);
    for (let place = 0; place < nombreCopies; place++) {
      const index = place % variantes.length;
      const produit = modele.clone();
      produit.name = variantes[index].nom;
      produit.userData.produitRacine = true;
      produit.userData.place = place;
      // Le GLB possède une échelle interne de 0.1489 : cette valeur remet
      // la sneaker à une taille cohérente avec le socle.
      produit.scale.setScalar(8);

      produit.traverse((objet) => {
        if (!objet.isMesh) return;
        objet.castShadow = true;
        objet.receiveShadow = true;
        const materiaux = Array.isArray(objet.material) ? objet.material : [objet.material];
        objet.material = materiaux.map((materiauOriginal) => {
          const materiau = conserver(materiauOriginal.clone());
          materiau.color.multiply(new THREE.Color(variantes[index].couleur));
          materiau.envMap = environnement;
          materiau.envMapIntensity = 0.8;
          return materiau;
        });
        if (objet.material.length === 1) objet.material = objet.material[0];
      });

      produits.push({ objet: produit, place });
      scene.add(produit);
    }

    // Le clic est attaché au canvas, mais seul le produit central répond.
    cliquerCanette = (evenement) => {
      if (profilActif.value || chargement.value || !renderer?.domElement) return;
      const rectangle = renderer.domElement.getBoundingClientRect();
      pointer.x = ((evenement.clientX - rectangle.left) / rectangle.width) * 2 - 1;
      pointer.y = -((evenement.clientY - rectangle.top) / rectangle.height) * 2 + 1;
      raycaster.setFromCamera(pointer, camera);
      const touchees = raycaster.intersectObjects(produits.map(({ objet }) => objet), true);
      let racine = touchees[0]?.object || null;
      while (racine && !racine.userData.produitRacine) racine = racine.parent;
      if (!racine) return;
      const distance = modulo(racine.userData.place - positionCarrousel + nombreCopies / 2, nombreCopies)
        - nombreCopies / 2;
      if (Math.abs(distance) < 0.65) {
        // Le produit central ouvre la fiche, les voisins font défiler la collection.
        ouvrirProfil();
      } else {
        changerCanette(distance > 0 ? 1 : -1);
      }
    };
    renderer.domElement.addEventListener("click", cliquerCanette);

    chargement.value = false;
    const debut = performance.now();
    let derniereImage = debut;
    const mouvementsReduits = window.matchMedia("(prefers-reduced-motion: reduce)");

    // 6. Déplacer toute la vague vers la cible choisie avec les flèches.
    function animer() {
      const maintenant = performance.now();
      const secondes = (maintenant - debut) / 1000;
      const delta = Math.min((maintenant - derniereImage) / 1000, 0.1);
      derniereImage = maintenant;
      const reduire = mouvementsReduits.matches;

      // Tenir compte du temps écoulé rend la vitesse indépendante de l'écran.
      const rapprochement = reduire ? 1 : 1 - Math.exp(-7 * delta);
      positionCarrousel += (positionCible - positionCarrousel) * rapprochement;
      if (Math.abs(positionCible - positionCarrousel) < 0.001) {
        positionCarrousel = positionCible;
      }

      // Le titre suit la sneaker la plus proche du centre, même pendant le mouvement.
      indexActif.value = modulo(Math.round(positionCarrousel), variantes.length);
      const progression = reduire ? 1 : Math.min(secondes / 1.5, 1);
      const entree = 1 - Math.pow(1 - progression, 3);
      const largeurVague = THREE.MathUtils.clamp(1440 / window.innerWidth, 1, 2.4);
      // La transition officielle laisse le temps de distinguer la rotation et l’étiquette.
      const transitionProfil = reduire ? 1 : 1 - Math.exp(-1.7 * delta);
      progressionProfil += (cibleProfil - progressionProfil) * transitionProfil;
      progressionEtape += (profilEtape.value - progressionEtape) * (reduire ? 1 : 1 - Math.exp(-1.8 * delta));
      progressionGammeComplete += (Number(modeGammeComplete.value) - progressionGammeComplete)
        * (reduire ? 1 : 1 - Math.exp(-1.2 * delta));
      const etapeLumiere = THREE.MathUtils.clamp(progressionEtape, 0, 4);
      const hauteursInformation = [0.1, 0.95, 0.55, 0.08, -0.42];
      const indexLumiere = Math.min(3, Math.floor(etapeLumiere));
      const cibleLumiere = etapeLumiere < 0.5
        ? hauteursInformation[0]
        : THREE.MathUtils.lerp(
          hauteursInformation[indexLumiere],
          hauteursInformation[indexLumiere + 1],
          etapeLumiere - indexLumiere,
        );
      lumiereInformation.position.y = cibleLumiere + 1.45;
      lumiereInformation.target.position.y = cibleLumiere;
      lumiereInformation.intensity = profilActif.value && !modeGammeComplete.value ? 18 : 0;
      // Dès que l'utilisateur revient à l'accueil, le socle réapparaît
      // immédiatement au lieu de rester caché pendant la transition.
      socle.visible = !profilActif.value || (progressionProfil < 0.45 && progressionGammeComplete < 0.25);
      const progressionBenefices = THREE.MathUtils.clamp(progressionEtape, 0, 1);
      // Réglages de la référence : fiche produit (z=6, FOV=40),
      // puis bénéfices (z=12, y=-2, FOV=20), à notre échelle ×0,5.
      // Le modèle mesure un peu plus de 4 unités de haut : ces distances
      // évitent qu'il soit coupé en haut ou en bas pendant la transition.
      const distanceDetail = THREE.MathUtils.lerp(9.4, 12.5, progressionBenefices);
      const hauteurDetail = THREE.MathUtils.lerp(0, -1, progressionBenefices);
      const fovDetail = THREE.MathUtils.lerp(40, 20, progressionBenefices);
      const fovProfil = THREE.MathUtils.lerp(20, fovDetail, progressionProfil);
      const affichageMobile = window.innerWidth < 600;
      camera.fov = THREE.MathUtils.lerp(fovProfil, 24, progressionGammeComplete);
      camera.position.z = THREE.MathUtils.lerp(
        Math.max(14.5, 7.5 / camera.aspect),
        affichageMobile ? distanceDetail + 2.2 : distanceDetail,
        progressionProfil,
      );
      camera.position.z = THREE.MathUtils.lerp(camera.position.z, 10, progressionGammeComplete);
      const cameraBaseY = THREE.MathUtils.lerp(0, hauteurDetail, progressionProfil)
        - (affichageMobile && profilActif.value ? 0.8 : 0);
      const cameraBaseX = THREE.MathUtils.lerp(0, -1.5, progressionGammeComplete);
      const vitesseParallaxe = reduire ? 1 : 1 - Math.exp(-3.5 * delta);
      parallaxeX += (parallaxeCibleX - parallaxeX) * vitesseParallaxe;
      parallaxeY += (parallaxeCibleY - parallaxeY) * vitesseParallaxe;
      camera.position.y = cameraBaseY - parallaxeY * 0.28;
      camera.position.x = cameraBaseX + parallaxeX * 0.45;
      camera.lookAt(0, -0.35, 0);
      camera.updateProjectionMatrix();

      // Chaque étape révèle un angle différent : face, talon, semelle puis
      // retour vers le côté. Le scroll devient ainsi une vraie timeline 3D.
      const rotationsDosY = [
        -Math.PI * 0.87,
        -Math.PI * 1.08,
        -Math.PI * 0.75,
        -Math.PI * 1.02,
      ];
      const etapeRotation = THREE.MathUtils.clamp(progressionEtape, 0, 4);
      let rotationDosY = rotationsDosY[0];
      if (etapeRotation >= 1) {
        const indexRotation = Math.min(2, Math.floor(etapeRotation - 1));
        rotationDosY = THREE.MathUtils.lerp(
          rotationsDosY[indexRotation],
          rotationsDosY[indexRotation + 1],
          etapeRotation - 1 - indexRotation,
        );
      } else {
        rotationDosY = THREE.MathUtils.lerp(0.262, rotationsDosY[0], etapeRotation);
      }

      produits.forEach(({ objet, place }) => {
        const distance = modulo(place - positionCarrousel + nombreCopies / 2, nombreCopies)
          - nombreCopies / 2;
        const hauteurVague = Math.sin(distance * 0.875 * largeurVague) * 0.5;
        const auCentre = Math.abs(distance) < 0.45;
        const transitionCentre = auCentre ? progressionProfil : 0;
        const facteurProfondeur = THREE.MathUtils.clamp(1 - Math.abs(distance) * 0.055, 0.72, 1);
        // L’ouverture montre la face de la saveur. Le premier scroll passe
        // ensuite à la face arrière qui porte toutes les mentions imprimées.
        const rotationProfilY = rotationDosY + rotationManuelle;
        // Pendant la lecture, la sneaker est droite et son côté est exactement
        // face à la caméra : les mentions ne restent jamais de profil.
        const rotationProfilX = THREE.MathUtils.lerp(-Math.PI * 0.208, -Math.PI * 0.18, progressionBenefices);
        const rotationProfilZ = THREE.MathUtils.lerp(
          Math.PI * 0.125,
          progressionEtape % 2 < 1 ? -Math.PI * 0.055 : Math.PI * 0.028,
          progressionBenefices,
        );
        const positionHero = new THREE.Vector3(
          distance * 1.55,
          hauteurVague - 0.7 - (1 - entree) * 3,
          -Math.abs(distance) * 1.75 - 0.1,
        );
        const rotationHero = new THREE.Euler(
          -Math.PI / 9,
          distance * 1.55 - Math.PI / 9,
          Math.PI / 16,
        );
        // La composition finale reste centrée même avec moins de copies sur mobile.
        const centreGammeFinale = affichageMobile ? (nombreCopies - 1) / 2 : 8.5;
        const finalOffset = place - centreGammeFinale;
        const positionFinale = new THREE.Vector3(
          finalOffset * 1.02,
          -1.2 + finalOffset * 0.17,
          -1.3 - Math.abs(finalOffset) * 0.06,
        );
        const rotationFinale = new THREE.Euler(-0.12, finalOffset * 0.055, finalOffset * 0.028 - 0.12);
        const estDansLaGammeFinale = place < (affichageMobile ? nombreCopies : 18);
        objet.visible = progressionGammeComplete > 0.08
          ? estDansLaGammeFinale
          : !profilActif.value || progressionProfil < 0.55 || auCentre;
        // Vue profil puis transition vers l’arc complet de produits.
        objet.position.lerp(positionHero, 1);
        // Sur mobile, le produit reste centré et remonte légèrement pour
        // laisser un espace clair au panneau de détails en bas.
        objet.position.lerp(new THREE.Vector3(
          affichageMobile ? 0 : 0.5,
          affichageMobile ? 0.55 : 0.1,
          0,
        ), transitionCentre);
        objet.position.lerp(positionFinale, progressionGammeComplete);
        objet.position.y += reduire ? 0 : Math.sin(secondes * 1.1 + distance * 0.5) * 0.025;
        objet.rotation.set(
          THREE.MathUtils.lerp(-Math.PI / 9, rotationProfilX, transitionCentre),
          THREE.MathUtils.lerp(distance * 1.55 - Math.PI / 9, rotationProfilY, transitionCentre),
          THREE.MathUtils.lerp(Math.PI / 16, rotationProfilZ, transitionCentre),
        );
        objet.rotation.x = THREE.MathUtils.lerp(objet.rotation.x, rotationFinale.x, progressionGammeComplete);
        objet.rotation.y = THREE.MathUtils.lerp(objet.rotation.y, rotationFinale.y, progressionGammeComplete);
        objet.rotation.z = THREE.MathUtils.lerp(objet.rotation.z, rotationFinale.z, progressionGammeComplete);
        const facteurTailleMobile = affichageMobile ? 0.84 : 1;
        objet.scale.setScalar(THREE.MathUtils.lerp(
          THREE.MathUtils.lerp(7.5, 10, transitionCentre),
          8,
          progressionGammeComplete,
        ) * facteurProfondeur * facteurTailleMobile);


        // Le GLB possède déjà l’orientation correcte de Shell/Top/Bottom.
        // Ne pas leur appliquer une rotation supplémentaire : elle couchait
        // les mentions du dos et les rendait illisibles pendant le profil.
      });

      renderer.render(scene, camera);
      animationId = requestAnimationFrame(animer);
    }
    animer();
  } catch (cause) {
    if (estDemonte) return;
    erreur.value = "Impossible de charger la scène. Rechargez la page pour réessayer.";
    chargement.value = false;
    console.error("Chargement de la gamme :", cause);
  }
});

onBeforeUnmount(() => {
  estDemonte = true;
  clearTimeout(minuteurEtape);
  ["pointerdown", "touchstart", "keydown", "wheel"].forEach((type) => {
    window.removeEventListener(type, deverrouillerAudio);
  });
  if (fermerAvecEchap) window.removeEventListener("keydown", fermerAvecEchap);
  window.removeEventListener("wheel", defilerProfil);
  window.removeEventListener("touchstart", commencerTouchProfil);
  window.removeEventListener("touchend", terminerTouchProfil);
  window.removeEventListener("touchstart", commencerTouchGamme);
  window.removeEventListener("touchend", terminerTouchGamme);
  if (cliquerCanette && renderer?.domElement) renderer.domElement.removeEventListener("click", cliquerCanette);
  if (contexteAudio) {
    void contexteAudio.close();
    contexteAudio = null;
  }
  if (window.ciaoSound) delete window.ciaoSound;
  cancelAnimationFrame(animationId);
  window.removeEventListener("resize", redimensionner);
  window.removeEventListener("pointermove", suivreSouris);
  if (renderer?.domElement) {
    renderer.domElement.removeEventListener("pointerdown", commencerGlisse);
    renderer.domElement.removeEventListener("pointermove", poursuivreGlisse);
    renderer.domElement.removeEventListener("pointerup", terminerGlisse);
    renderer.domElement.removeEventListener("pointercancel", terminerGlisse);
  }
  ressources.forEach((ressource) => ressource.dispose());
  ressources.clear();

  if (renderer) {
    renderer.dispose();
    renderer.domElement.remove();
  }
});
</script>
<style scoped>
@font-face {
  font-family: "Franklin Ciao";
  src: url("/franklin-black-italic.woff2") format("woff2");
  font-style: italic;
  font-weight: 900;
  font-display: swap;
}

.experience {
  --accent: #b6ff5c;
  --couleur-saveur: #9089d3;
  position: relative;
  min-height: 100svh;
  overflow: hidden;
  isolation: isolate;
  color: #f4f4ed;
  background:
    radial-gradient(ellipse at 50% 115%, color-mix(in srgb, var(--couleur-saveur) 48%, transparent), transparent 50%),
    linear-gradient(180deg, #020203 20%, #0c0c0e 43%, #505054 86%, #242029 100%);
}

.experience.profil-ouvert {
  background:
    radial-gradient(ellipse at 50% 72%, color-mix(in srgb, var(--couleur-saveur) 42%, transparent), transparent 58%),
    linear-gradient(180deg, #050607 8%, #111316 48%, #25282d 100%);
}

.experience.mode-gamme-complete {
  background:
    radial-gradient(ellipse at 50% 76%, rgb(115 112 120 / 45%), transparent 48%),
    linear-gradient(180deg, #020203 18%, #161519 60%, #39383b 100%);
}

.entete-site {
  position: absolute;
  z-index: 4;
  top: 42px;
  right: 4%;
  left: 4%;
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}

.logo-ciao {
  color: white;
  font-family: "Franklin Ciao", sans-serif;
  font-size: clamp(1.5rem, 2.6vw, 2.8rem);
  font-style: italic;
  font-weight: 900;
  line-height: 0.72;
  text-align: center;
  text-shadow: 0 0 12px rgb(255 255 255 / 32%);
  transform: skew(-8deg);
}

.logo-ciao span { font-size: 0.82em; }
.actions-site { position: absolute; right: 0; display: flex; align-items: center; gap: 28px; pointer-events: auto; }
.menu-site { display: inline-flex; align-items: center; gap: 4px; color: rgb(255 255 255 / 80%); font-size: 0.62rem; letter-spacing: 0.1em; }
.menu-site i { width: 3px; height: 3px; border-radius: 50%; background: currentColor; }
.contact-site { padding: 11px 20px; border-radius: 7px; color: #111; background: #fff; font-size: 0.65rem; text-decoration: none; box-shadow: 0 0 18px rgb(255 255 255 / 38%); }

.experience::after {
  position: absolute;
  z-index: 1;
  inset: 0;
  pointer-events: none;
  content: "";
  opacity: 0.025;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 160 160' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.85' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='.45'/%3E%3C/svg%3E");
}

.canvas-container {
  position: absolute;
  z-index: 0;
  inset: 0;
}

.canvas-container :deep(.webgl-canvas) {
  display: block;
  width: 100%;
  height: 100%;
  outline: none;
  cursor: pointer;
}

.hud-superieur {
  position: absolute;
  z-index: 3;
  top: 21px;
  left: 5.3%;
  width: 89.4%;
  height: 2px;
  pointer-events: none;
}

.hud-ligne {
  position: relative;
  display: block;
  width: 100%;
  height: 1px;
  background: rgb(255 255 255 / 22%);
}

.hud-ligne::before {
  position: absolute;
  inset: -1px 0 auto;
  height: 1px;
  content: "";
  background: linear-gradient(90deg, transparent, rgb(255 255 255 / 18%), transparent);
}

.hud-ligne i {
  position: absolute;
  top: 50%;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: white;
  box-shadow: 0 0 5px 2px rgb(255 255 255 / 85%), 0 0 18px 5px rgb(255 255 255 / 48%);
  transform: translate(-50%, -50%);
  transition: left 0.7s cubic-bezier(0.22, 1, 0.36, 1);
}

.halo-gamme {
  position: absolute;
  z-index: 1;
  right: -20%;
  bottom: -18%;
  left: -20%;
  height: 57%;
  pointer-events: none;
  opacity: 0.48;
  background: radial-gradient(ellipse at center, color-mix(in srgb, var(--couleur-saveur) 55%, transparent) 0%, transparent 66%);
  filter: blur(18px);
}

.message-chargement {
  position: absolute;
  z-index: 2;
  inset: 0;
  display: grid;
  place-items: center;
  margin: 0;
  padding: 24px;
  text-align: center;
  font-size: 0.875rem;
}

.chargement-premium {
  position: absolute;
  z-index: 5;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  color: rgb(255 255 255 / 82%);
  background: radial-gradient(circle at center, rgb(42 45 50 / 76%), rgb(4 5 6 / 96%) 58%);
  letter-spacing: 0.16em;
  pointer-events: none;
}

.chargement-rond {
  width: 34px;
  height: 34px;
  border: 1px solid rgb(255 255 255 / 20%);
  border-top-color: white;
  border-radius: 50%;
  animation: tourner-chargement 0.9s linear infinite;
}

.chargement-titre {
  color: white;
  font-family: "Franklin Ciao", sans-serif;
  font-size: 1.5rem;
  font-style: italic;
  font-weight: 900;
  letter-spacing: 0.05em;
}

.chargement-label { color: rgb(255 255 255 / 48%); font-size: 0.55rem; }
.chargement-barre { width: min(260px, 55vw); height: 2px; overflow: hidden; background: rgb(255 255 255 / 16%); }
.chargement-barre i { display: block; width: 38%; height: 100%; background: white; box-shadow: 0 0 10px white; animation: avancer-chargement 1.5s ease-in-out infinite; }

@keyframes tourner-chargement { to { transform: rotate(360deg); } }
@keyframes avancer-chargement { 0% { transform: translateX(-110%); } 100% { transform: translateX(280%); } }

.bouton-son {
  position: absolute;
  z-index: 3;
  top: 48px;
  left: 38px;
  display: inline-flex;
  align-items: center;
  gap: 9px;
  min-width: 58px;
  padding: 6px 8px;
  border: 0;
  color: rgb(255 255 255 / 88%);
  background: transparent;
  font-size: 0.65rem;
  letter-spacing: 0.12em;
  cursor: pointer;
}

.bouton-son:hover { color: white; }
.bouton-son:focus-visible { outline: 1px solid white; outline-offset: 4px; }
.barres-son { display: inline-flex; align-items: center; gap: 2px; height: 12px; }
.barres-son i { display: block; width: 2px; border-radius: 1px; background: currentColor; }
.bouton-son[aria-pressed="false"] .barres-son { opacity: 0.35; }

.commandes-gamme {
  position: absolute;
  z-index: 2;
  inset: 0;
  pointer-events: none;
}

.zone-produit {
  position: absolute;
  z-index: 1;
  top: 24%;
  left: 50%;
  width: clamp(110px, 13vw, 190px);
  height: 48%;
  padding: 0;
  border: 0;
  background: transparent;
  transform: translateX(-50%);
  pointer-events: auto;
  cursor: pointer;
}

.zone-produit:focus-visible {
  outline: 1px solid rgb(255 255 255 / 58%);
  outline-offset: 8px;
}

.fleche-gamme {
  position: absolute;
  top: 48.5%;
  display: grid;
  place-items: center;
  width: 48px;
  height: 60px;
  padding: 0;
  border: 0;
  border-radius: 8px;
  color: white;
  background: transparent;
  transform: translate(-50%, -50%);
  pointer-events: auto;
  cursor: pointer;
}

.fleche-gauche { left: calc(50% - clamp(140px, 11.1vw, 185px)); }
.fleche-droite { left: calc(50% + clamp(140px, 11.1vw, 185px)); }
.fleche-gamme svg { width: 14px; height: 28px; fill: currentColor; }
.fleche-gamme .vers-droite { transform: scaleX(-1); }
.fleche-gamme:hover { background: rgb(255 255 255 / 6%); }
.fleche-gamme:focus-visible { outline: 2px solid white; outline-offset: 4px; }

.nom-produit {
  position: absolute;
  bottom: 10%;
  left: 50%;
  width: calc(100% - 32px);
  margin: 0;
  transform: translateX(-50%);
  color: white;
  text-align: center;
  text-transform: uppercase;
  font-family: "Franklin Ciao", sans-serif;
  font-style: italic;
  font-weight: 900;
  font-size: clamp(28px, 3.05vw, 48px);
  line-height: 0.82;
  text-shadow: 0 2px 20px rgb(0 0 0 / 50%);
}

.nom-produit span { display: block; }

.indice-clic {
  position: absolute;
  bottom: 5.2%;
  left: 50%;
  margin: 0;
  color: rgb(255 255 255 / 30%);
  font-size: 0.52rem;
  letter-spacing: 0.14em;
  transform: translateX(-50%);
  white-space: nowrap;
  pointer-events: none;
}

.pagination-gamme {
  position: absolute;
  z-index: 2;
  bottom: 2.1%;
  left: 50%;
  width: min(34vw, 490px);
  min-width: 240px;
  transform: translateX(-50%);
  pointer-events: none;
}

.pagination-ligne {
  position: relative;
  height: 4px;
  border-radius: 999px;
  background: linear-gradient(90deg, #9089d3 0%, #00a6e2 20%, #71bd96 40%, #eeb169 60%, #e59de6 80%, #ff659d 100%);
  box-shadow: 0 0 13px rgb(255 255 255 / 24%);
}

.pagination-point {
  position: absolute;
  top: 50%;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #9089d3;
  box-shadow: 0 0 10px rgb(144 137 211 / 80%);
  transform: translate(-50%, -50%);
  transition: left 0.35s ease;
}

.scroll-discover {
  display: block;
  margin-top: 15px;
  color: rgb(255 255 255 / 48%);
  font-size: 0.58rem;
  letter-spacing: 0.09em;
  text-align: center;
}

.profil-produit {
  position: absolute;
  z-index: 4;
  inset: 0;
  pointer-events: none;
}

.profil-carte {
  position: absolute;
  top: 50%;
  left: clamp(28px, 10vw, 150px);
  width: min(390px, 36vw);
  transform: translateY(-50%);
  pointer-events: auto;
}

.profil-subhead { display: inline-flex; align-items: stretch; margin-bottom: 28px; color: #16151e; background: rgb(255 255 255 / 90%); font-size: 0.72rem; font-weight: 600; letter-spacing: 0.06em; }
.profil-subhead-icon { display: grid; place-items: center; width: 30px; color: #1c1731; background: var(--couleur-saveur); font-size: 1.05rem; }
.profil-subhead-label { padding: 8px 10px 7px; text-decoration: line-through; text-decoration-thickness: 1px; }

.profil-carte h2 {
  display: flex;
  flex-direction: column;
  margin: 0 0 24px;
  color: white;
  font-family: "Franklin Ciao", sans-serif;
  font-size: clamp(2.8rem, 5vw, 5rem);
  font-style: italic;
  font-weight: 900;
  line-height: 0.78;
  text-transform: uppercase;
}

.profil-carte p {
  max-width: 28rem;
  margin: 0;
  color: rgb(255 255 255 / 86%);
  font-size: clamp(0.95rem, 1.2vw, 1.12rem);
  line-height: 1.52;
}

.profil-texte-enter-active,
.profil-texte-leave-active {
  transition: opacity 0.32s ease, transform 0.32s ease;
}

.profil-texte-enter-from { opacity: 0; transform: translateY(12px); }
.profil-texte-leave-to { opacity: 0; transform: translateY(-12px); }

.profil-infos {
  display: flex;
  flex-wrap: wrap;
  gap: 8px 14px;
  margin-top: 25px;
  color: rgb(255 255 255 / 52%);
  font-size: 0.58rem;
  letter-spacing: 0.12em;
}

.profil-scroll {
  margin-top: 24px !important;
  color: rgb(255 255 255 / 38%) !important;
  font-size: 0.56rem !important;
  letter-spacing: 0.14em;
}

.bouton-fermer {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  margin-top: 35px;
  padding: 9px 0;
  border: 0;
  color: rgb(255 255 255 / 72%);
  background: transparent;
  font-size: 0.62rem;
  letter-spacing: 0.13em;
  cursor: pointer;
}

.bouton-fermer span { font-size: 1.35rem; line-height: 0.5; }
.bouton-fermer:hover { color: white; }
.bouton-fermer:focus-visible { outline: 1px solid white; outline-offset: 5px; }

.fin-gamme {
  position: absolute;
  z-index: 4;
  right: 0;
  bottom: 7%;
  left: 0;
  margin: 0;
  color: rgb(255 255 255 / 42%);
  font-size: 0.58rem;
  letter-spacing: 0.14em;
  text-align: center;
  pointer-events: none;
}

.profil-navigation {
  position: absolute;
  top: 34%;
  right: clamp(28px, 7vw, 102px);
  display: flex;
  flex-direction: column;
  gap: 18px;
  pointer-events: auto;
}

.profil-navigation button {
  display: grid;
  place-items: center;
  width: 52px;
  height: 52px;
  padding: 12px;
  border: 1px solid rgb(255 255 255 / 22%);
  border-radius: 50%;
  color: rgb(255 255 255 / 58%);
  background: rgb(8 8 12 / 22%);
  cursor: pointer;
  transition: color 0.25s ease, border-color 0.25s ease, background 0.25s ease, box-shadow 0.25s ease;
}

.profil-navigation button:hover,
.profil-navigation button:focus-visible {
  color: white;
  border-color: rgb(255 255 255 / 65%);
}

.profil-navigation button.actif {
  color: white;
  border-color: rgb(255 255 255 / 80%);
  background: radial-gradient(circle, rgb(255 255 255 / 38%), rgb(103 93 172 / 58%));
  box-shadow: 0 0 18px rgb(145 133 255 / 42%);
}

.profil-navigation button:focus-visible { outline: 1px solid white; outline-offset: 5px; }
.profil-navigation svg { width: 100%; height: 100%; fill: none; stroke: currentColor; stroke-linecap: round; stroke-linejoin: round; stroke-width: 1.4; }

.points-chaussure {
  position: absolute;
  z-index: 5;
  inset: 0;
  pointer-events: none;
}

.point-chaussure {
  position: absolute;
  width: 28px;
  height: 28px;
  padding: 0;
  border: 1px solid rgb(255 255 255 / 78%);
  border-radius: 50%;
  color: #17191c;
  background: rgb(255 255 255 / 90%);
  font-size: 0.65rem;
  font-weight: 700;
  pointer-events: auto;
  cursor: pointer;
  transform: translate(-50%, -50%);
  transition: transform 0.25s ease, color 0.25s ease, background 0.25s ease;
}

.point-chaussure::after {
  position: absolute;
  inset: -6px;
  border: 1px solid rgb(255 255 255 / 28%);
  border-radius: inherit;
  content: "";
  animation: pulsation-point 1.8s ease-in-out infinite;
}

.point-chaussure::before {
  position: absolute;
  right: 100%;
  top: 50%;
  width: 34px;
  height: 1px;
  content: "";
  background: linear-gradient(90deg, transparent, rgb(255 255 255 / 65%));
  transform-origin: right center;
  transform: rotate(-16deg);
}

.point-chaussure:hover,
.point-chaussure.actif,
.point-chaussure:focus-visible {
  color: white;
  background: var(--couleur-saveur);
  transform: translate(-50%, -50%) scale(1.2);
}

.point-chaussure:focus-visible { outline: 1px solid white; outline-offset: 4px; }

@keyframes pulsation-point {
  0%, 100% { opacity: 0.35; transform: scale(0.9); }
  50% { opacity: 0; transform: scale(1.25); }
}

@media (max-width: 599px) {
  .bouton-son { top: 22px; left: 14px; }
  .hud-superieur { top: 21px; left: 5%; width: 90%; }
  .entete-site { top: 38px; right: 20px; left: 20px; justify-content: flex-end; }
  .logo-ciao { position: absolute; left: 50%; font-size: 1.65rem; transform: translateX(-50%) skew(-8deg); }
  .actions-site { gap: 10px; }
  .menu-site { font-size: 0.52rem; }
  .contact-site { padding: 8px 11px; font-size: 0.52rem; }
  .zone-produit { top: 25%; width: 120px; height: 46%; }
  .fleche-gauche { left: 28px; }
  .fleche-droite { left: calc(100% - 28px); }
  .nom-produit { bottom: 11%; font-size: clamp(26px, 7.2vw, 36px); }
  .indice-clic { bottom: 7.5%; font-size: 0.45rem; }
  .pagination-gamme { bottom: 2.6%; width: calc(100% - 80px); min-width: 0; }
  .profil-carte {
    top: auto;
    right: 16px;
    bottom: 14px;
    left: 16px;
    width: auto;
    max-height: calc(100svh - 132px);
    overflow-y: auto;
    overscroll-behavior: contain;
    padding: 18px 18px 16px;
    border: 1px solid rgb(255 255 255 / 12%);
    border-radius: 16px;
    background: linear-gradient(180deg, rgb(5 6 8 / 18%), rgb(5 6 8 / 86%));
    backdrop-filter: blur(8px);
  }
  .profil-carte h2 { margin-bottom: 12px; font-size: clamp(1.55rem, 7vw, 2.2rem); line-height: 0.88; }
  .profil-carte p { max-width: none; font-size: 0.76rem; line-height: 1.46; }
  .profil-infos { gap: 8px 12px; margin-top: 18px; }
  .profil-subhead { margin-bottom: 18px; font-size: 0.58rem; }
  .profil-scroll { margin-top: 18px !important; line-height: 1.4; }
  .bouton-fermer { margin-top: 16px; }
  .profil-navigation { top: 20%; right: 14px; gap: 9px; }
  .profil-navigation button { width: 42px; height: 42px; padding: 9px; }
}

.topbar {
  position: absolute;
  z-index: 3;
  top: 0;
  left: 0;
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  align-items: center;
  width: 100%;
  padding: 28px 38px;
}

.brand {
  justify-self: start;
  color: inherit;
  font-size: 0.82rem;
  font-weight: 800;
  letter-spacing: 0.08em;
  text-decoration: none;
}

.brand span,
.eyebrow {
  color: var(--accent);
}

.lesson,
.chapter,
.status,
.eyebrow {
  margin: 0;
  font-size: 0.62rem;
  font-weight: 700;
  line-height: 1;
  letter-spacing: 0.18em;
}

.lesson,
.chapter {
  color: rgb(244 244 237 / 55%);
}

.menu-button {
  display: grid;
  grid-template-columns: repeat(2, 3px);
  gap: 3px;
  justify-self: end;
  padding: 9px;
  border: 1px solid rgb(255 255 255 / 18%);
  border-radius: 50%;
  color: inherit;
  background: rgb(255 255 255 / 3%);
}

.menu-button span {
  width: 3px;
  height: 3px;
  border-radius: 50%;
  background: currentColor;
}

.hero {
  position: absolute;
  z-index: 2;
  top: 50%;
  left: clamp(28px, 7vw, 112px);
  width: min(42rem, 48vw);
  transform: translateY(-50%);
  pointer-events: none;
}

.eyebrow {
  margin-bottom: 22px;
}

.hero h1 {
  display: flex;
  flex-direction: column;
  margin: 0;
  font-size: clamp(3.6rem, 8.5vw, 9.3rem);
  font-weight: 800;
  line-height: 0.77;
  letter-spacing: -0.075em;
}

.hero h1 span:last-child {
  color: transparent;
  -webkit-text-stroke: 1px rgb(244 244 237 / 45%);
}

.intro {
  max-width: 31rem;
  margin: 34px 0 0 6px;
  color: rgb(244 244 237 / 62%);
  font-size: clamp(0.8rem, 1.15vw, 1rem);
  line-height: 1.55;
}

.status,
.chapter {
  position: absolute;
  z-index: 3;
  bottom: 31px;
}

.status {
  left: 38px;
  display: flex;
  align-items: center;
  gap: 9px;
  max-width: 75vw;
  color: rgb(244 244 237 / 48%);
}

.status-dot {
  flex: 0 0 auto;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  color: #ffad5c;
  background: currentColor;
  box-shadow: 0 0 12px currentColor;
}

.is-ready .status-dot {
  color: var(--accent);
}

.chapter {
  right: 38px;
}

.corner {
  position: absolute;
  z-index: 3;
  width: 15px;
  height: 15px;
  border-color: rgb(255 255 255 / 28%);
  border-style: solid;
  pointer-events: none;
}

.corner-top-left {
  top: 82px;
  left: 38px;
  border-width: 1px 0 0 1px;
}

.corner-top-right {
  top: 82px;
  right: 38px;
  border-width: 1px 1px 0 0;
}

.corner-bottom-left {
  bottom: 62px;
  left: 38px;
  border-width: 0 0 1px 1px;
}

.corner-bottom-right {
  right: 38px;
  bottom: 62px;
  border-width: 0 1px 1px 0;
}

@media (max-width: 899px) {
  .experience {
    background:
      radial-gradient(circle at 50% 56%, color-mix(in srgb, var(--couleur-saveur) 48%, transparent), transparent 30%),
      linear-gradient(155deg, #080a0b, #1c2021 65%, #090a0b);
  }

  .topbar {
    grid-template-columns: 1fr 1fr;
    padding: 22px 20px;
  }

  .lesson {
    display: none;
  }

  .hero {
    top: 20%;
    left: 20px;
    width: calc(100% - 40px);
    transform: none;
  }

  .eyebrow {
    margin-bottom: 14px;
  }

  .hero h1 {
    font-size: clamp(3.2rem, 17vw, 6rem);
  }

  .intro {
    display: none;
  }

  .status {
    bottom: 25px;
    left: 20px;
  }

  .chapter {
    right: 20px;
    bottom: 27px;
  }

  .corner-top-left,
  .corner-bottom-left {
    left: 20px;
  }

  .corner-top-right,
  .corner-bottom-right {
    right: 20px;
  }

  .corner-top-left,
  .corner-top-right {
    top: 67px;
  }

  .corner-bottom-left,
  .corner-bottom-right {
    bottom: 54px;
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
  }
}
</style>
