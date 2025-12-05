# Confirmation officielle des incompatibilités entre VPN IPsec Route-Based et Policy-Based

L'"incompatibilité officielle" réside dans le fait que ces deux types de VPN utilisent des méthodes fondamentalement différentes pour négocier les **Associations de Sécurité (SA) de Phase 2**.

Bien que des constructeurs comme Cisco, Palo Alto et Fortinet permettent de connecter les deux, ils traitent tous cela comme un scénario spécial d'"interopérabilité" nécessitant des contournements spécifiques. Sans ces contournements, la connexion échouera.

## L'Incompatibilité Principale

*   **Policy-Based (Hérité/Legacy) :** Utilise une liste de contrôle d'accès (ACL) pour définir le trafic. Il négocie un tunnel IPsec séparé (SA) pour *chaque* ligne de cette ACL.
    *   *Exemple :* Si vous avez 5 sous-réseaux parlant à 5 sous-réseaux distants, il tente de construire 25 petits tunnels VPN séparés.
*   **Route-Based (Moderne) :** Utilise une interface virtuelle (VTI). Il négocie **un** tunnel IPsec géant (SA) pour tout (généralement `0.0.0.0/0` vers `0.0.0.0/0`).
    *   *Résultat :* Lorsqu'un pare-feu Route-Based essaie de dire "Je veux tout chiffrer", le pare-feu Policy-Based le rejette en disant "Je n'attendais que du trafic pour le sous-réseau A".

---

## Positions Officielles des Constructeurs & Confirmations

### 1. Palo Alto Networks (Natif Route-Based)
Les pare-feux Palo Alto sont basés sur le routage par conception. Leur documentation officielle indique explicitement que vous **devez** configurer des "Proxy IDs" pour imiter un pair basé sur des règles.

> **Confirmation officielle :** "Si le pair prend en charge les VPN basés sur des règles... vous devez configurer des Proxy IDs. Si vous ne configurez pas de Proxy IDs, le pare-feu utilise le Proxy ID par défaut (local : 0.0.0.0/0, distant : 0.0.0.0/0), qui pourrait ne pas correspondre à la configuration de la liste d'accès du pair."

*   **La solution :** Vous devez saisir manuellement chaque paire de sous-réseaux dans l'onglet "Proxy IDs" de la configuration du tunnel pour correspondre exactement aux ACL du côté distant.

### 2. Cisco IOS (Prend en charge les deux)
Cisco distingue officiellement "Crypto Map" (Policy-Based) et "VTI" (Route-Based).

> **Confirmation officielle :** La documentation Cisco sur "IPSec Virtual Tunnel Interface" note que les protocoles de routage dynamique (OSPF/BGP) sont pris en charge sur VTI (Route-Based) mais **pas** sur les Crypto Maps (Policy-Based).

*   **La solution :** Si vous connectez un VTI Cisco (Route-Based) à une Crypto Map standard (Policy-Based), vous devez généralement utiliser une route statique pointant vers l'interface et vous assurer que la Crypto Map de l'autre côté est configurée pour accepter le trafic. Le routage dynamique ne fonctionnera pas.

### 3. Fortinet (Prend en charge les deux)
Fortinet prend en charge les modes `Interface-based` (Route) et `Policy-based`, mais avertit de la perte de fonctionnalités lors de l'utilisation du mode Policy-based.

> **Confirmation officielle :** La documentation Fortinet liste explicitement les limitations des VPN Policy-Based, notant qu'ils ne peuvent pas utiliser les fonctionnalités de routage avancées.

*   **La solution :** Lorsqu'un FortiGate (Route-Based) se connecte à un pair Policy-Based, vous devez définir des **Sélecteurs de Phase 2** qui correspondent exactement à la politique distante, plutôt que d'utiliser le défaut `0.0.0.0/0`.

## Résumé des "Incompatibilités"

| Fonctionnalité | Route-Based | Policy-Based | Résultat |
| :--- | :--- | :--- | :--- |
| **Sélecteurs de Trafic** | 0.0.0.0/0 (Tout) | Sous-réseaux spécifiques | **Non-concordance** (Le tunnel ne s'établit pas) |
| **Nombre de SA** | 1 SA par Tunnel | 1 SA par paire de sous-réseaux | **Instabilité** (Certains sous-réseaux fonctionnent, d'autres échouent) |
| **Routage Dynamique** | Supporté (BGP/OSPF) | Non Supporté | **Échec de Routage** (Les routes ne se propageront pas) |

**Conclusion :** Ils sont incompatibles par défaut. Pour les faire fonctionner, vous devez forcer l'appareil Route-Based à "rétrograder" son comportement en définissant manuellement des **Proxy IDs / Sélecteurs de Trafic** pour correspondre aux attentes rigides de l'appareil Policy-Based.
