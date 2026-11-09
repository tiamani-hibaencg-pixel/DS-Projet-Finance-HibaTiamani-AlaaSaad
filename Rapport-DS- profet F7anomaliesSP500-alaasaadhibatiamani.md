## Détection d'anomalies sur le S&P 500 (Projet F7)

Auteurs : Alaa Saad et Hiba Tiamani · Cours : Data sciences · Date : 10 novembre 2026

## Résumé

Ce projet cherche à repérer, par apprentissage non supervisé, les journées atypiques du marché américain des actions entre novembre 2016 et novembre 2026. Deux modèles (Isolation Forest et LOF) sont appliqués à deux variables : le rendement quotidien et la volatilité sur 20 jours. Avec une proportion d'anomalies fixée à 2 %, chaque modèle signale 50 journées sur 2 493. Les anomalies se concentrent surtout sur deux épisodes : le krach de mars-avril 2020 (Covid-19) et les turbulences d'avril 2025. Les modèles repèrent donc les régimes de

volatilité extrême, mais pas les épisodes de stress plus modérés ni les baisses graduelles.

- 1. Question de recherche

Quelles journées de marché sont statistiquement atypiques, et correspondent-elles à des événements financiers connus ? Il s'agit d'un problème non supervisé : aucune étiquette n'indique quelles journées sont des anomalies. Il n'existe donc ni découpage entraînement/test, ni mesure d'exactitude. L'évaluation est qualitative : on compare les anomalies détectées à des événements

documentés.

- 2. Données

- Source : FRED (Federal Reserve Bank of St. Louis), série SP500 , niveau de clôture quotidien de l'indice S&P 500.

- Période : du 7 novembre 2016 au 5 novembre 2026.

- Nettoyage : le fichier brut compte 2 609 lignes et deux colonnes (date et valeur de clôture). Il débute le 7 novembre 2016 à 2 131,52 points. Les 96 jours sans cotation (jours fériés, codés par un point « . ») sont supprimés, ce qui laisse 2 513 observations.

- Observations analysées : 2 493. Le calcul des rendements et de la volatilité glissante fait perdre les 20 premières observations.


## 2.1 Niveau de l'indice

| Statistique | Valeur (points) |
| --- | --- |
| Observations | 2 513 |
| Moyenne | 4 213,98 |
| Écart-type | 1 515,38 |
| Minimum | 2 131,52 |
| 1er quartile (25 %) | 2 867,19 |
| Médiane (50 %) | 3 999,09 |
| 3e quartile (75 %) | 5 209,91 |
| Maximum | 7 818,93 |

Interprétation. Le minimum correspond à la première observation (2 131,52) et le maximum atteint 7 818,93 : sur dix ans, l'indice est multiplié par environ 3,7. L'écart-type élevé (1 515 points) et l'écart entre la moyenne et la médiane traduisent cette tendance de fond, pas un risque. C'est pourquoi on n'analyse pas le niveau de l'indice, mais des variables qui ne dépendent pas de la tendance : le rendement et la volatilité.

*Figure 1. Niveau de clôture du S&P 500.*

