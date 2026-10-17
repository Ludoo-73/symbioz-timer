# symbioz-timer — notes projet

Repo GitHub Pages `Ludoo-73/symbioz-timer`. Réponses et commentaires en français,
HTML/CSS/JS vanilla, pas de dépendances lourdes (sauf SDK Firebase via CDN).

## Fichiers

- `index.html` : landing de présentation de l'app.
- `privacy.html` : politique de confidentialité.
- `tournoi-blackball.html` : page standalone d'organisation de tournois de blackball
  (fichier unique autonome — fonctionne aussi en `file://`).

## Déploiement

Push sur `main` → GitHub Pages rebuild automatiquement (~1 min).
Vérifier le statut : `gh api /repos/Ludoo-73/symbioz-timer/pages/builds/latest --jq .status`.
Un build peut rester coincé en `building` : le relancer via
`gh api -X POST /repos/Ludoo-73/symbioz-timer/pages/builds`.
URL live : `https://ludoo-73.github.io/symbioz-timer/tournoi-blackball.html`.

## Tournoi Blackball — Firebase (suivi live)

- Projet Firebase : `symbioz-tournois` (ID `symbioz-tournois-bdfe7`), compte Google thiveb@gmail.com.
- RTDB : `https://symbioz-tournois-bdfe7-default-rtdb.firebaseio.com` (us-central1).
- Règles publiées (durcies) : lecture/écriture uniquement sous `tournois/$code` —
  le code live (8 caractères, alphabet sans ambiguïtés) fait office de secret.
  Pas d'auth. Validation : un nœud doit avoir la forme `{v, state}` avec `state`
  chaîne < 500 Ko ; enfants parasites refusés (`$other: false`), écritures hors
  `tournois/` refusées. Le delete (`liveRef.remove()`, arrêt du live) reste permis.
- Config web injectée dans `FIREBASE_CONFIG` en haut du script de la page
  (apiKey + databaseURL). Sans ces valeurs la page retombe en mode local pur.
- Données : `tournois/{code}/state` = **`state` sérialisé en chaîne JSON**
  (pas un objet) — Firebase supprime les champs `null` d'un objet brut
  (matchId, s1/s2, group…), ce qui corrompait l'état en retour.
  **Auto-arbitrage** : hôte et participants écrivent tous (dernier écrivain
  gagne) et écoutent tous via `bindViewerListener()` ; `saveState(true)`
  = persistance locale sans repush (anti-boucle). `liveSynced` distingue
  le snapshot null initial (hôte pas encore poussé) du live terminé.
  L'hôte qui arrête le live supprime le nœud → les participants gardent
  leur copie locale.
- Les matchs se lancent **manuellement** (`launchMatch`, bouton "Lancer" sur
  chaque match en attente) — `state.tables` = nombre max de matchs simultanés,
  pas des tables physiques. Chrono en 2 phases : `prepMin` (mise en place) puis
  `RT×minManche` (jeu).
- **Arbre visuel** en bas de l'écran tournoi (`#koTreeSec`, `renderKoTree`) :
  bracket miroir — demi-arbres des deux demi-finales convergent vers la finale
  au centre, consolante en stage séparé. Layout calculé en JS (divs absolus +
  connecteurs SVG), structure déclarative `koSpecMain()`/`koSpecConso()` qui
  reflète les croisements de `maybeAdvanceElim`/`maybeAdvanceKo` — **toute
  modif des pairings doit être reportée là**. Attention : id joueur `0` est
  valide (tests `!== undefined`, pas de truthiness).
- **Clôture** : quand la finale est `done`, une bannière propose « Clôturer et
  archiver » → l'état complet part dans `bbct_tournoi_archive` (10 dernières
  gardées, résumé podium + state pour réexport), le live est arrêté (hôte =
  nœud supprimé) et retour au setup. Liste consultable sur l'écran de création
  (`renderArchives`, overlay lecture seule `viewArchive`).
- Clés localStorage : `bbct_tournoi` (état), `bbct_tournoi_names` (noms),
  `bbct_live` (rôle+code), `bbct_tournoi_archive` (tournois clôturés).

## Test de la logique tournoi

Harnais Node : extraire les `<script>` inline dans un fichier, `eval` avec un stub DOM
minimal (voir historique — `/tmp/test-tournoi.js` était le harnais). Simuler les
combinaisons 6/8/10/12 joueurs × 2/3 tables × poules/élim × consolante/petite finale.
