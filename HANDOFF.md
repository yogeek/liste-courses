# HANDOFF — suivi du projet

Journal de passation entre sessions Claude. **À lire en début de session, à mettre à jour en fin de session** (état, décisions, prochaines étapes). Le dépôt est public : rien de personnel ici (pas de liste réelle, pas d'e-mail).

## Contexte utilisateur
- Le propriétaire est ingénieur cloud/DevOps ; les utilisateurs finaux sont sa famille puis d'autres familles testeuses (non techniques). Réponses en français, simples, orientées usage en magasin.
- Il teste souvent l'appli **en plein supermarché** : privilégier des correctifs rapides, sûrs, et toujours testés avant mise en ligne.
- Flux git : développer sur la branche de session, pousser sur cette branche **et** sur `main` (GitHub Pages publie `main`). Toujours bumper `APP_VERSION`.

## État actuel (2026-11-21)
- En ligne : https://yogeek.github.io/liste-courses/ — `APP_VERSION = "2026-11-21.1"`, SW `courses-v8`.
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

## À vérifier / en attente côté utilisateur
- Les **règles Firebase** du README (avec le chemin `avis`) doivent être publiées par l'utilisateur dans la console. Symptôme si mal collées : « ⚠️ accès refusé (règles) » en haut à droite → lui demander ce qu'il a collé.
- Retours des familles testeuses (lire `/avis` dans la console Firebase).

## Idées proposées, non lancées (attendre son feu vert)
- Recherche toujours sur tous les magasins, avec badge du magasin sur les résultats.
- **Lot 2** : ordre des rayons appris d'après l'ordre réel des coches ; « mode magasin » (gros boutons, rayon courant) ; « qui prend quoi » (attribuer un article à une personne).
- **Lot 3** : backend Cloudflare (Workers/D1) à la place de Firebase ; scan code-barres + Open Food Facts ; saisie vocale / IA ; tags NFC ; notifications push.

## Pièges connus
- Firebase supprime les tableaux vides et peut renvoyer des tableaux sous forme d'objets → passer par `arr()` / `cleanDoc`.
- Firebase et jsonblob sont bloqués depuis le bac à sable : tests uniquement avec des routes Playwright mockées (voir `CLAUDE.md`).
- iOS zoome sur les champs < 16px.
- Toute modification hors `touch()` casse la fusion entre appareils.

## Journal des sessions
- **2026-11-20 → 21** : création du projet, lots 0–1, Firebase, activité, mise à jour auto, ouverture multi-familles, doublons, purge de l'historique, CLAUDE.md/HANDOFF.md.