Interprétation. La tendance est nettement haussière, mais elle est interrompue par plusieurs replis. Les trois plus visibles sont la chute brutale du début 2020 (suivie d'un rebond rapide), la baisse prolongée de 2022 et le repli net d'avril 2025, lui aussi suivi d'un rebond. On distingue aussi un repli plus modeste fin 2018 et, d'après le graphique, un autre début 2026. Ces replis n'ont pas la même nature : brutaux en 2020 et en 2025, graduels en 2022. Cette différence compte pour la suite, car un modèle d'anomalies réagit surtout aux mouvements brusques.

## 2.2 Variables construites

On construit deux variables plus stables que le niveau :

- le rendement logarithmique quotidien, rt = ln(Pt) − ln(Pt−1) ;

- la volatilité glissante sur 20 jours, écart-type des rendements des 20 dernières séances (environ un mois).


| Variable |   |   | Moyenne Médiane Écart-type Minimum Maximum |   |   |
| --- | --- | --- | --- | --- | --- |
| Rendement quotidien 0,048 % |   | 0,080 % | 1,15 % | −12,77 % | +9,09 % |
| Volatilité sur 20 jours | 0,94 % | 0,77 % | 0,66 % | 0,21 % | 6,17 % |

Interprétation. Le rendement moyen est proche de zéro, alors que l'écart-type vaut 1,15 % : la moitié des jours se situe entre −0,38 % et +0,59 % (quartiles). Les extrêmes sont en revanche très éloignés : −12,77 % et +9,09 % représentent environ −11 et +8 écarts-types. De tels mouvements seraient quasi impossibles si les rendements suivaient une loi normale. Les rendements ont donc des queues épaisses, ce qui justifie le recours à la détection d'anomalies. Pour la volatilité, la moyenne (0,94 %) dépasse la médiane (0,77 %) : quelques périodes de très forte volatilité tirent la moyenne vers le haut. Le maximum (6,17 %) vaut environ 8 fois la

médiane.

*Figure 2. Rendement quotidien et volatilité sur 20 jours.*

Interprétation. Les deux séries montrent des « grappes » de volatilité : des périodes calmes alternent avec des périodes agitées. L'amplitude des rendements est maximale en mars 2020 (de −12,8 % à environ +9 %). On voit aussi des rafales marquées fin 2018, en 2022 et surtout en avril 2025 (pics d'environ +9 % et −6 %). La volatilité sur 20 jours reste d'environ 0,4 à 0,6 % en 2017, monte à environ 1,7 % début 2018 (Volmageddon), puis à 2 % fin 2018 et en 2019, et oscille entre 1 % et 2,2 % en 2022. Elle atteint 6,17 % au printemps 2020, soit une dizaine de fois son niveau de 2017, puis environ 3,2 % au printemps 2025. Le pic de 2020 domine donc nettement tous les autres : c'est lui qui servira de référence « extrême » aux modèles.

## 3. Méthode

## 3.1 Standardisation

Les deux variables n'ont pas la même échelle. Elles sont standardisées (moyenne 0, écart-type 1) pour qu'aucune ne domine, ce qui compte surtout pour LOF, fondé sur des distances.


## 3.2 Modèles

- Isolation Forest : des arbres découpent les données au hasard ; un point atypique est isolé en peu de découpages et reçoit un score d'anomalie élevé (300 arbres, graine aléatoire fixée à 42).

- LOF (Local Outlier Factor) : compare la densité locale autour d'un point à celle de ses voisins ; un point situé dans une zone bien moins dense que son voisinage est atypique (35 voisins).

## 3.3 Proportion d'anomalies

Le paramètre contamination fixe la part des journées classées comme anomalies. C'est un choix de l'analyste, pas un résultat. La valeur de référence est 2 %, soit 50 journées sur 2 493. Une analyse de sensibilité teste ensuite 1 %, 3 % et 5 %. Comme ce paramètre fixe le nombre d'anomalies, les deux modèles en signalent toujours autant : la comparaison porte donc sur lesquelles ils choisissent.

## 3.4 Évaluation par comparaison avec des événements

Sept fenêtres d'événements ont été fixées avant l'exécution des modèles. L'indicateur retenu est la part des anomalies qui tombent dans ces fenêtres, comparée à la part des jours que les fenêtres représentent.

| Événement | Fenêtre |
| --- | --- |
| Volmageddon | 1er – 14 février 2018 |
| Chute de fin 2018 | 1er octobre – 31 décembre 2018 |
| Krach lié au Covid-19 | 20 février – 30 avril 2020 |
| Inflation et hausse des taux | 1er janvier – 31 octobre 2022 |
| Faillite de SVB | 8 – 20 mars 2023 |
| Turbulences d'août 2024 | 1er – 12 août 2024 |
| Annonces tarifaires | 1er – 15 avril 2025 |

