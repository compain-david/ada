# Specs — Dashboard Olympiades Alix & David

Version 2 · consolidée, prête pour le développement

> Changements v1 → v2 : bonus retiré · règles de calcul (ex aequo, départage) tranchées · comportement de correction simplifié · architecture deux fenêtres durcie · critères de recette ajoutés · raccourcis clavier figés · couleur d'accent proposée.

---

## 1. Décisions structurantes

| Sujet | Décision | Raison |
|---|---|---|
| Support | Fichier HTML unique en local, ouvert dans le navigateur du laptop | Zéro dépendance réseau, zéro risque de panne le soir J |
| Accès téléphones invités | **Abandonné** *(confirmé)* | Nécessite un backend pour la synchro live, tue l'effet de révélation, détourne les invités du jeu |
| Écrans | Mode écran étendu : contrôle sur le laptop, affichage sur le projecteur *(confirmé)* | Le public ne voit jamais les manipulations |
| Saisie | Rangs 1 à 10 par épreuve, conversion automatique en points | Plus rapide à saisir, aucun calcul mental |
| Opérateur | David seul | Un seul point de vérité |
| Persistance | Sauvegarde automatique à chaque action | Résiste à un rafraîchissement ou une fermeture accidentelle |
| Thème | Dark uniquement | Projection de nuit en extérieur |
| Bonus | **Aucun bonus** | Simplifie le modèle et le calcul, aucun objet orphelin |

---

## 2. Architecture des écrans

Deux fenêtres distinctes ouvertes depuis le même fichier. La fenêtre contrôle ouvre la fenêtre affichage (voir §7 pour le mécanisme).

### 2.1 Fenêtre AFFICHAGE (projetée)

Afficheur pur, sans logique de calcul. Elle ne fait que rendre l'état que la fenêtre contrôle lui envoie. Quatre vues :

**Vue A · Équipes** — Affichée pendant l'accueil. Les 10 équipes avec leur couleur, leur nom et les 7 membres.

**Vue B · Classement général** — Vue par défaut entre les épreuves. Les 10 équipes ordonnées par points cumulés.

**Vue C · Détail par épreuve** — Tableau croisé : 10 équipes en lignes, 5 épreuves en colonnes, rang et points par cellule, plus une colonne total.

**Vue D · Podium final** — Révélation progressive 3 puis 2 puis 1, avec les 7 membres de chaque équipe.

### 2.2 Fenêtre CONTRÔLE (laptop)

Non projetée. Source de vérité unique. Contient :
- Édition des 10 équipes : nom, couleur, 7 membres
- Saisie des rangs par épreuve
- Bouton de révélation d'épreuve
- Navigation entre les vues de la fenêtre affichage
- Correction de toute saisie déjà validée
- Bouton « Départage manuel » (voir §3.3)
- Bouton « Ouvrir l'affichage » et « Réinitialiser » (confirmation requise)

---

## 3. Modèle de données et règles de calcul

```
Equipe {
  id: 1..10
  nom: string          // saisi sur place
  couleur: hex         // choisie à la main par David
  membres: string[7]
}

Epreuve {
  id: 1..5
  nom: string
  rangs: { equipeId: rang }   // 1..10, entiers, uniques par épreuve
  revelee: boolean
}

Etat {
  equipes: Equipe[10]
  epreuves: Epreuve[5]
  vueActive: 'A' | 'B' | 'C' | 'D'
  ordreDepartageManuel: equipeId[] | null   // override optionnel, voir 3.3
}
```

### 3.1 Barème de conversion rang → points

| Rang | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| Points | 12 | 10 | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |

### 3.2 Ex aequo à l'intérieur d'une épreuve

Si plusieurs équipes partagent le même rang, elles reçoivent **la moyenne des points des rangs qu'elles occupent** (et non la moyenne des rangs).

> Exemple. Deux équipes 1res ex aequo occupent les rangs 1 et 2 → points = (12 + 10) / 2 = **11 chacune**. La suivante prend le rang 3 (8 points). Trois équipes ex aequo aux rangs 2-3-4 → (10 + 8 + 7) / 3 = **8,33 chacune**.

