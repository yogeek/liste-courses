# HANDOFF — suivi du projet

Journal de passation entre sessions Claude. **À lire en début de session, à mettre à jour en fin de session** (état, décisions, prochaines étapes). Le dépôt est public : rien de personnel ici (pas de liste réelle, pas d'e-mail).

## Contexte utilisateur
- Le propriétaire est ingénieur cloud/DevOps ; les utilisateurs finaux sont sa famille puis d'autres familles testeuses (non techniques). Réponses en français, simples, orientées usage en magasin.
- Il teste souvent l'appli **en plein supermarché** : privilégier des correctifs rapides, sûrs, et toujours testés avant mise en ligne.
- Flux git : développer sur la branche de session, pousser sur cette branche **et** sur `main` (GitHub Pages publie `main`). Toujours bumper `APP_VERSION`.

## État actuel (2026-11-21)
- En ligne : https://yogeek.github.io/liste-courses/ — `APP_VERSION = "2026-11-21.4"`, SW `courses-v8`.
- Partage via Firebase RTDB (`DB_URL` dans `index.html`), temps réel SSE + polling 15 s. Un lien de partage familial est déjà utilisé en production : **ne jamais casser le format `#l=<id>` ni le schéma du doc** (sinon prévoir une migration dans `normalize`/`cleanX`).
- Historique git réécrit le 2026-11-21 pour purger une liste personnelle (commit unique `7089723`). Si suppression totale voulue : demande au support GitHub (« Remove sensitive data ») pour les anciens SHA `30a4c07`, `5e392f1`, `5c115fe` — laissé à l'utilisateur.

## Fait
- Lot 0 : appli mobile par rayons, PWA, hors ligne, écran allumé, sauvegarde/restauration.
- Lot 1 : catalogue, réguliers ⭐, « à racheter » 💡 (rythme d'achat), « pas trouvé » ⚠️, magasins multiples, coller une liste avec classement auto.
- Partage Firebase (remplace jsonblob, qui échouait en « fetch failed »), fusion LWW par élément, annuler multi-appareils.
- Mise à jour auto (bandeau version), fil d'activité 🔔 (« X a pris Y »), nom par appareil.
- Feedback visible du bouton « + » (ligne d'indice, défilement, toast au-dessus du clavier).
- Ouverture à d'autres familles : accueil neutre (coller / exemple / vide / rejoindre), guide 4 étapes, 💬 avis → Firebase `/avis`.
- Doublons souples (quantités, précisions, fautes de frappe) dans l'ajout et l'aperçu du collage, avec choix par ligne.
- `CLAUDE.md` + ce fichier.
- Publication Pages par GitHub Actions (`.github/workflows/pages.yml`) : seuls les fichiers de l'appli sont en ligne.
- Revue complète (2026-11-21, v`.2`), bugs corrigés :
  - **Envoi inutile à chaque synchro** : Firebase renvoie les clés triées, l'appareil non → la comparaison voyait toujours une différence et renvoyait toute la liste (toutes les 15 s, par chaque appareil). Comparaison désormais à clés triées (`stable`).
  - **« Annuler » écrasait les changements des autres** faits entre-temps (articles ajoutés par un autre supprimés, coches perdues) : `restore` ne revient plus que sur les éléments modifiés par l'action.
  - Annuler un ajout / « Tout ajouter » (💡) / coller une liste pouvaient agir sur une copie périmée après une synchro → recherche par id.
  - Lien de partage erroné/incomplet : la liste locale était envoyée à cette adresse inconnue. Maintenant : message, retour à la situation précédente ; ids hors 16–40 caractères refusés.
  - Temps réel : la connexion SSE n'était jamais recréée si le navigateur l'avait fermée définitivement.
  - Magasin sélectionné supprimé sur un autre appareil → liste vide « Rien à acheter » ; retour à « tous ».
  - Catalogue : champ de filtre recréé à chaque lettre (clavier qui se ferme sur iPhone) ; articles d'un rayon inconnu invisibles.
  - Recherche « ??? » affichait tout ; emojis de rayons/magasins (données partagées) non échappés dans le HTML.

- **Gérer les rayons** (v`.3`, demandé par le propriétaire) : Menu ⋯ → 🗂 Rayons : monter/descendre (renumérote `o`), renommer + emoji, ajouter, supprimer (articles → « Divers »). Un rayon renommé garde sa clé de rangement auto dans le champ optionnel `k` des rayons (`catForKey` cherche d'abord `k`, sinon le nom), sinon l'ancien rayon serait recréé au prochain ajout.
- **Ordre des rayons par magasin** (v`.4`) : dans 🗂 Rayons, onglets « Par défaut » + un par magasin (ouvert sur le magasin courant). Un magasin réglé porte `co = {idRayon: rang}` (champ optionnel du magasin, absent = ordre par défaut, ↩️ pour y revenir) ; `catsFor(shopId)` donne l'ordre utilisé par la liste et le catalogue ; un rayon absent de `co` va à la fin. Nom/emoji/ajout/suppression restent communs à tous les magasins.

## À vérifier / en attente côté utilisateur
- Passer **Settings → Pages → Source** sur « GitHub Actions » (sinon Pages continue de publier toute la branche), puis relancer le workflow « Publication de l'appli » si son premier passage a échoué. Vérifier ensuite que `/HANDOFF.md` renvoie 404 et que l'appli marche.
- Les **règles Firebase** du README (avec le chemin `avis`) doivent être publiées par l'utilisateur dans la console. Symptôme si mal collées : « ⚠️ accès refusé (règles) » en haut à droite → lui demander ce qu'il a collé.
- Retours des familles testeuses (lire `/avis` dans la console Firebase).

## Idées proposées, non lancées (attendre son feu vert) — priorités revues le 2026-11-21
1. ~~Gérer les rayons~~ et ~~ordre par magasin~~ : faits (v`.3`, v`.4`). « Ordre appris » seulement si l'ordre manuel ne suffit pas.
2. **Recherche sur tous les magasins** avec badge du magasin : quasi-bug actuel (en vue « Bio », taper « lait » dit « Déjà sur la liste » mais ne le montre pas).
3. **Filets de sécurité** : copie locale avant de rejoindre une liste (rejoindre remplace la liste de l'appareil) ; règle Firebase `newData.exists()` pour qu'aucune écriture ne puisse effacer une liste ; à terme, écriture conditionnelle (ETag Firebase) pour fermer la petite fenêtre où deux envois simultanés s'écrasent (aujourd'hui rattrapée à la synchro suivante).
4. Lot 2 restant : « qui prend quoi » → attendre une demande des familles. « Mode magasin » → probablement superflu (👁 masquer + replier les rayons existent) ; à réévaluer avec les retours.
5. Lot 3 revu : **backend Cloudflare déconseillé** tant que l'offre gratuite Firebase suffit (largement pour quelques familles) ; **saisie vocale abandonnée** (la dictée du clavier iOS/Android le fait déjà) ; **NFC abandonné** ; scan code-barres faible valeur pour une liste de courses ; notifications push coûteuses (SDK + appli installée sur iPhone) → seulement si demandé.

## Pièges connus
- Firebase supprime les tableaux vides et peut renvoyer des tableaux sous forme d'objets → passer par `arr()` / `cleanDoc`.
- Firebase et jsonblob sont bloqués depuis le bac à sable : tests uniquement avec des routes Playwright mockées (voir `CLAUDE.md`).
- iOS zoome sur les champs < 16px.
- Toute modification hors `touch()` casse la fusion entre appareils.

## Journal des sessions
- **2026-11-21 (ordre par magasin)** : ordre des rayons propre à chaque magasin.
- **2026-11-21 (rayons)** : écran 🗂 Rayons (ordre, noms, ajout, suppression).
- **2026-11-21 (revue)** : revue complète, 9 bugs corrigés (voir « Fait »), priorités revues ; tests Playwright avec un faux Firebase qui trie les clés.
- **2026-11-20 → 21** : création du projet, lots 0–1, Firebase, activité, mise à jour auto, ouverture multi-familles, doublons, purge de l'historique, CLAUDE.md/HANDOFF.md, publication de l'appli seule via Actions.
