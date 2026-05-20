# ⚔ Chroniques — Planificateur de Jeux de Rôle

Application web mono-fichier (`index.html`) pour organiser vos sessions de JDR.  
Backend 100 % Firebase (Auth + Firestore). Hébergement via GitHub Pages (gratuit).

---

## Fonctionnalités

| Rôle | Fonctionnalités |
|------|----------------|
| **MJ** | Créer / modifier / supprimer des événements, définir des dates et horaires, inviter des joueurs par pseudonyme ou par code, consulter le résumé des présences |
| **Joueur** | Rejoindre un événement via code, répondre aux dates (✅ Oui / ❌ Non / 🤔 Peut-être / — Effacer), visualiser le calendrier |
| **Commun** | Création de compte, modification du profil, changement de rôle, suppression du compte |

---

## Étape 1 — Créer le projet Firebase

1. Allez sur **[console.firebase.google.com](https://console.firebase.google.com)** et cliquez **Ajouter un projet**.
2. Donnez-lui un nom (ex : `chroniques-jdr`) et suivez l'assistant.

### Activer l'authentification

- Menu latéral → **Authentication** → **Commencer**
- Onglet **Sign-in method** → activez **E-mail/Mot de passe**

### Créer la base Firestore

- Menu latéral → **Firestore Database** → **Créer une base de données**
- Choisissez **Mode production** (vous ajusterez les règles ensuite)
- Région conseillée : `europe-west3 (Frankfurt)`

### Règles de sécurité Firestore

Dans **Firestore → Règles**, remplacez le contenu par :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Utilisateurs : lecture publique (pour chercher par username), écriture par soi-même
    match /users/{userId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.auth.uid == userId;
      allow update, delete: if request.auth != null && request.auth.uid == userId;
    }

    // Événements
    match /events/{eventId} {
      // Lecture : créateur ou joueur invité
      allow read: if request.auth != null &&
        (resource.data.createdBy == request.auth.uid ||
         request.auth.uid in resource.data.invitedUids);

      // Création : tout utilisateur authentifié (MJ)
      allow create: if request.auth != null;

      // Mise à jour : créateur (modif complète) OU joueur invité (uniquement ses réponses)
      allow update: if request.auth != null && (
        resource.data.createdBy == request.auth.uid ||
        request.auth.uid in resource.data.invitedUids
      );

      // Suppression : créateur uniquement
      allow delete: if request.auth != null && resource.data.createdBy == request.auth.uid;
    }
  }
}
```

### Récupérer la configuration Firebase

- Menu latéral → **Paramètres du projet** (icône ⚙️) → onglet **Général**
- Faites défiler jusqu'à **Vos applications** → cliquez l'icône `</>` (Web)
- Nommez l'app et copiez l'objet `firebaseConfig`

---

## Étape 2 — Configurer index.html

Ouvrez `index.html` et repérez ce bloc (~ligne 480) :

```javascript
const firebaseConfig = {
  apiKey:            "VOTRE_API_KEY",
  authDomain:        "votre-app.firebaseapp.com",
  projectId:         "votre-app-id",
  storageBucket:     "votre-app.appspot.com",
  messagingSenderId: "000000000000",
  appId:             "1:000000000000:web:000000000000000000000000"
};
```

Remplacez chaque valeur par celles copiées depuis la console Firebase.

---

## Étape 3 — Publier sur GitHub Pages

### 3a. Créer le dépôt GitHub

```bash
git init
git add index.html README.md
git commit -m "Initial commit — Chroniques JDR"
```

Sur **github.com** → **New repository** → nommez-le (ex : `chroniques-jdr`) → créez-le **public**.

```bash
git remote add origin https://github.com/VOTRE_PSEUDO/chroniques-jdr.git
git branch -M main
git push -u origin main
```

### 3b. Activer GitHub Pages

- Sur GitHub : **Settings** → **Pages**
- **Source** : `Deploy from a branch` → branche `main` → dossier `/ (root)`
- Cliquez **Save**

Après ~1 minute, votre app est en ligne à l'adresse :  
`https://VOTRE_PSEUDO.github.io/chroniques-jdr/`

### 3c. Autoriser le domaine dans Firebase

- Console Firebase → **Authentication** → **Settings** → **Domaines autorisés**
- Cliquez **Ajouter un domaine** → entrez `VOTRE_PSEUDO.github.io`

---

## Étape 4 — Premiers pas dans l'app

1. Ouvrez l'URL GitHub Pages
2. Créez un compte **MJ** (Meneur de jeu)
3. Créez un événement, ajoutez des dates/horaires
4. Copiez le **code** affiché sur la carte de l'événement
5. Créez un second compte **Joueur** (autre onglet / navigateur)
6. Dans l'onglet **Rejoindre**, collez le code
7. Le joueur peut maintenant répondre aux dates depuis le **Calendrier**
8. Le MJ voit le **Résumé** avec toutes les présences

---

## Structure du fichier

```
index.html
├── <style>          — CSS complet (thème sombre, variables)
├── #auth-screen     — Connexion / Inscription
├── #main-screen     — App principale (header + navigation)
│   ├── panel-calendar   — Calendrier mensuel
│   ├── panel-events     — Gestion des événements (MJ)
│   ├── panel-join       — Rejoindre par code (Joueur)
│   └── panel-profile    — Profil utilisateur
├── Modals           — Création/édition, détail, confirmation
└── <script type="module">  — Firebase + toute la logique JS
```

---

## Mises à jour futures

Pour mettre à jour l'app après modification :

```bash
git add index.html
git commit -m "Description de la modification"
git push
```

GitHub Pages se met à jour automatiquement en ~1 minute.

---

## Dépannage

| Problème | Solution |
|----------|----------|
| Écran blanc | Ouvrez la console (F12) — probablement la config Firebase incorrecte |
| "Permission denied" Firestore | Vérifiez les règles de sécurité (Étape 1) |
| Connexion impossible | Vérifiez que E-mail/Mot de passe est activé dans Authentication |
| Domaine non autorisé | Ajoutez `VOTRE_PSEUDO.github.io` dans Authentication → Domaines |