Règle générale : pour un groupe de `k` équipes ex aequo occupant les rangs `r … r+k-1`, chaque équipe reçoit `moyenne(points[r], …, points[r+k-1])`. Les points peuvent être décimaux ; ils s'affichent avec au plus une décimale (§6).

### 3.3 Départage du classement général

Deux équipes peuvent finir à **égalité de points cumulés**. Résolution en deux niveaux :

1. **Départage automatique (toujours appliqué)** — comparaison du palmarès rang par rang : l'équipe avec le plus de 1res places passe devant ; à égalité, le plus de 2es places ; etc. Cela garantit que l'affichage n'exhibe **jamais** deux équipes à la même position. Si l'égalité persiste jusqu'au bout (palmarès strictement identiques), l'ordre se fait par `id` d'équipe croissant, de façon déterministe.

2. **Départage manuel (override optionnel)** — pour le podium, David peut prévoir une **épreuve de barrage** le soir J. Le bouton « Départage manuel » permet alors de forcer l'ordre du sommet du classement (`ordreDepartageManuel`). Tant qu'il n'est pas utilisé, le départage automatique fait foi. Le podium (Vue D) n'affiche donc jamais d'ex aequo bloquant, avec ou sans barrage.

> Options de départage envisagées et raison du choix — pour mémoire :
> - **A · Épreuve de barrage** : sudden-death, spectaculaire → retenu comme geste principal (override manuel).
> - **B · Palmarès rang par rang** : 100 % automatique → retenu comme filet de sécurité par défaut.
> - **C · Résultat de la dernière épreuve** : automatique mais arbitraire → écarté.
> - **D · Épreuve reine désignée** : narratif mais à figer en amont → écarté.

### 3.4 Les 5 épreuves

1. Head Shoulder Knee Cup
2. Ballon dans gobelet
3. Plateau à ficelles
4. Kahoot
5. Tir à la corde

---

## 4. Mécanique de révélation

C'est le cœur du dashboard. Un seul moteur d'animation, réutilisé partout, pour garder la cohérence visuelle.

**Étape 1 — saisie masquée.** David saisit les 10 rangs dans la fenêtre contrôle. L'affichage projeté ne bouge pas, il reste sur le classement précédent.

**Étape 2 — déclenchement.** Bouton « Révéler l'épreuve ». La fenêtre affichage passe en mode révélation. Le bouton n'est actif que si les 10 rangs sont saisis et valides (§4 bis).

**Étape 3 — révélation ascendante.** Les résultats de l'épreuve apparaissent du 10e au 1er, une ligne toutes les 1,2 s. Le suspense monte vers le vainqueur.

**Étape 4 — réordonnancement animé.** Le classement général se recompose. Les lignes se déplacent physiquement vers leur nouvelle position, animation de 800 ms avec easing sortant. Chaque équipe affiche sa variation : flèche haut, flèche bas ou tiret, avec le nombre de places gagnées ou perdues.

**Étape 5 — retour au repos.** Le classement général reste affiché jusqu'à l'épreuve suivante.

### 4 bis. Validation de saisie

- Une épreuve n'est révélable que si les **10 rangs sont renseignés**.
- Les rangs doivent couvrir **1 à 10 sans trou** ; un ex aequo est autorisé mais crée alors un rang « sauté » cohérent (ex : 1,1,3,4… si deux équipes partagent la 1re place). Le contrôle signale toute saisie incohérente avant de laisser révéler.
- Tant que la saisie est incomplète ou invalide, le bouton « Révéler » est désactivé et un message indique ce qui manque.

### 4 ter. Correction d'une épreuve déjà révélée — au plus simple

Toute saisie validée reste modifiable depuis la fenêtre contrôle. En cas de correction d'une épreuve **déjà révélée** :
- les totaux et le classement se **recalculent immédiatement** ;
- la mise à jour est **silencieuse** : aucune animation de réordonnancement, aucune re-révélation.