Ensemble, ces fenêtres couvrent 360 jours, soit 14 % des 2 493 jours analysés. Si les anomalies étaient réparties au hasard, environ 14 % d'entre elles tomberaient dans ces fenêtres.

## 4. Résultats

## 4.1 Accord entre les deux modèles

Chaque modèle signale 50 journées. Les deux s'accordent sur 36 d'entre elles ; au moins un des deux en signale 64. L'indice de Jaccard est de 0,56 : les modèles se rejoignent sur un peu plus de la moitié des journées signalées, mais pas sur toutes. Leurs critères diffèrent (isolement global pour l'un, densité locale pour l'autre), et cette divergence est attendue.


## 4.2 Localisation des anomalies

*Figure 3. Anomalies détectées par Isolation Forest (haut) et LOF (bas).*

Interprétation. Les deux modèles signalent massivement mars-avril 2020 : les points rouges forment une colonne serrée, car l'indice a chuté très vite. Ils signalent aussi l'épisode d'avril 2025, où les points s'étalent sur quelques semaines. Isolation Forest ajoute plusieurs points en 2022 et un point fin 2018 ; LOF place en plus, d'après la frise, quelques points dans la période calme de 2017. Un point isolé apparaît aussi vers mi-2020, chez les deux modèles. D'après la frise, plusieurs anomalies situées juste après les fenêtres d'événements (mai-juin 2020, fin avril-mai 2025) appartiennent vraisemblablement aux mêmes épisodes : la volatilité reste élevée après le choc, et les fenêtres retenues s'arrêtent un peu tôt. Aucune anomalie n'apparaît en 2026, y compris pour le repli visible sur la Figure 1.


## 4.3 Les 15 journées les plus atypiques (Isolation Forest)

| Date |   |   | S&P 500 Rendement (%) Volatilité 20 j (%) Score |
| --- | --- | --- | --- |
| 2020-03-12 2 480,64 | −9,99 | 3,75 | 0,790 |
| 2020-03-13 2 711,02 | 8,88 | 4,41 | 0,792 |
| 2020-03-16 2 386,13 | −12,77 | 5,11 | 0,813 |
| 2020-03-17 2 529,19 | 5,82 | 5,38 | 0,804 |
| 2020-03-18 2 398,10 | −5,32 | 5,42 | 0,806 |
| 2020-03-20 2 304,92 | −4,43 | 5,47 | 0,801 |
| 2020-03-23 2 237,40 | −2,97 | 5,47 | 0,774 |
| 2020-03-24 2 447,33 | 8,97 | 5,96 | 0,828 |
| 2020-03-26 2 630,07 | 6,05 | 6,13 | 0,822 |
| 2020-03-27 2 541,47 | −3,43 | 6,17 | 0,791 |
| 2020-03-30 2 626,65 | 3,30 | 6,12 | 0,796 |
| 2020-04-01 2 470,50 | −4,51 | 6,04 | 0,806 |
| 2020-04-02 2 526,90 | 2,26 | 6,07 | 0,778 |
| 2020-04-06 2 663,68 | 6,80 | 6,06 | 0,822 |
| 2020-04-08 2 749,98 | 3,35 | 5,90 | 0,795 |

## Interprétation.

- Les 15 journées tombent toutes entre le 12 mars et le 8 avril 2020, soit quatre semaines.

- Le modèle est symétrique : il signale 8 hausses et 7 baisses. Il détecte les mouvements extrêmes dans les deux sens, pas seulement les chutes.

- Le score le plus élevé (0,828) est celui du 24 mars 2020 (+8,97 % avec une volatilité de 5,96 %), et non celui de la plus forte baisse du 16 mars (−12,77 %, score de 0,813). Le score dépend donc de la combinaison du rendement et de la volatilité : le 16 mars, la volatilité sur 20 jours (5,11 %) n'avait pas encore atteint son niveau maximal.

