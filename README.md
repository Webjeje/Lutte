
# Compteur de Lutte – Webapp HTML/CSS/JS

Application web **mobile-first** en thème sombre pour compter les actions de deux lutteurs (**Bleu** et **Rouge**), avec coefficients personnalisables, **Undo/Redo**, **statistiques** (Chart.js) et **export en QR** (QRious).
Titre de la page : **🤼 Compteur de Lutte**

---

## Sommaire

1. [Aperçu rapide](#aperçu-rapide)
2. [Fonctionnalités](#fonctionnalités)
3. [Mode d’emploi (exhaustif)](#mode-demploi-exhaustif)
4. [Caractéristiques techniques](#caractéristiques-techniques)
5. [Développement](#développement)
6. [Personnalisation](#personnalisation)
7. [Données & formats](#données--formats)
8. [FAQ / Problèmes connus](#faq--problèmes-connus)
9. [Licence](#licence)

---

## Aperçu rapide

* Deux colonnes **Bleu** et **Rouge**.
* **5 actions** par lutteur : `saisie`, `chute`, `passage dos`, `mise en danger`, `tombé`.
* En haut de chaque colonne : **Score total** (avec coef) et **Volume** (nombre d’actions sans coef).
* Barre d’outils (icônes seules) : **Annuler**, **Rétablir**, **Réinitialiser**, **Paramètres**, **Statistiques**, **Export QR**.
* **Sauvegarde automatique** dans le navigateur (LocalStorage).

---

## Fonctionnalités

* **Saisie ultra-rapide** : 1 tap = +1 action.
* **Undo/Redo** : piles d’historique pour corriger à la volée.
* **Paramètres par lutteur** :

  * Coefficient de chaque action (0 à 5).
  * Activation/désactivation d’une action (désactive le bouton).
* **Statistiques** : graphe **lignes** des volumes par action (Bleu/Rouge) via Chart.js.
* **Export** :

  * **Ligne de données** compatible CSV/Excel (séparateur `,` ou `;`).
  * **QR code** encodant cette ligne + **téléchargement PNG**.
* **Thème sombre mobile-first**, accent **Bleu/Rouge** renforcé (bordures, fonds, hover).
* **Raccourcis clavier** : `Ctrl/Cmd+Z` (Annuler), `Ctrl/Cmd+Y` (Rétablir), `Échap` (fermer modals).

---

## Mode d’emploi (exhaustif)

### 1) Saisir les actions

* Touchez un bouton d’action dans la colonne **Bleu** ou **Rouge**.
* Le **Volume** (actions) et le **Score** (avec coefficients) se mettent à jour automatiquement.

### 2) Lire les totaux

* **Volume** : somme des clics (sans coef).
* **Score** : somme des `compteurs[i] × coefficients[i]`.

### 3) Annuler / Rétablir

* Icônes du header ou raccourcis clavier.
* `Annuler` retire la **dernière action** saisie (pile d’historique).
* `Rétablir` ré-applique la dernière action annulée (pile futur).

### 4) Réinitialiser

* Icône **Réinitialiser** → **alerte de confirmation** → remet à zéro tous les compteurs et vide l’historique.

### 5) Paramètres (⚙️)

* Ouvre un **modal** avec deux panneaux : **Lutteur BLEU** et **Lutteur ROUGE**.
* Pour chaque action :

  * **Coefficient** (liste 0 → 5).
  * **Actif** (switch) : si inactif, le bouton d’action est grisé et inutilisable.
* Les changements sont **persistés** (LocalStorage) et appliqués immédiatement.

### 6) Statistiques (📈)

* Ouvre un modal avec un **graphe en lignes** (Chart.js) :

  * Axe X : les 5 actions.
  * Deux courbes : **Bleu** et **Rouge** (volumes, sans coef).
* Les données se mettent à jour à chaque ouverture.

### 7) Export QR (▣)

* Ouvre un modal **Export**.
* Renseignez : **Arbitre**, **Lutteur BLEU**, **Lutteur ROUGE**.
* Choisissez le **séparateur** :

  * `,` : CSV classique.
  * `;` : recommandé pour **Excel/Numbers** (paramétrages régionaux).
* Cliquez **Générer le QR** :

  * La **ligne de données** s’affiche (copiable).
  * Le **QR code** encode exactement cette ligne.
  * **Télécharger le PNG** du QR (bouton fourni).
* Bouton **Copier la ligne** : place la ligne exportée dans le presse-papiers.

### 8) Sauvegarde / Persistance

* L’app **sauvegarde automatiquement** l’état dans le **LocalStorage** : compteurs, coefficients, activations, noms.
* Pour repartir de zéro : **Réinitialiser** depuis le header ou **vider** le stockage du site (outils du navigateur).

---

## Caractéristiques techniques

### Stack & dépendances

* **Vanilla HTML/CSS/JS** (aucun framework requis).
* **Chart.js** 4.4.1 – graphe des volumes.
* **QRious** 4.0.2 – génération de QR code (canvas).
* **Bootstrap Icons** 1.11.3 – icônes de la barre d’outils (CDN).

### Architecture du code (principales fonctions)

* **État global `state`** :

  ```js
  state = {
    blue: { counts:[0..4], coeffs:[0..5], active:[bool×5], name:"BLEU" },
    red:  { counts:[0..4], coeffs:[0..5], active:[bool×5], name:"ROUGE" },
    referee: "",
    history: [],  // pour Undo
    future: []    // pour Redo
  }
  ```
* **Persistance** : `save()` et `load()` sur `localStorage` (`LS_KEY = "lutte_app_v1"`).
* **Calculs** : `totalPoints(side)`, `totalVolume(side)`.
* **Rendu** :

  * `renderColumn(side)` crée les boutons d’actions, pastilles “coef” et compteurs.
  * `renderSettings()` fabrique les lignes de paramétrage (coef + actif) par lutteur.
  * `renderAll()` met à jour les deux colonnes.
* **Actions utilisateur** :

  * `onActionClick(e)` incrémente un compteur et pousse dans `history`.
  * `undo() / redo()` gèrent piles `history` et `future`.
  * `resetCounts()` remet tout à zéro avec confirmation.
* **Modals** : `openModal(id)`, `closeModals()` (fond blur, fermeture au clic sur l’arrière-plan ou bouton “Fermer”).
* **Export** :

  * `buildExportLine(sep)` construit la **ligne CSV**.
  * `updateQr()` met à jour `textarea`, `QRious`, lien **PNG** et persiste les noms.
* **Statistiques** : `openStats()` (instancie/rafraîchit Chart.js).

### Thème & UI

* **CSS variables** (`:root`) pour couleurs : `--blue`, `--red`, `--bg`, `--card`, etc.
* Accent **Bleu/Rouge** renforcé : bordures, dégradés d’entête, hover des boutons.
* **Icônes seules** dans le header, avec `aria-label` et `title` (accessibilité).
* **Mobile-first** : grilles responsives, modals adaptatifs.

### Accessibilité & i18n

* Icônes accompagnées de `aria-label`.
* Police système ; contrastes élevés en thème sombre.
* Texte d’interface en **français**.

### Sécurité & confidentialité

* **Aucune donnée envoyée** côté serveur.
* Tout est **local au navigateur** (LocalStorage).
* Les QR encodent uniquement la **ligne d’export** affichée.

---

## Développement

### Prérequis

* Un navigateur moderne.
* Aucun build ni serveur requis (peut s’ouvrir via `file://` ou hébergement statique).

### Lancer en local

1. Cloner le dépôt.
2. Ouvrir `index.html` dans un navigateur.
3. Optionnel : servir en local (par ex. `python -m http.server`) si votre navigateur bloque certains accès aux fichiers.

### Dépendances CDN (incluses)

* Chart.js, QRious, Bootstrap Icons via `<script>`/`<link>` CDN.
  Pour une utilisation **offline**, embarquez des copies locales.

---

## Personnalisation

### Ajouter / retirer une action

1. Modifier le tableau `ACTIONS` dans le script :

   ```js
   const ACTIONS = ["saisie","chute","passage dos","mise en danger","tombé"];
   ```
2. Adapter les longueurs de `counts`, `coeffs`, `active` si vous modifiez le nombre d’actions (initialisation & chargement).
3. Le graphe et l’export suivent automatiquement l’ordre de `ACTIONS`.

### Changer le thème

* Ajuster les **variables CSS** dans `:root` (couleurs, opacités, rayons, etc.).

### Modifier l’export

* Adapter `buildExportLine(sep)` pour changer l’**ordre** ou le **contenu** des champs exportés.
* Modifier la **taille** du QR dans `new QRious({ size: 256, ... })`.

### Couleurs du graphe

* Définies dans `openStats()` via `borderColor` des datasets Bleu/Rouge. Modifier au besoin.

---

## Données & formats

### Ordre des champs exportés (ligne CSV / Excel)

1. `arbitre`
2. `nom_bleu`
3. `nom_rouge`
4. `total_bleu` *(score avec coefficients)*
5. `volume_bleu` *(somme des actions sans coef)*
6. `total_rouge` *(score avec coefficients)*
7. `volume_rouge` *(somme des actions sans coef)*
8. `vol_bleu_saisie`
9. `vol_bleu_chute`
10. `vol_bleu_passage_dos`
11. `vol_bleu_mise_en_danger`
12. `vol_bleu_tombé`
13. `vol_rouge_saisie`
14. `vol_rouge_chute`
15. `vol_rouge_passage_dos`
16. `vol_rouge_mise_en_danger`
17. `vol_rouge_tombé`

> Le séparateur est **choisi par l’utilisateur** : `,` ou `;` (recommandé pour Excel/Numbers selon la locale).

### Exemple de ligne exportée

```
Dupont, Martin, Durand, 14, 7, 18, 8, 3,0,2,1,1, 2,1,3,1,1
```

Ici : arbitre = Dupont, bleu = Martin, rouge = Durand, etc.

### Coefficients par défaut

| Action         | Coefficient par défaut |
| -------------- | ---------------------- |
| saisie         | 1                      |
| chute          | 2                      |
| passage dos    | 3                      |
| mise en danger | 4                      |
| tombé          | 5                      |

> Les coefficients sont **indépendants par lutteur** et modifiables de 0 à 5.

---

## FAQ / Problèmes connus

**Le QR ne s’affiche pas**

* Vérifier que le CDN **QRious** charge correctement (console).
* Essayez d’ouvrir la page via un petit serveur local plutôt que `file://`.
* Si usage offline, servez des **copies locales** des librairies au lieu des CDN.

**Excel ne sépare pas les colonnes**

* Sélectionner le séparateur `;` dans le modal Export pour s’aligner sur les locales FR.
* Ou utiliser l’assistant d’import d’Excel avec séparateur personnalisé.

**Je veux repartir à zéro**

* Bouton **Réinitialiser** (avec confirmation) ou vider le **LocalStorage** du site.

**Les boutons sont grisés**

* L’action peut être **désactivée** dans les **Paramètres** (switch “actif”).

---

## Licence


* Médias/texte : proposer **CC BY 4.0**



Si tu veux, je te fournis aussi un **`README.md` prêt à déposer** dans ton repo (avec une mini capture d’écran en Markdown si tu me donnes un PNG).