L'animation de réordonnancement (étape 4) est **exclusivement** déclenchée par le geste live « Révéler l'épreuve ». Une correction ne rejoue jamais le spectacle.

---

## 5. Contenu de chaque vue

### Vue A · Équipes
- Grille de 10 cartes, 5 par ligne
- Chaque carte : pastille de couleur, nom de l'équipe en gros, 7 prénoms en dessous
- Utilisée uniquement pendant l'accueil

### Vue B · Classement général
Par ligne :
- Rang, en très gros à gauche
- Pastille de couleur de l'équipe
- Nom de l'équipe
- Points cumulés, en gros à droite (une décimale si nécessaire)
- Indicateur de progression depuis l'épreuve précédente (▲ n / ▼ n / –)
- Le top 3 est visuellement distingué

### Vue C · Détail par épreuve
- 10 lignes, 7 colonnes : équipe, puis les 5 épreuves, puis total
- Chaque cellule affiche le rang et les points
- Les épreuves non encore révélées apparaissent grisées
- Le meilleur rang de chaque colonne est mis en avant
- Accessible pendant la soirée **et** à la fin *(voir §8, à confirmer)*

### Vue D · Podium final
- Révélation en 3 temps : 3e, puis 2e, puis 1er
- Pour chaque équipe révélée : couleur, nom, total, et les 7 membres
- Le 1er occupe tout l'écran à la fin
- Ordre garanti sans ex aequo (§3.3)

---

## 6. Direction artistique

**Base.** Système shadcn en dark mode, recalibré pour la projection. Le shadcn standard est conçu pour une lecture à 60 cm, il est illisible à 5 mètres.

**Ajustements pour la projection**

| Élément | shadcn standard | Ici |
|---|---|---|
| Corps de texte | 14 px | 28 à 32 px |
| Titres | 24 px | 64 à 80 px |
| Rang et score | — | 96 à 120 px |
| Densité | Forte | Aérée, 10 lignes doivent remplir l'écran |
| Bordures | 1 px subtiles | 2 px, contraste renforcé |
| Contraste texte | Doux | Élevé, minimum 7:1 |

**Palette**
- Fond : neutre très sombre, proche de `#09090B`, jamais du noir pur
- Surfaces : `#18181B` avec bordure `#27272A`
- Texte principal : `#FAFAFA`
- Texte secondaire : `#A1A1AA`
- Accent (top 1) : **`#F59E0B` (ambre chaud) proposé** — chaud, lisible en projection, contraste fort sur fond sombre. À valider ou remplacer.
- Couleurs d'équipe : saisies à la main par David, 10 teintes distinctes et saturées. Une palette de secours lisible en projection est fournie si besoin (§8).

**Typographie.** Une seule famille sans-serif géométrique, deux graisses maximum. Chiffres en variante tabulaire pour que les colonnes restent alignées. Les points décimaux s'affichent avec au plus une décimale (ex : `8,3`), les entiers sans décimale (ex : `12`).

**Animation**
- Réordonnancement : transformation CSS, 800 ms, easing sortant
- Apparition des lignes : fondu et translation verticale, décalage de 1,2 s entre chaque
- Aucune animation décorative en dehors des moments de révélation

**Ton.** Sobre. Le nom du couple et la date en petit dans un coin, jamais en gros. Le classement est le sujet.

---

## 7. Comportements techniques

### 7.1 Contraintes générales
- Fichier HTML unique, aucune dépendance externe, aucun CDN
- Sauvegarde automatique dans le stockage local du navigateur à chaque modification
- Chargement automatique de l'état sauvegardé à l'ouverture
- Bouton de réinitialisation totale, protégé par une confirmation
- Toute saisie validée reste modifiable, le classement se recalcule immédiatement (§4 ter)
- Le layout doit tenir en 16:9 sans défilement sur chacune des quatre vues

### 7.2 Architecture deux fenêtres — la plus robuste

Objectif : zéro réseau, zéro dépendance, et une synchro qui ne casse pas en `file://`.

