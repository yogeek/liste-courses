# 🛒 Liste de courses partagée

Appli web mobile (un seul `index.html`, sans dépendance) publiée sur GitHub Pages : https://yogeek.github.io/liste-courses/

- Rayons dans l'ordre du magasin (Menu ⋯ → 🗂 Rayons : un ordre par magasin, renommer, ajouter, supprimer), cases à cocher, barre de progression
- Champ unique **ajouter / chercher** : filtre en direct, rayon deviné automatiquement, pas de doublon
- 📚 **Catalogue** : un article acheté retourne au catalogue (« ✅ Terminer la course ») et se remet en un tap
- ⭐ **Réguliers** : un bouton remet tous les articles habituels sur la liste
- 💡 **À racheter** : suggestions d'après le rythme des achats passés
- ⚠️ **Pas trouvé** : reste sur la liste pour la prochaine fois, ou bascule vers un autre magasin
- 🏪 **Magasins** : supermarché, bio, bricolage… la liste n'affiche que ce qu'on trouve là où on est
- 📋 **Coller une liste** : texte libre ou « [ ] article », rangé automatiquement par rayon ; doublons détectés malgré quantités, précisions et fautes de frappe, ressemblances proposées, lignes décochables
- 👥 **Liste partagée** : un lien à envoyer, tout le monde coche la même liste (synchro en temps réel, vérification toutes les 15 s en secours, fusion article par article, marche hors ligne)
- 🔔 **Activité** : notification quand quelqu'un d'autre modifie la liste (« Maman a pris Yaourts ») et historique des derniers changements
- 🆕 **Mises à jour** : bandeau « Nouvelle version → Mettre à jour » (penser à changer `APP_VERSION` à chaque mise en ligne)
- Écran toujours allumé, installable sur l'écran d'accueil

## Premier lancement
Écran d'accueil (coller sa liste, liste d'exemple, liste vide, rejoindre via un lien) puis guide en 4 étapes (aussi dans Menu ⋯ → ❓).
La liste d'exemple est le tableau `EXAMPLE` dans `index.html`.

## Avis des utilisateurs
Menu ⋯ → 💬 Donner mon avis : les messages arrivent dans Firebase sous `/avis` (console Firebase → Realtime Database → Données).

## Stockage partagé
La liste partagée est stockée dans une base Firebase Realtime Database (`DB_URL` dans `index.html`), chemin `/listes/<id>`, mises à jour en temps réel.
Toute personne ayant le lien peut lire et modifier la liste : ne pas y mettre d'infos sensibles.

Règles Firebase (aucune liste ne peut être énumérée ; chaque liste n'est accessible qu'avec son identifiant ; les avis sont en écriture seule) :

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "listes": {
      "$id": {
        ".read": "$id.length >= 16 && $id.length <= 40",
        ".write": "$id.length >= 16 && $id.length <= 40"
      }
    },
    "avis": {
      "$id": {
        ".write": "!data.exists() && newData.child('text').isString() && newData.child('text').val().length <= 2000"
      }
    }
  }
}
```