- Plusieurs journées ont un rendement modéré (2,26 % ou −2,97 %) mais une volatilité très élevée (supérieure à 5 %). Le modèle détecte donc un régime de forte turbulence, pas seulement des chocs d'une journée.


## 4.4 Comparaison avec les événements

| Événement | Jours Anomalies Isolation Forest Anomalies LOF |   |   |
| --- | --- | --- | --- |
| Volmageddon (février 2018) | 10 | 0 | 0 |
| Chute de fin 2018 | 63 | 1 | 1 |
| Krach lié au Covid-19 | 50 | 36 | 30 |
| Inflation et hausse des taux (2022) | 209 | 3 | 0 |
| Faillite de SVB (mars 2023) | 9 | 0 | 0 |
| Turbulences d'août 2024 | 8 | 0 | 0 |
| Annonces tarifaires (avril 2025) | 11 | 4 | 5 |

| Indicateur | Résultat |
| --- | --- |
| Part des jours situés dans les fenêtres | 14 % |
| Part des anomalies Isolation Forest dans les fenêtres | 88 % |
| Part des anomalies LOF dans les fenêtres | 72 % |

## Interprétation.

- Les anomalies se trouvent environ 6 fois (Isolation Forest) et 5 fois (LOF) plus souvent dans les fenêtres que ne le voudrait le hasard (14 %).

- Le Covid-19 domine : Isolation Forest signale 36 des 50 jours de la fenêtre (72 %) et LOF en signale 30 (60 %). Pour avril 2025, ils signalent respectivement 4 et 5 des 11 jours (36 % et 45 %).

- Les autres fenêtres reçoivent au plus 3 anomalies. Volmageddon, SVB et août 2024 n'en reçoivent aucune.

- Robustesse sans le Covid-19 : pour Isolation Forest, 8 des 14 anomalies hors Covid tombent dans les autres fenêtres (57 %, contre 12,7 % des jours), soit environ 4,5 fois le hasard. Pour LOF, 6 des 20 anomalies hors Covid y tombent (30 %), soit environ 2,4 fois le hasard. Ces chiffres, calculés à partir du tableau ci-dessus, reposent sur de petits effectifs et doivent être lus avec prudence. Ils montrent néanmoins qu'Isolation Forest s'aligne mieux sur les événements que LOF.

## 4.5 Sensibilité à la proportion d'anomalies

| Contamination | Anomalies (chaque | Isolation Forest : part dans | LOF : part dans les |
| --- | --- | --- | --- |
|   | modèle) | les fenêtres | fenêtres |
| 1 % | 25 | 100 % | 92 % |
| 2 % | 50 | 88 % | 72 % |
| 3 % | 75 | 76 % | 65 % |
| 5 % | 125 | 70 % | 51 % |

Interprétation. Plus on accepte d'anomalies, plus la part située dans les fenêtres diminue (de 100 % à 70 % pour Isolation Forest, de 92 % à 51 % pour LOF). Les journées les plus extrêmes sont donc bien liées à des événements connus : à 1 %, la totalité des anomalies d'Isolation


Forest y tombent. Les journées ajoutées ensuite sont moins extrêmes et tombent plus souvent hors des fenêtres. Même à 5 %, les deux parts (70 % et 51 %) restent au-dessus des 14 % attendus au hasard, ce qui renforce le résultat principal. LOF perd en revanche plus vite sa cohérence avec les événements.

## 5. Discussion

Ce que les modèles détectent. Les anomalies correspondent surtout à des régimes de volatilité extrême. Le krach de mars-avril 2020 en regroupe l'essentiel, et les annonces tarifaires d'avril 2025 constituent le second épisode net. Les journées les plus atypiques sont