- **Une seule source de vérité** : la fenêtre CONTRÔLE détient l'état complet en mémoire. La fenêtre AFFICHAGE est un afficheur sans état propre : elle ne calcule rien, elle rend ce qu'on lui envoie.
- **Ouverture** : la fenêtre contrôle ouvre l'affichage via `window.open(location.href + '?display', …)`. Les deux instances proviennent du même fichier → même origine, communication directe autorisée.
- **Canal de communication** : lien direct par référence de fenêtre (`postMessage` sur la référence retournée par `window.open`, et `window.opener` côté affichage). On **ne dépend pas** de `BroadcastChannel` ni des événements `storage`, dont le comportement est incertain en `file://` selon les navigateurs.
- **Protocole** : à chaque mutation, le contrôle envoie un **snapshot complet de l'état** + un éventuel événement d'animation (`{type:'reveal', epreuveId}`). L'affichage applique le snapshot et joue l'animation si demandée. Envoyer l'état complet (et non des deltas) rend l'affichage insensible aux messages perdus.
- **Reconnexion** : si la fenêtre affichage est fermée par accident, un clic sur « Ouvrir l'affichage » la rouvre et le contrôle repousse immédiatement le snapshot courant. Aucune perte.
- **Persistance** : écriture dans `localStorage` à chaque mutation, encapsulée dans un `try/catch`. Si `localStorage` est indisponible (`file://` bridé), la soirée fonctionne quand même en mémoire ; seule la reprise après fermeture totale est perdue. Un indicateur discret signale à David si la persistance est active.
- **Navigateur recommandé** : Chrome ou Edge, à **tester sur le laptop réel et le projecteur** au moins une fois avant le soir J (voir recette §9).

### 7.3 Raccourcis clavier (fenêtre contrôle)

| Touche | Action |
|---|---|
| `1` `2` `3` `4` | Basculer l'affichage sur la vue A / B / C / D |
| `Espace` | Révéler l'épreuve en cours de saisie (si valide) |
| `→` | Avancer d'un cran dans une révélation par étapes (podium) |
| `R` | Réinitialiser (demande confirmation) |

Les raccourcis n'agissent que dans la fenêtre contrôle ; la fenêtre affichage ne capte aucune touche.

---

## 8. Points à confirmer avant développement

1. Les 10 couleurs d'équipe : tu les fournis ou je propose une palette lisible en projection.
2. La couleur d'accent : je propose `#F59E0B` (ambre) — tu valides ou tu changes.
3. Départage : je code **le palmarès rang par rang en automatique** + **override manuel** pour l'épreuve de barrage. OK pour toi ?
4. Vue Détail par épreuve : accessible pendant toute la soirée, ou seulement à la fin ?
5. Raccourcis clavier du §7.3 : ça te convient ou tu veux un autre mapping ?

---

## 9. Recette — critères d'acceptation (soir J)

À vérifier une fois avant l'événement, sur le matériel réel :

- [ ] Ouverture du fichier → fenêtre contrôle s'affiche, bouton « Ouvrir l'affichage » fonctionne, affichage apparaît sur le 2e écran.
- [ ] Les 4 vues (A, B, C, D) tiennent en 16:9 **sans défilement**, texte lisible à 5 m.
- [ ] Saisie des 10 rangs d'une épreuve, révélation : lignes du 10e au 1er à 1,2 s, puis réordonnancement animé 800 ms avec flèches de variation.
- [ ] Ex aequo : deux équipes au même rang → points = moyenne attendue (ex : 11 pour un ex aequo 1-2), classement correct.
- [ ] Correction d'une épreuve déjà révélée → totaux et classement mis à jour **sans animation**.
- [ ] Égalité de points en fin de tournoi → aucune position dupliquée à l'écran ; « Départage manuel » force l'ordre voulu.
- [ ] Rafraîchissement accidentel de la fenêtre contrôle → l'état est rechargé depuis le stockage local.
- [ ] Fermeture accidentelle de la fenêtre affichage → réouverture re-synchronise l'état courant.
- [ ] Bouton réinitialisation → demande confirmation, remet à zéro proprement.
- [ ] Aucune requête réseau émise (mode avion : tout fonctionne).
```
