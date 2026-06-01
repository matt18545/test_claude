# CentrO-Flux · Intelligence Routière 28

Application web de **macro-simulation et de modélisation géographique du trafic routier**
pour le département de l'**Eure-et-Loir (28)**. Elle simule et visualise dynamiquement la
charge du réseau routier en fonction de variables démographiques, temporelles et
comportementales.

➡️ **Ouvrir l'application : [`centro-flux-28.html`](./centro-flux-28.html)** (un simple
double-clic dans un navigateur moderne suffit ; une connexion Internet est requise pour
charger les CDN Leaflet / Chart.js / FontAwesome et le fond de carte).

---

## Fonctionnalités

- **Carte interactive (Leaflet)** centrée sur l'Eure-et-Loir : graphe routier de 24 nœuds
  (17 communes + 7 verrous de transit) et 34 tronçons / 68 arcs orientés reliant
  **Chartres, Dreux, Châteaudun, Nogent-le-Rotrou** et les accès vers l'Île-de-France,
  Le Mans, Alençon, Orléans et Évreux (A11, N10, N12, N154, D928, RD910…).
- **Coloration dynamique** des arcs selon le taux de charge `V/C` :
  vert (fluide) → orange (dense) → rouge (congestion) → pourpre (asphyxie), épaisseur
  proportionnelle au volume de véhicules.
- **Tableau de bord analytique (Chart.js)** : profil 24 h (vitesse + congestion), kilomètres
  congestionnés par catégorie de voie, répartition des véh·km par type de route, et KPIs en
  direct (vitesse moyenne, temps perdu, débit global, émissions de CO₂, réseau saturé).
- **Panneau de configuration** : heure de la journée, typologie de conduite
  (Éco / Standard / Agressif), indice d'agressivité, respect des limitations, taux de
  télétravail, intensité de la demande, et lecture **temps réel / pas à pas**.
- **Flux d'événements** signalant les saturations en cours et **imminentes** (projection h+1).
- **Panneau Méthodologie** documentant les modèles et la structure du graphe.

## Modèle mathématique

| Étape | Modèle |
|-------|--------|
| Génération / distribution | Modèle **gravitaire** (Pareto spatial), déterrence `(d+d₀)^−γ` |
| Profil temporel | Double pic gaussien (08 h / 17 h 30) + socle de transit |
| Logique d'activités | Navettes domicile→travail (matin) / travail→domicile (soir) — pointes **directionnelles** |
| Coût des liens | **Bureau of Public Roads** : `T = T₀·(1 + α·(V/C)^β)`, α = 0,15 · β = 4 |
| Vitesse–densité | **Greenshields** : `v = v_libre·(1 − k/k_jam)`, capacité `q_max = v_libre·k_jam/4` |
| Affectation | **Équilibre usager approché** par affectation incrémentale (Dijkstra à tas binaire, 6 tranches) → report de trafic sur le réseau secondaire quand un axe sature |
| Émissions | Facteur en U `e(v)=A/v+B+C·v²` majoré en régime arrêt-relance saturé |

Population de référence : **≈ 430 000 habitants**. Les données géographiques sont procédurales
mais calibrées sur la hiérarchie réelle des voies du département (valeurs indicatives, à
vocation pédagogique).

## Architecture du code

Fichier unique `centro-flux-28.html` (HTML + CSS + JS), structuré en modules clairs :
**Store centralisé** de l'état · **données géographiques** · **moteur de calcul isolé**
(BPR, Greenshields, gravité) · **affectation** (Dijkstra / incrémental) · **rendu**
(carte Leaflet, graphiques Chart.js, KPIs, toasts) — les calculs sont entièrement découplés
de l'affichage.

## Tests

La logique pure (graphe, gravité, BPR, Greenshields, Dijkstra, affectation, KPIs, profil 24 h)
a été validée par une suite d'assertions exécutée sous Node (26 vérifications) : bornes
exactes de la BPR, chute de vitesse Greenshields en saturation, directionnalité de la pointe
matinale, sensibilité à la demande, connexité du graphe, etc.
