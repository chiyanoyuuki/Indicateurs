# Biais Journalier — S&P 500 / Nasdaq / Or / Dow Jones

Indicateur **TradingView (Pine Script v6)** qui affiche, dans un tableau flottant,
un biais **HAUSSIER / NEUTRE / BAISSIER** pour la séance en cours sur 4 actifs :

- **S&P 500**
- **Nasdaq**
- **Or**
- **Dow Jones**

Il fonctionne **quel que soit le graphique** sur lequel il est appliqué (les
données des 4 actifs sont récupérées en interne via `request.security`).

Aperçu du tableau affiché à l'écran :

```
┌───────────┬────────┬───────┬──────────┐
│ Biais D   │ Var %  │ Score │ Biais    │
├───────────┼────────┼───────┼──────────┤
│ S&P 500   │ +0.74 %│  +4   │ HAUSSIER │
│ Nasdaq    │ +1.10 %│  +6   │ HAUSSIER │
│ Or        │ -0.20 %│   0   │ NEUTRE   │
│ Dow Jones │ -0.55 %│  -3   │ BAISSIER │
└───────────┴────────┴───────┴──────────┘
```

---

## Installation

1. Ouvre [TradingView](https://www.tradingview.com/) → un graphique → **Pine Editor** (en bas).
2. Copie/colle le contenu de [`biais-journalier.pine`](biais-journalier.pine).
3. Clique sur **« Ajouter au graphique »** (Add to chart).
4. (Optionnel) **« Enregistrer »** puis, depuis la liste des indicateurs, tu peux
   l'ajouter à tes favoris pour le retrouver facilement.

> Pas besoin d'être sur un graphique précis : que tu sois sur l'EUR/USD ou sur
> le Bitcoin, le tableau affichera toujours le biais des 4 actifs configurés.

---

## Méthode de calcul

Le biais repose sur un **score de confluence** allant de **-6 à +6**, calculé sur
l'**unité de temps journalière (D)**. Chaque signal apporte **+1** (haussier),
**-1** (baissier) ou **0** (neutre) :

| # | Signal | +1 si | -1 si |
|---|--------|-------|-------|
| 1 | Tendance court terme | cours > EMA rapide (20) | cours < EMA rapide |
| 2 | Tendance moyen terme | cours > EMA lente (50) | cours < EMA lente |
| 3 | Référence veille | cours > clôture de la veille | cours < clôture de la veille |
| 4 | Sens du jour | cours > ouverture du jour | cours < ouverture du jour |
| 5 | Momentum (RSI 14) | RSI > 55 | RSI < 45 |
| 6 | Cassure du range veille | cours > plus haut de la veille | cours < plus bas de la veille |

**Classement final** (seuil par défaut = 2) :

```
score >= +2  ->  HAUSSIER
score <= -2  ->  BAISSIER
sinon        ->  NEUTRE
```

Tous ces paramètres (longueurs d'EMA, seuils RSI, seuil de biais, unité de temps)
sont **modifiables** dans les réglages de l'indicateur.

---

## Réglages

| Réglage | Défaut | Description |
|---------|--------|-------------|
| Source des symboles | `Futures` | `Futures` / `CFD` / `Indice (comptant)` / `Personnalisé` |
| Symboles personnalisés | futures | Utilisés uniquement en mode `Personnalisé` |
| Unité de temps du biais | `D` | Timeframe du calcul (ex. `W` pour hebdo) |
| EMA rapide / lente | 20 / 50 | Moyennes mobiles de tendance |
| Longueur RSI | 14 | Période du RSI |
| Seuils RSI haussier / baissier | 55 / 45 | Bornes de momentum |
| Seuil \|score\| | 2 | Force minimale pour valider un biais |
| Position du tableau | Haut droite | Emplacement à l'écran |
| Variation % / Score | activés | Colonnes optionnelles |

---

## Symboles

Le réglage **« Source des symboles »** bascule les 4 actifs d'un seul clic. Les
symboles utilisés selon la source choisie :

| Actif | **Futures** (défaut) | **CFD** | Indice (comptant) | ETF |
|-------|----------------------|---------|-------------------|-----|
| S&P 500 | `CME_MINI:ES1!` | `OANDA:SPX500USD` | `SP:SPX` | `AMEX:SPY` |
| Nasdaq | `CME_MINI:NQ1!` | `OANDA:NAS100USD` | `NASDAQ:NDX` | `NASDAQ:QQQ` |
| Or | `COMEX:GC1!` | `OANDA:XAUUSD` | `TVC:GOLD` | `AMEX:GLD` |
| Dow Jones | `CBOT_MINI:YM1!` | `OANDA:US30USD` | `DJ:DJI` | `AMEX:DIA` |

- **Futures** et **CFD** cotent quasiment 24 h/24 → biais plus réactif en
  pré-marché (recommandé pour le day-trading).
- Le **CFD** utilise par défaut le fournisseur **OANDA** (largement accessible).
  Si tu utilises un autre courtier (Capital.com, Pepperstone, FXCM…), passe en
  mode **`Personnalisé`** et saisis tes propres symboles (ex. `CAPITALCOM:US500`,
  `CAPITALCOM:US100`, `CAPITALCOM:GOLD`, `CAPITALCOM:US30`).
- Pour le **Nasdaq Composite** plutôt que le Nasdaq 100 : `NASDAQ:IXIC` (mode
  `Personnalisé`).

> En mode **`Personnalisé`**, ce sont les 4 champs « Symboles personnalisés » qui
> sont utilisés (ils valent les futures par défaut).

---

## Alertes

L'indicateur fournit des **conditions d'alerte** (menu *Alerte* → *Condition*) qui
se déclenchent quand un actif **passe** en biais HAUSSIER ou BAISSIER, par
exemple : « S&P 500 : passage en biais HAUSSIER ».

---

## Notes importantes

- **« Repaint » volontaire** : pour la séance en cours, le biais évolue au fil de
  la journée (il intègre le prix en temps réel) et **se fige à la clôture
  journalière**. C'est le comportement attendu pour un *biais du jour*.
- Le calcul utilise les sessions/données telles que fournies par TradingView pour
  chaque symbole. Les futures (cotation étendue) donnent un biais plus réactif que
  les indices au comptant.
- **Ceci n'est pas un conseil en investissement.** Outil d'aide à la décision, à
  utiliser à vos propres risques.
