# Rapport de débogage — React Developer Tools

## 1. Mise en place de l'application

L'application fournie (`app-avec-bugs/`) est une application `create-react-app`
simple composée de trois composants :

- `Counter.js` — un compteur avec état local (`useState`).
- `UserList.js` — une liste rendue via `.map()`.
- `User.js` — un composant enfant qui reçoit `name`, `email`, `role` en props.

Installation et lancement :

```bash
cd app-avec-bugs
npm install
npm start
```

## 2. Installation de React Developer Tools

Extension installée depuis le Chrome Web Store / Firefox Add-ons : « React
Developer Tools ». Une fois installée, deux nouveaux onglets apparaissent
dans les outils de développement du navigateur : **Components** et
**Profiler**.

## 3. Inspection de l'arbre des composants

Dans l'onglet **Components**, l'arbre affiche :

```
App
 ├─ Counter
 └─ UserList
     ├─ User
     ├─ User
     └─ User
```

En sélectionnant chaque nœud, le panneau de droite affiche ses `props` et
son `hooks` (état). C'est ce panneau qui a permis d'identifier les trois
problèmes ci-dessous.

## 4. Problèmes identifiés

### Bug 1 — État qui ne se met pas à jour (`Counter.js`)

En cliquant sur le bouton « Incrémenter », la valeur affichée à l'écran
restait à 0. Dans l'onglet Components, le hook `State` de `Counter`
confirmait que la valeur ne changeait jamais après un clic.

**Cause** : la fonction `increment` mutait directement la variable `count`
au lieu d'appeler `setCount`. React ne peut détecter un changement d'état
que via son setter ; une mutation directe ne déclenche aucun nouveau rendu.

```js
// Avant
const increment = () => {
  count = count + 1;
};
```

### Bug 2 — Accessoire manquant (`User.js` via `UserList.js`)

Le troisième utilisateur affichait « Rôle : » suivi de rien. En sélectionnant
ce nœud `User` dans l'onglet Components, le panneau `props` montrait
`role: undefined`.

**Cause** : l'objet correspondant dans `users.js` ne définissait pas
d'attribut `role`, et rien dans `User.js` ne prévoyait de valeur de repli.

### Bug 3 — Comportement inattendu de la liste (`UserList.js`)

La console affichait l'avertissement :

```
Warning: Each child in a list should have a unique "key" prop.
```

En inspectant le nœud `UserList` dans l'arbre des composants, les trois
enfants `User` apparaissaient sans identifiant de clé, ce qui expose
l'application à un mauvais réordonnancement du DOM si la liste venait à
changer (tri, suppression, etc.).

**Cause** : le `.map()` ne fournissait pas de prop `key` à chaque élément
rendu.

## 5. Outils utilisés pour le diagnostic

- **Onglet Components** : inspection des `props` et de l'état (`hooks`) de
  chaque nœud pour confirmer les valeurs réellement reçues par chaque
  composant, plutôt que de supposer leur contenu à la lecture du code.
- **Console du navigateur** : lecture des avertissements React (clé de
  liste manquante).
- **Interaction manuelle** : clics répétés sur le bouton du compteur en
  observant en temps réel le panneau `State` pour confirmer que le rendu ne
  suivait pas les clics.

## 6. Corrections apportées

| Bug | Fichier | Correction |
|---|---|---|
| État non mis à jour | `Counter.js` | Remplacement de la mutation directe par `setCount((prev) => prev + 1)` |
| Accessoire manquant | `users.js`, `User.js` | Ajout du rôle manquant dans les données, et `User.defaultProps = { role: "Rôle non renseigné" }` en filet de sécurité |
| Clé de liste absente | `UserList.js` | Ajout de `key={user.id}` sur chaque `<User />` rendu par `.map()` |

La version corrigée se trouve dans `app-corrigee/`.

## 7. Vérification post-correction

- Le clic sur « Incrémenter » met désormais à jour l'affichage à chaque
  clic, et le hook `State` dans React DevTools reflète la valeur courante.
- Les trois utilisateurs affichent un rôle valide (aucun `undefined` à
  l'écran ni dans le panneau `props`).
- L'avertissement de clé manquante a disparu de la console.

Tous les problèmes identifiés lors de l'inspection initiale ont été résolus
et vérifiés dans `app-corrigee/`.
