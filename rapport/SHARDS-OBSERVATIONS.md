# Shards — observations du compte et pistes d’audit

> **Statut : observation préliminaire — aucune vulnérabilité confirmée.**  
> Cette note consigne une réponse observée lors d’une navigation authentifiée normale sur le compte de test/personnel de l’auditeur. Les valeurs sont un instantané et peuvent évoluer.

## 1. Valeurs observées

Source : réponse HTTP 200 de l’appel RPC `my_shards` sur l’API Yapin.

| Champ | Valeur observée |
|---|---:|
| `balance` | `60` |
| `earned` | `460` |
| `level` | `4` |
| `streak` | `7` |
| `frozen` | `false` |
| `next_cost` | `100` |
| `invites.pending` | `3` |

Autres éléments présents dans la même réponse :
- `today.paid = 25`
- `today.daily = true`
- `today.minutes = 25`
- `invites.paid = 0`
- `boost = 1`
- `event = null`

Configuration renvoyée :
- `daily = 5`, `streak_bonus = 30`, `invite = 100`
- `activity_cap = 25`, `per_block = 5`, `block_minutes = 5`
- `level_step = 100`, `level_max = 100`
- `price_min = 25`, `price_max = 5000`
- `streak_days = 7`
- `invite_cap_30d = 10`, `invite_age_days = 7`, `invite_active_days = 3`
- `min_age_days = 1`, `buyer_age_days = 7`, `seller_age_days = 5`
- `artist_share = 70`

## 2. Interprétation prudente

- `balance = 60` et `earned = 460` ne sont pas nécessairement incohérents : si `earned` représente le cumul des gains et `balance` le solde disponible après dépenses, l’écart peut être normal.
- `level = 4` et `next_cost = 100` ne suffisent pas à démontrer une erreur de calcul ou de progression.
- `streak = 7`, `today.daily = true` et `today.paid = 25` décrivent un état de progression, sans prouver qu’une récompense peut être obtenue plusieurs fois.
- `invites.pending = 3` peut simplement représenter trois invitations en attente. Il faut connaître les conditions de validation et de paiement avant de conclure à un problème.
- `frozen = false` indique seulement que le compte n’est pas marqué comme gelé dans cette réponse.

## 3. Pistes d’audit à vérifier

Ces éléments sont des **hypothèses de vérification**, pas des failles établies :

1. **Calcul du solde côté serveur** — vérifier que les gains, dépenses et remboursements sont calculés et validés côté serveur, et qu’un client ne peut pas imposer directement un nouveau solde.
2. **Cohérence du ledger** — comparer, sur les seules données autorisées du compte audité, les écritures du journal avec l’évolution du solde et du cumul des gains. Clarifier la sémantique de chaque type d’écriture.
3. **Isolation des données** — vérifier dans les politiques RLS et les fonctions RPC que chaque utilisateur ne peut consulter que ses propres données privées.
4. **Récompenses et invitations** — vérifier que les conditions d’âge, d’activité, de plafond et d’éligibilité sont contrôlées côté serveur et qu’une même récompense ne peut pas être créditée plusieurs fois.
5. **Atomicité et idempotence** — vérifier que deux demandes simultanées ou répétées ne peuvent pas créer de double crédit ou de dépense incohérente.
6. **Progression et prix** — documenter précisément les règles de niveau, de coût suivant et les limites configurées.

## 4. État des preuves

À ce stade, la réponse de `my_shards` confirme uniquement que l’API a renvoyé les valeurs ci-dessus pour la session observée. Elle ne prouve ni accès à des données d’autres comptes, ni possibilité de modifier le solde, ni contournement des conditions de récompense.

Aucune requête de modification, tentative de falsification de récompense ou test sur le compte d’un tiers n’est documenté ici. Les prochaines vérifications doivent rester passives ou être réalisées dans un périmètre de test explicitement autorisé, sans affecter d’autres utilisateurs.

## 5. Conclusion provisoire

**Risque potentiel à investiguer :** logique métier des Shards (solde, ledger, invitations, récompenses et progression).  
**Résultat actuel :** aucune vulnérabilité confirmée ; il manque notamment les règles serveur et les contrôles d’accès pour conclure.