donc bien liées à des événements connus. Ce qu'ils ne détectent pas. Volmageddon, la faillite de SVB et les turbulences d'août 2024 ne reçoivent aucune anomalie ; la chute de fin 2018 en reçoit une seule par modèle, et la période 2022 trois pour Isolation Forest et aucune pour LOF. Ces épisodes sont des chocs brefs ou des baisses graduelles : à l'échelle de dix ans, ils restent moins extrêmes que le Covid-19. Une anomalie est ici un écart relatif à l'échantillon ; sur une période dominée par un épisode exceptionnel, un stress modéré paraît presque normal. Une proportion de 2 % ne retient que les

journées les plus extrêmes. Isolation Forest et LOF. Isolation Forest s'aligne davantage sur les événements (88 % contre 72 % des anomalies dans les fenêtres ; l'écart se creuse sans le Covid-19). LOF, fondé sur la densité locale, signale aussi des journées dans des périodes calmes. Ces points peuvent correspondre à des événements absents de notre liste ou à de simples écarts locaux sans

portée économique : seule une recherche date par date permettrait de trancher. Un effet de la construction. La volatilité est calculée sur 20 jours : un seul choc gonfle cette variable pendant environ un mois. Les anomalies apparaissent donc en grappes, et les 36

anomalies du Covid-19 ne correspondent pas à 36 événements indépendants. Des fenêtres un peu courtes. D'après la frise, certaines anomalies placées juste après les

fenêtres appartiennent vraisemblablement aux mêmes épisodes. La part d'anomalies liées aux événements est donc plutôt sous-estimée que surestimée.

## 6. Limites

- Évaluation subjective. Sans étiquettes, il n'existe pas de bonne réponse pour mesurer la performance. La comparaison avec des événements est qualitative, et la largeur des fenêtres (63 jours pour fin 2018, 209 pour 2022, 9 pour SVB) influence les parts calculées.

- Paramètres arbitraires. La proportion d'anomalies, le nombre de voisins de LOF et la liste des événements sont des choix de l'analyste.

- Deux variables seulement. Les volumes, le VIX ou les écarts de taux pourraient modifier les résultats.

- Période courte et dominée par un épisode. Dix ans de données et un épisode exceptionnel (Covid-19) limitent la portée des conclusions.

- Analyse rétrospective. La standardisation et l'entraînement utilisent toute la période, y compris le futur par rapport à chaque date. Ce montage convient à une étude historique, pas à un système de détection en temps réel.

- Anomalie n'est pas erreur. Une journée atypique reflète ici un vrai mouvement de marché, pas un problème de données.


- 7. Conclusion

Sur dix ans de données du S&P 500, Isolation Forest et LOF identifient 50 journées atypiques chacun (2 % de l'échantillon), avec un accord partiel (indice de Jaccard de 0,56). Ces journées se concentrent dans des périodes de turbulence connues : 88 % des anomalies d'Isolation Forest et 72 % de celles de LOF tombent dans des fenêtres qui ne couvrent que 14 % des jours. Cet effet tient en grande partie au Covid-19 et aux annonces tarifaires d'avril 2025 ; les épisodes plus modérés (Volmageddon, SVB, août 2024) et les baisses graduelles (fin 2018, 2022) passent largement inaperçus. Les résultats résistent à un changement de la proportion d'anomalies, mais l'évaluation reste qualitative. Des pistes d'amélioration seraient d'ajouter des variables (volume, VIX), de tester d'autres fenêtres de volatilité et d'utiliser une liste

d'événements définie de façon plus systématique.

## Références

- S&P Dow Jones Indices LLC, S&P 500 [SP500], extrait de FRED, Federal Reserve Bank of St. Louis : https://fred.stlouisfed.org/series/SP500

- Liu, F. T., Ting, K. M., & Zhou, Z.-H. (2008). Isolation Forest. IEEE International Conference on Data Mining (ICDM).

- Breunig, M. M., Kriegel, H.-P., Ng, R. T., & Sander, J. (2000). LOF: Identifying Density-Based Local Outliers. ACM SIGMOD.

- Documentation de scikit-learn : IsolationForest , LocalOutlierFactor .
