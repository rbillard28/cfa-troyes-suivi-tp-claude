[README.md](https://github.com/user-attachments/files/29767600/README.md)
# CFA Troyes — Suivi des TP · Base de données Google Sheet + Interface web

Ce projet transforme `CFA_Troyes_Suivi_TP_V2.xlsx` en base de données pilotée
par **Google Sheets**, exposée via une **API Google Apps Script**, et
consultée/éditée depuis une **interface web** hébergée sur GitHub + Netlify.

```
Navigateur (Netlify) --fetch--> Apps Script Web App --lit/écrit--> Google Sheet
      index.html                     Code.gs                  (la base de données)
```

## Pourquoi cette architecture ?

Le classeur original contient déjà des formules qui calculent les coûts
(coût/apprenant, coût total du TP, amortissement du matériel réutilisable...).
**Le script ne recalcule rien lui-même** : il lit/écrit uniquement les
cellules de saisie (TP réalisé, effectif, désignation, quantité...) et laisse
Google Sheets recalculer automatiquement les colonnes formulées. C'est le
choix le plus fiable : aucun risque de divergence entre la logique du
tableur et celle de l'appli web.

---

## Étape 1 — Préparer le Google Sheet

1. Allez sur [sheets.google.com](https://sheets.google.com) → **Fichier > Importer**
   → onglet **Importer** → sélectionnez `CFA_Troyes_Suivi_TP_V2.xlsx`.
   Choisissez **"Remplacer la feuille de calcul"** pour garder un seul fichier
   avec tous les onglets (Projection, Suivi Global, Nomenclatures_TP,
   Parametres, Suivi consommable, Suivi dépense matériel).
2. Une fois importé, ouvrez chaque onglet et vérifiez que les formules
   affichent bien des valeurs (pas de `#REF!` ou `#NAME?`). Les tableaux
   structurés Excel ("Tables") sont convertis par Google Sheets en plages
   normales : les formules `INDEX/MATCH` qui les référencent continuent de
   fonctionner, mais vérifiez tout de même l'onglet **Suivi Global**
   (colonnes J/K) et l'onglet **Nomenclatures_TP** (colonne K).
3. **Important** : `Code.gs` repère les lignes de données grâce à leur
   position (numéros de ligne/colonne indiqués dans `CONFIG` en haut du
   fichier). Si l'import Google Sheets décale des lignes ou colonnes par
   rapport au fichier original, ouvrez `Code.gs` et ajustez les valeurs de
   `CONFIG` en conséquence (elles sont commentées).
4. Renommez le fichier si besoin, par exemple **"CFA Troyes — Suivi TP (BDD)"**.

## Étape 2 — Déployer l'API Apps Script

1. Dans le Google Sheet : **Extensions > Apps Script**.
2. Supprimez le contenu par défaut de `Code.gs` et collez le contenu du
   fichier `Code.gs` fourni ici.
3. Cliquez sur l'icône ⚙ **Paramètres du projet**, cochez
   *"Afficher le fichier manifeste appsscript.json dans l'éditeur"*, puis
   ouvrez `appsscript.json` et remplacez son contenu par celui du fichier
   `appsscript.json` fourni.
4. Cliquez sur **Déployer > Nouveau déploiement**.
   - Type : **Application Web**
   - Description : `API Suivi TP v1`
   - Exécuter en tant que : **Moi**
   - Qui a accès : **Tout le monde**
5. Autorisez les permissions demandées (l'app accède au classeur en votre
   nom). Copiez l'**URL de déploiement** (se termine par `/exec`).

> ⚠️ À chaque fois que vous modifiez `Code.gs`, utilisez **Déployer > Gérer
> les déploiements > ✏️ Modifier > Nouvelle version** pour que les
> changements soient pris en compte par l'URL existante (sinon l'URL reste
> figée sur l'ancienne version du code).

### Tester l'API

Collez l'URL `/exec` suivie de `?action=dashboard` dans un navigateur : vous
devez voir un JSON `{"ok":true,"data":{...}}`.

## Étape 3 — Mettre le code en ligne sur GitHub

```bash
cd cfa-troyes-tp
git init
git add index.html README.md
git commit -m "Interface web Suivi TP CFA Troyes"
git branch -M main
git remote add origin https://github.com/<votre-compte>/cfa-troyes-suivi-tp.git
git push -u origin main
```

> Ne poussez pas `Code.gs` / `appsscript.json` sur un repo public si vous
> préférez garder la logique de l'API privée — ils ne sont utiles que dans
> l'éditeur Apps Script, pas sur GitHub. Vous pouvez les garder dans un
> dossier séparé `/apps-script` du repo si vous voulez les versionner.

## Étape 4 — Déployer sur Netlify

1. [app.netlify.com](https://app.netlify.com) → **Add new site > Import an
   existing project** → connectez votre compte GitHub → sélectionnez le repo.
2. Build command : *(laisser vide)* — Publish directory : `.`
   *(c'est un site 100% statique, aucun build n'est nécessaire)*
3. Déployez. Netlify vous donne une URL du type
   `https://cfa-troyes-suivi-tp.netlify.app`.

## Étape 5 — Connecter l'interface au Google Sheet

1. Ouvrez le site Netlify.
2. Cliquez sur l'icône ⚙ en bas de la barre latérale.
3. Collez l'URL Apps Script (`.../exec`) obtenue à l'étape 2 → **Enregistrer
   & tester**.
4. Le voyant en bas de la barre latérale passe au vert (`CONNECTÉ`).

> L'URL est mémorisée dans le navigateur (`localStorage`), donc chaque
> utilisateur ne la saisit qu'une seule fois sur son poste. Si vous préférez
> qu'elle soit préconfigurée pour tout le monde, ouvrez `index.html`,
> repérez la constante `HARD_CODED_API_URL` en haut du `<script>` et collez
> l'URL directement dedans avant de pousser sur GitHub.

---

## Ce que fait chaque page

| Page | Onglet source | Actions disponibles |
|---|---|---|
| **1. Accueil** | Calculé à partir de *Suivi Global* + *Nomenclatures_TP* + *Parametres* | Lecture seule : taux de réalisation, coût total, avancement par section, coût par type de TP |
| **2. Suivi global** | `Suivi Global` | Sélectionner le TP réalisé et l'effectif réel par module ; le coût se recalcule automatiquement (formules du Sheet) |
| **3. Nomenclature TP** | `Nomenclatures_TP` | Ajouter/modifier/supprimer un composant d'un TP existant ; créer un nouveau TP avec sa liste de composants |
| **4. Catalogue matériel** | `Parametres` | Ajouter/modifier/supprimer un article (désignation, catégorie, prix, durée de vie) ; consulter la légende des codes et les paramètres annuels |

## Limites connues / points d'attention

- **Nouveau composant** : la désignation saisie doit correspondre
  exactement à un article déjà présent dans le catalogue (`Parametres`),
  car les colonnes *Type* et *Unitaire/TP* de `Nomenclatures_TP` sont des
  formules qui recherchent l'article par désignation. Un champ
  autocomplete (liste déroulante) est fourni dans l'interface pour éviter
  les erreurs de saisie.
- **Nouvel article catalogue avec matériel réutilisable** : le calcul du
  *"Prix par TP"* (amortissement) dépend de formules existantes dans
  `Parametres`. Le script copie automatiquement ces formules depuis la
  ligne précédente lors de l'ajout d'un article ; vérifiez le résultat dans
  le Sheet après ajout d'un article réutilisable.
- **Tableau de synthèse par section** (bas de l'onglet *Suivi Global*,
  lignes 121-131 dans le fichier d'origine) n'est pas modifié
  automatiquement lors de la création d'un nouveau TP : il reste tel quel,
  car il s'agit d'un tableau de reporting, pas de saisie.
- Le script suppose que la structure des onglets (lignes d'en-tête,
  colonnes) reste identique à celle du fichier fourni. Si vous réorganisez
  fortement les onglets, mettez à jour l'objet `CONFIG` en haut de
  `Code.gs`.

## Fichiers fournis

- `Code.gs` — API Apps Script (lecture/écriture du Google Sheet)
- `appsscript.json` — manifeste de déploiement Apps Script
- `index.html` — interface web complète (4 pages, autonome, aucune
  dépendance externe hors polices Google Fonts)
- `README.md` — ce guide
