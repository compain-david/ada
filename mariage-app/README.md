# 🏆 Dashboard Olympiades — Alix & David

Tableau de bord de soirée pour animer un tournoi par équipes, **100 % local**, sans réseau ni dépendance. Un seul fichier : `dashboard.html`.

## Ce que c'est

- **Fenêtre contrôle** (sur ton laptop) : tu saisis tout, tu pilotes.
- **Fenêtre affichage** (sur le projecteur) : le public ne voit que le spectacle — classement, révélations, podium.
- 10 équipes · 5 épreuves · conversion automatique des rangs en points · animations de révélation.

## Démarrage (le soir J)

1. **Ouvre `dashboard.html`** dans Chrome ou Edge (double-clic).
2. Clique **« 🖥️ Ouvrir l'affichage »** → une 2e fenêtre s'ouvre. Glisse-la sur le **projecteur** et mets-la en plein écran (`F11`).
3. Garde la fenêtre contrôle sur ton **laptop**.

> Si tu fermes l'affichage par accident, reclique « Ouvrir l'affichage » : l'état se resynchronise tout seul.

## Préparer les équipes (en amont)

1. Remplis **`modele-equipes.csv`** dans Excel :
   - colonne `couleur` : un code hex (ex. `#EF4444`) — déjà pré-rempli avec 10 teintes lisibles ;
   - colonnes `membre1…membre7` : les prénoms ;
   - `nom` : tu peux le laisser vide, tu taperas les noms d'équipe le jour J.
2. Dans Excel : **Fichier → Enregistrer sous → CSV**.
3. Dans la fenêtre contrôle : **« 📄 Importer un CSV »** (ou « 📋 Coller un CSV »).
4. Le jour J, tape les **noms d'équipe** directement dans l'éditeur.

## Déroulé d'une épreuve

1. Dans **« Saisie & révélation »**, entre les **rangs 1→10** de l'épreuve (ex aequo autorisé : `1,1,3,…`). L'affichage ne bouge pas.
2. Clique **« 🎬 Révéler l'épreuve »** : l'affichage joue la révélation du 10e au 1er, puis le classement se recompose en animation.
3. Une correction après coup se recalcule **sans rejouer l'animation**.

## Les vues (boutons en haut du contrôle)

| Bouton | Vue |
|---|---|
| 1 · Équipes | Les 10 équipes et leurs membres (accueil) |
| 2 · Classement | Classement général cumulé |
| 3 · Détail | Classement + grille des rangs/points par épreuve |
| 4 · Évolution | Graphe : la position de chaque équipe épreuve par épreuve (rang seul) |
| 5 · Podium | Révélation 3e → 2e → 1er (bouton « Étape podium suivante ») |

## Départage

- **Automatique** : à égalité de points, l'équipe avec le meilleur palmarès (plus de 1res places, etc.) passe devant. Jamais deux équipes à la même place.
- **Manuel** (épreuve de barrage) : dans « Podium & départage », ouvre « Départage manuel du sommet » pour forcer l'ordre des 3 premiers.

## Barème

| Rang | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| Points | 12 | 10 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |

Ex aequo : moyenne des points des rangs occupés (deux 1ers → 11 pts chacun).

## Bon à savoir

- **Sauvegarde automatique** dans le navigateur : un rafraîchissement ne perd rien. Un pastille « ✓ sauvegardé » le confirme.
- **Réinitialiser tout** : bouton en bas (avec confirmation).
- Tout fonctionne **hors ligne** — teste-le une fois sur le vrai laptop + projecteur avant la soirée.

---

Spec détaillée : `specs.md`.
