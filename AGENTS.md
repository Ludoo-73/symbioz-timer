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
- Règles publiées : lecture/écriture uniquement sous `tournois/$code` — le code live
  (8 caractères, alphabet sans ambiguïtés) fait office de secret. Pas d'auth.
- Config web injectée dans `FIREBASE_CONFIG` en haut du script de la page
  (apiKey + databaseURL). Sans ces valeurs la page retombe en mode local pur.
- Données : `tournois/{code}/state` = snapshot complet de `state` poussé par l'hôte
  à chaque `saveState()` ; les spectateurs s'abonnent via `bindViewerListener()`.
- Clés localStorage : `bbct_tournoi` (état), `bbct_tournoi_names` (noms), `bbct_live` (rôle+code).

## Test de la logique tournoi

Harnais Node : extraire les `<script>` inline dans un fichier, `eval` avec un stub DOM
minimal (voir historique — `/tmp/test-tournoi.js` était le harnais). Simuler les
combinaisons 6/8/10/12 joueurs × 2/3 tables × poules/élim × consolante/petite finale.
