# Modèle de données Simple Western – documentation des relations

Projet portfolio ELN (eLabFTW) · version 2.2 du 24 novembre 2026 · données fictives

Ce document accompagne le diagramme `eln-simple-western-data-model_V2.drawio`. Il décrit un **modèle conceptuel** : la réalité du laboratoire, indépendamment de l'outil. La dernière section explique comment chaque élément est implémenté dans eLabFTW.

## 1. Objectif

Pouvoir répondre, à partir de n'importe quel résultat, à trois questions :
1. **Avec quoi** a-t-il été obtenu ? (lots d'anticorps, de modules, instrument)
2. **Sur quel échantillon**, préparé comment ? (condition, extraction, préparation)
3. **Qu'est-ce qui a changé** entre deux runs qui donnent des résultats différents ?

## 2. Les cinq familles d'entités

| Famille | Entités | Rôle |
|---|---|---|
| Produits | Primary antibody, Separation module, Detection module | Le catalogue : caractéristiques qui ne changent pas |
| Lots physiques et instrument | Primary antibody lot, Separation module lot, Detection module lot, Instrument | Les tubes, kits et machines réels |
| Chaîne de l'échantillon | Biological condition, Extraction, Sample preparation | Du phénotype jusqu'à ce qui est chargé dans la plaque |
| Exécution | Experiment | Le run Jess, avec son plan de plaque Compass joint |
| Résultats | Result | Ce qui a été mesuré, un enregistrement par pic |

## 3. Les clés

| Clé | Rôle | Exemple |
|---|---|---|
| PK (clé technique) | Numéro attribué automatiquement par la base, unique, jamais modifié ni réutilisé | `id=41` dans l'adresse de la page eLabFTW |
| UK (code lisible unique) | Nom utilisé par les humains, unique, corrigible | SM-FL004, SM-FL004 · lot P-1187, JS-5104 |
| FK (référence) | Pointe vers la PK d'une autre fiche | Le lot pointe vers le module n°3 |

Quand un identifiant officiel existe déjà, il sert directement de UK au lieu d'un code inventé : numéro de catalogue pour les modules, numéro de lot fournisseur pour les lots, numéro de série pour l'instrument.

Pour les lots, un numéro de lot fournisseur n'est unique que pour un produit donné. Le code lisible combine donc le code du produit et le numéro de lot : `SM-FL004 · lot P-1187`. Le lien vers le produit reste ce qui fait foi ; le préfixe sert à lire et à choisir le bon lot au moment de créer un lien.

## 4. Les relations

Lecture : « 1 → 0..n » signifie qu'un objet de gauche est relié à zéro, un ou plusieurs objets de droite, et que chaque objet de droite est relié à exactement un objet de gauche. « n ↔ n » signifie plusieurs de chaque côté.

| De | Vers | Cardinalité | Signification métier |
|---|---|---|---|
| Primary antibody | Primary antibody lot | 1 → 0..n | Un anticorps peut avoir été reçu en plusieurs lots |
| Separation module | Separation module lot | 1 → 0..n | Idem pour les modules de séparation |
| Detection module | Detection module lot | 1 → 0..n | Idem pour les modules de détection |
| Biological condition | Extraction | 1 → 0..n | Un même phénotype peut être extrait plusieurs fois (réplicats biologiques) |
| Extraction | Sample preparation | 1 → 0..n | Un même lysat est dilué à nouveau pour chaque manip |
| Experiment | Sample preparation | 1 → 0..n | Les échantillons sont préparés frais pour un run donné |
| Instrument | Experiment | 1 → 0..n | Chaque run s'exécute sur un instrument |
| Separation module lot | Experiment | 1 → 0..n | Chaque run utilise un lot de module de séparation |
| Primary antibody lot | Experiment | n ↔ n | Un run utilise un ou plusieurs lots d'anticorps ; un lot sert dans plusieurs runs |
| Detection module lot | Experiment | n ↔ n | Idem pour les modules de détection (un par canal en multiplex) |
| Experiment | Experiment | 0..1 → 0..n | Une relance ou un réplicat pointe vers son expérience d'origine |
| Experiment | Result | 1 → 0..n | Un run produit un résultat par pic détecté |
| Result | Result | 0..1 → 0..n | Un résultat normalisé par une protéine de ménage pointe vers le pic de référence |

## 5. Les décisions de conception

| # | Décision | Justification métier |
|---|---|---|
| 1 | Une expérience = un run Jess | Le fichier brut et les consommables se rattachent sans ambiguïté au run |
| 1 bis | Type de run (premier run, relance technique, réplicat biologique) + lien vers l'expérience d'origine | Comparer directement relances et réplicats |
| 2 | Traçabilité au niveau du run : les lots sont liés à l'expérience, et le plan de plaque Compass dit ce qui est dans chaque capillaire | Les capillaires n'ont pas d'attribut propre utile ; le plan de plaque annoté dans Compass fait foi |
| 3 | Plan de plaque = fichier CSV exporté de Compass, joint à chaque expérience | Le plan est rempli dans Compass et réimporté pour un nouveau run ; pas de fiche dédiée |
| 4 | Échantillon en trois niveaux : condition → extraction → préparation | En cas de mauvais signal, on investigue la solubilisation, l'extraction, la dénaturation, la dilution et la déglycosylation |
| 4 bis | Préparation fraîche pour chaque run, rattachée à l'expérience | Les échantillons sont dilués à nouveau pour chaque manip |
| 4 ter | La concentration du lysat stock est notée une seule fois, dans l'extraction | La préparation la retrouve par le lien ; pas de double saisie |
| 5 | Le résultat porte les informations lues dans l'export Compass : n° de capillaire, échantillon, anticorps, canal | Le numéro de capillaire apparaît dans l'export ; pas d'entité Capillaire |
| 6 | Plusieurs pics peuvent être attendus : un enregistrement par pic | Isoformes, formes phosphorylées, multiplex |
| 7 | Normalisation optionnelle, par la protéine totale ou par une protéine de ménage (même capillaire ou capillaire adjacent) | Tous les runs ne sont pas normalisés, certains servent uniquement au screening d'anticorps |
| 8 | Réanalyse dans Compass : les résultats sont remplacés ; l'Analysis log de Compass peut être joint (optionnel) | Compass trace déjà chaque modification avec la date et l'heure |
| 9 | Les liens entre fiches font foi ; le code lisible est une étiquette unique | Corriger un code ne casse jamais la traçabilité |
| 10 | Identifiants officiels comme UK (catalogue, lot fournisseur, numéro de série) | Moins de codes à inventer, et le même nom sur l'étiquette et dans l'ELN |
| 11 | Pas de statut de lot ; pas de quantité restante sauf, en option, pour les lots de modules de séparation | Ne documenter que ce qui sert au labo |
| 12 | Code lisible d'un lot = code du produit · numéro de lot fournisseur | Un numéro de lot n'est unique que pour un produit ; lors d'un lien, eLabFTW propose les fiches par leur titre |
| 13 | Les réglages du run (diluant d'anticorps / tampon de blocage, protocole par défaut ou modifié, modifications apportées) sont des attributs de l'expérience, pas des entités | Ils décrivent un seul run ; le détail complet fait foi dans le fichier de run Compass joint. Le schéma reste lisible pour un stakeholder |
| — | Réactifs de préparation (enzyme de déglycosylation) en attributs texte | Traçables sans multiplier les entités |

## 6. Exemples de questions de traçabilité

**« Dans quelles expériences le lot de module de séparation SM-FL004, lot fournisseur P-1187, a-t-il été utilisé ? »**
Separation module lot → Experiment (liens directs).

**« Pourquoi le réplicat 2 donne-t-il un signal plus faible que le réplicat 1 ? »**
Comparer les deux expériences liées par « origine » : lots utilisés (Experiment), puis préparation, extraction et condition (Experiment → Sample preparation → Extraction → Condition).

**« Ce résultat est-il normalisé, et par quoi ? »**
Result → méthode de normalisation ; si protéine de ménage, Result → pic de référence.

**« Ce run pouvait-il détecter une protéine de 250 kDa ? »**
Experiment → Separation module lot → Separation module → gamme de taille (12-230 kDa) : non.

## 7. Implémentation dans eLabFTW

| Élément du modèle | Implémentation dans eLabFTW |
|---|---|
| Produits, lots, instrument | Catégories de ressources, modèles avec champs structurés, liens ressource → ressource |
| Condition, extraction, préparation | Catégories de ressources reliées entre elles ; la préparation est aussi liée à l'expérience |
| Experiment | Expérience créée depuis le modèle Simple Western, reliée aux lots utilisés |
| Plan de plaque | CSV exporté de Compass, joint à l'expérience |
| Result | Tableau de résultats dans l'expérience, importé par le script d'intégration |
| Liens qui font foi | Section « ressources liées » (identifiant interne d'eLabFTW) |
| Règles (lot inconnu, champ manquant) | Validation dans le script : les champs obligatoires d'eLabFTW ne sont qu'indicatifs |
| Unicité des codes lisibles | Non imposée par eLabFTW : règle d'usage (chercher avant de créer) et contrôle par le script |

**Limite principale :** les lots sont liés au run, pas au capillaire. Dans un test rare comparant deux lots du même anticorps dans un même run, savoir quel lot était dans quel capillaire demande de lire le plan de plaque Compass. C'est un choix assumé de simplicité, à documenter dans la note de conception.

## 8. Hors périmètre

- Décompte automatique des quantités consommées.
- Concentration propre à chaque lot d'anticorps : disponible dans la fiche technique du fournisseur et rarement utilisée ; trop d'information tue l'information.
- Rôle typé des capillaires (ladder, contrôles).
- Traçabilité du lot au niveau du capillaire.
- Versionnage des résultats dans l'ELN (l'historique repose sur Compass et son Analysis log).
