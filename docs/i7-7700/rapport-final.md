# Rapport final — expérimentations LLM en local sur machine CPU-only ancienne

## 1. Objet du document

Ce rapport synthétise une série d’expérimentations menées sur une machine **CPU-only** afin d’identifier quels modèles GGUF restent réellement intéressants en usage local, malgré une plateforme modeste.

L’objectif n’est pas de produire un classement “absolu”, mais de répondre à une question pratique :

> **quels modèles valent le coup sur un vieux PC sans GPU, avec 32 Go de RAM, et pour quels usages précis ?**

Le document regroupe :

- la logique de test retenue ;
- les résultats de performance utiles ;
- les résultats de capacité sur 10 prompts ;
- les enseignements les plus robustes ;
- des recommandations concrètes pour rejouer ou prolonger les tests.

---

## 2. Contexte matériel et logiciel

Configuration de test :

- **CPU** : Intel **i7-7700** (4C/8T)
- **RAM** : **32 Go**
- **GPU** : aucun
- **OS** : **Debian 13 headless**
- **Backend** : **llama.cpp / llama-bench / llama-server**
- **Compilation CPU** : AVX2 / FMA / F16C / OpenMP / build native

Conséquence directe : sur cette machine, la limite principale n’est pas seulement la “taille du modèle”, mais surtout le couple :

- **bande passante mémoire** ;
- **coût du cache KV quand le contexte grandit**.

Autrement dit, un modèle peut sembler très bon “à vide”, puis devenir beaucoup moins intéressant dès que le contexte s’allonge.

---

## 3. Méthode de travail retenue

### 3.1 Deux familles de tests

Les expérimentations ont séparé deux dimensions :

1. **performance machine**  
   Mesurée avec `llama-bench`, en regardant surtout :
   - `pp1024` : vitesse de préfill ;
   - `tg128` : vitesse de génération ;
   - la dégradation à `d1024`, `d2048`, `d4096`.

2. **capacité utile réelle**  
   Évaluée via un mini benchmark de **10 prompts**, notés **/25** chacun, couvrant :
   - raisonnement ;
   - code ;
   - compréhension/synthèse ;
   - usage réel/méthodologie.

Cette séparation est essentielle : un modèle très rapide n’est pas forcément bon, et un très bon modèle n’est pas forcément rentable sur cette machine.

### 3.2 Réglages de screening retenus

Le protocole a convergé vers une base simple et cohérente pour les comparaisons :

```bash
-t 8 -b 1024 --ubatch-size 1024 --flash-attn 1
```

Avec, dans la majorité des benchs comparatifs, un KV cache en `q8_0` côté K/V quand cela fonctionne correctement.

### 3.3 Ce que les premiers tests ont appris

Les premiers essais ont montré plusieurs choses utiles :

- **8 threads** est un bon point d’équilibre global sur cette machine.
- **`type_v` a plus d’impact que `type_k`**.
- Les réglages intermédiaires “un peu au hasard” coûtent vite du temps pour peu de gain.
- Le vrai discriminant n’est pas seulement la vitesse brute, mais la **tenue avec la profondeur de contexte**.

---

## 4. Résultat principal en une phrase

Le constat le plus fort de cette campagne est le suivant :

> **sur une vieille machine CPU-only, les meilleurs compromis ne sont pas forcément les modèles denses intermédiaires ; les MoE à faible nombre de paramètres actifs et certains petits modèles denses bien entraînés sont souvent bien plus intéressants.**

C’est probablement l’enseignement le plus utile du rapport.

---

## 5. Enseignements transverses les plus solides

### 5.1 Les MoE “léger en actif” sont la vraie bonne surprise

Les modèles de type **A1B** ou **A3B** montrent qu’un gros total de paramètres n’est pas forcément synonyme d’inutilisabilité sur CPU. Ce qui compte vraiment ici, c’est le **nombre de paramètres actifs par token**.

Quand ce nombre reste faible :

- le préfill peut rester très élevé ;
- la génération reste fluide ;
- la machine encaisse mieux le modèle que prévu ;
- le compromis qualité/vitesse devient très intéressant.

### 5.2 Les denses 7B–14B ne sont pas automatiquement un bon milieu de gamme

Intuitivement, on pourrait s’attendre à ce qu’un 8B ou un 14B dense soit un “bon compromis naturel”. Sur cette machine, ce n’est pas toujours vrai.

Dans plusieurs cas, ces modèles :

- sont **nettement plus lents** que les meilleurs petits modèles ;
- restent **moins convaincants qualitativement** que les meilleurs gros MoE ;
- et n’occupent donc pas toujours une vraie place rentable.

Autrement dit : **le milieu de gamme dense peut être le moins intéressant**.

### 5.3 Les petits modèles denses gardent une vraie valeur

Les modèles autour de **1B à 3B** gardent un intérêt fort quand on cherche :

- de la réactivité ;
- des tâches courtes ;
- du filtrage, de l’assistance simple, du shell ou du pré-tri ;
- un usage très léger en RAM.

Ils ne remplacent pas les meilleurs généralistes, mais ils peuvent être les plus agréables à l’usage sur des tâches simples.

### 5.4 Le long contexte change réellement la hiérarchie

Un modèle excellent en `pp1024` ou `tg128` à vide peut perdre beaucoup d’intérêt dès qu’on pousse la profondeur.

C’est un point décisif du rapport :

- certains modèles chutent fortement ;
- d’autres restent étonnamment stables ;
- et cette stabilité compte souvent plus que le score brut initial.

---

## 6. Tableau comparatif de lecture rapide

> **Attention** : les vitesses sont celles de cette machine et de ce protocole. Elles servent à comparer entre elles les options testées, pas à généraliser à n’importe quelle plateforme.

| Modèle | Type | Taille approx. | pp1024 | tg128 | pp1024 @ d4096 | tg128 @ d4096 | Score capacité moyen | Lecture rapide |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| **BailingMoe2 16B.A1B Q4_K_M** | MoE | 9.22 GiB | 128.28 | 35.10 | 82.31 | 18.02 | 16.6/25 | Monstre de vitesse CPU, plus spécialisé que polyvalent |
| **LFM2-8B-A1B-Q4_K_M** | MoE | 4.70 GiB | 88.36 | 28.90 | 77.30 | 22.68 | 18.4/25 | Excellent compromis vitesse + tenue contexte |
| **Qwen2 3B Q4_K** | Dense | ~1.8 GiB | 47.36 | 14.67 | 21.11 | 10.51 | 11.5/25 | Rapide et léger, mais qualité inégale |
| **SmolLM3 3B Q4_K_M** | Dense | 1.78 GiB | 46.48 | 14.85 | 20.52 | 10.30 | 15.7/25 | Petit dense réactif, utile pour tâches simples |
| **SmolLM2 1.7B** (`llama-1.7B-Instruct-Q4_K_M`) | Dense | 1005 MiB | 72.92 | 26.46 | 19.50 | 9.16 | 5.4/25 | Ultra rapide, mais trop faible qualitativement |
| **Qwen3-8B-Q4_K_M** | Dense | 4.68 GiB | 19.79 | 6.22 | 9.05 | 3.54 | 19.6/25 | Qualité correcte mais rendement machine faible |
| **Qwen3-Coder-30B-A3B Q4_K_M** | MoE | 17.28 GiB | 31.55 | 14.32 | 9.47 | 6.36 | 23.4/25 | Très bon qualitativement, mais lourd en contexte long |
| **Gemma-4-26B-A4B** | MoE | 15.70 GiB | 31.65 | 7.18 | 26.56 | 5.59 | 24.5/25 | Le plus solide en qualité générale |
| **Qwen3.6-35B-A3B** | MoE | 19.91 GiB | 23.74 | 8.51 | 24.97 | 9.27 | 24.2/25 | Très fort et étonnamment stable en profondeur |

---

## 7. Lecture des résultats par grande famille

### 7.1 Les meilleurs profils “performance pure CPU”

#### BailingMoe2 16B.A1B Q4_K_M

C’est le résultat le plus spectaculaire de la campagne côté débit brut.

Points marquants :

- **128.28 t/s** en préfill ;
- **35.10 t/s** en génération ;
- reste encore à **82.31 / 18.02 t/s** à `d4096`.

Ce modèle montre très bien ce qu’un **MoE à faible actif** peut produire sur CPU. Il donne une sensation de vitesse presque “hors catégorie” pour la machine.

Sa limite n’est pas la vitesse, mais plutôt son profil qualitatif :

- excellent en code concret et message naturel ;
- beaucoup moins convaincant en raisonnement, analyse technique profonde et méthodologie.

**Conclusion pratique** : excellent moteur de terrain, pas le meilleur cerveau généraliste.

#### LFM2-8B-A1B-Q4_K_M

C’est probablement le **meilleur compromis global performance/stabilité** de la campagne.

Points marquants :

- **88.36 t/s** en préfill ;
- **28.90 t/s** en génération ;
- surtout une tenue remarquable à `d4096` : **77.30 / 22.68 t/s**.

Là où beaucoup de modèles s’écroulent avec la profondeur, LFM2 reste très haut. C’est un très bon profil pour du travail réel avec contexte déjà chargé.

Qualitativement, il est :

- excellent en logique ;
- très bon en débogage ;
- très bon en synthèse ;
- nettement plus faible en génération de code “from scratch” et en méthodologie structurée.

**Conclusion pratique** : un des meilleurs choix si l’on cherche un assistant vif, robuste et très rentable sur CPU-only.

### 7.2 Les meilleurs profils “qualité générale”

#### Gemma-4-26B-A4B

C’est le modèle qui ressort le plus nettement comme **généraliste haut niveau**.

Score moyen de capacité : **24.5/25**.

Il combine :

- excellent raisonnement ;
- excellent code ;
- excellente compréhension ;
- très bon comportement en usage réel.

Techniquement, il n’est pas le plus rapide en génération, mais il reste exploitable :

- **31.65 t/s** en préfill ;
- **7.18 t/s** en génération ;
- décroissance modérée avec le contexte.

**Conclusion pratique** : si l’objectif est d’avoir le modèle le plus fiable et le plus polyvalent du lot, Gemma est une référence forte.

#### Qwen3.6-35B-A3B

Très proche de Gemma sur la qualité, avec un profil peut-être un peu plus orienté technique/code.

Score moyen de capacité : **24.2/25**.

Il se distingue par une **stabilité contextuelle étonnamment bonne** pour sa taille :

- `tg128` reste autour de **9 t/s** jusqu’à `d4096` ;
- `pp1024 @ d4096` reste proche de **25 t/s**.

C’est un résultat important : ce modèle coûte en RAM, mais il paie mieux ce coût que beaucoup de concurrents.

**Conclusion pratique** : très bon choix pour un assistant technique “premium” sur CPU, à condition d’accepter son poids mémoire.

#### Qwen3-Coder-30B-A3B Q4_K_M

Le modèle est très fort qualitativement (**23.4/25**), notamment sur :

- raisonnement ;
- correction de code ;
- shell ;
- méthodologie.

En revanche, sur cette machine, son coût contexte est plus sévère :

- **31.55 / 14.32 t/s** à vide ;
- mais seulement **9.47 / 6.36 t/s** à `d4096`.

**Conclusion pratique** : excellent pour la qualité pure, mais moins séduisant que Qwen3.6-35B si l’on veut de la tenue en profondeur.

### 7.3 Les petits modèles denses qui restent intéressants

#### SmolLM3 3B Q4_K_M

SmolLM3 montre qu’un petit dense bien choisi garde une vraie utilité.

Forces :

- **46.48 / 14.85 t/s** ;
- faible empreinte mémoire ;
- bon comportement pour des tâches simples à modérées.

Faiblesses :

- qualité inégale ;
- faible structuration méthodologique ;
- plafond vite atteint sur les tâches complexes.

**Conclusion pratique** : très bon candidat “léger et nerveux”, surtout pour l’assistance simple.

#### Qwen2 3B Q4_K

Il reste performant en vitesse pour sa taille, mais la qualité moyenne observée est plus faible que ce qu’on espère d’un vrai modèle principal.

Il garde néanmoins de l’intérêt pour :

- shell ;
- petites tâches techniques ;
- usage léger et rapide ;
- environnements où chaque GiB compte.

**Conclusion pratique** : modèle d’appoint plus que modèle central.

#### SmolLM2 1.7B (`llama-1.7B-Instruct-Q4_K_M`)

Point important pour la lecture des résultats :

> le **SmolLM2 1.7B** correspond ici au modèle benchmarké sous le nom **`llama-1.7B-Instruct-Q4_K_M`**.

Il a bien été testé **en technique et en capacité**.

Le modèle est très impressionnant en vitesse brute pour sa taille :

- **72.92 t/s** en préfill ;
- **26.46 t/s** en génération.

Mais la qualité moyenne mesurée (**5.4/25**) reste trop basse pour en faire autre chose qu’un démonstrateur de réactivité ou un outil ultra-basique.

**Conclusion pratique** : intéressant pour mesurer ce qu’un très petit modèle peut offrir en latence, pas comme assistant principal.

### 7.4 Le cas des denses intermédiaires

#### Qwen3-8B-Q4_K_M

Ce modèle est utile dans le rapport car il illustre très bien un phénomène important : un modèle **correct qualitativement** peut malgré tout être **peu rentable sur la machine**.

Son score capacité est loin d’être mauvais (**19.6/25**), mais ses performances techniques sont faibles relativement à son gabarit :

- **19.79 / 6.22 t/s** à vide ;
- **9.05 / 3.54 t/s** à `d4096`.

Il ne prend donc pas une position très convaincante entre les petits denses rapides et les MoE beaucoup plus intéressants.

**Conclusion pratique** : bon exemple d’un milieu de gamme dense qui ne justifie pas toujours sa place sur ce type de PC.

---

## 8. Classement par usage

### 8.1 Meilleurs choix si la priorité est la vitesse

1. **BailingMoe2 16B.A1B**
2. **LFM2-8B-A1B**
3. **SmolLM2 1.7B**
4. **SmolLM3 3B**

Lecture : ici on cherche le débit et la sensation de réactivité, pas la qualité maximale.

### 8.2 Meilleurs choix si la priorité est la qualité générale

1. **Gemma-4-26B-A4B**
2. **Qwen3.6-35B-A3B**
3. **Qwen3-Coder-30B-A3B**
4. **LFM2-8B-A1B**

Lecture : ici on privilégie la fiabilité intellectuelle globale.

### 8.3 Meilleurs choix si la priorité est le compromis global sur ce PC

1. **LFM2-8B-A1B-Q4_K_M**
2. **Qwen3.6-35B-A3B**
3. **Gemma-4-26B-A4B**
4. **BailingMoe2 16B.A1B Q4_K_M**
5. **SmolLM3 3B Q4_K_M**

Pourquoi ce classement ?

- **LFM2** combine vitesse, tenue en contexte et vraie utilité pratique.
- **Qwen3.6** combine très haute qualité et stabilité étonnamment bonne.
- **Gemma** est probablement le plus fiable, mais plus lent en génération.
- **BailingMoe2** est fabuleux en vitesse, mais moins complet intellectuellement.
- **SmolLM3** reste une option très saine si l’on veut un petit modèle dense vraiment exploitable.

---

## 9. Recommandations pratiques selon le besoin

### 9.1 Pour un assistant principal local polyvalent

Choix recommandés :

- **Gemma-4-26B-A4B**
- **Qwen3.6-35B-A3B**

Ce sont les meilleures options si la priorité est la **qualité de réponse**.

### 9.2 Pour un assistant local rapide et étonnamment rentable

Choix recommandés :

- **LFM2-8B-A1B-Q4_K_M**
- **BailingMoe2 16B.A1B Q4_K_M**

Ce sont les options les plus convaincantes si la priorité est le **rapport vitesse / intérêt réel**.

### 9.3 Pour une machine encore plus contrainte ou des tâches simples

Choix recommandés :

- **SmolLM3 3B Q4_K_M**
- **Qwen2 3B Q4_K**

À réserver aux usages où l’on accepte un plafond qualitatif plus bas en échange d’une très bonne légèreté.

---

## 10. Rejouer les tests : points d’attention importants

### 10.1 Cas particulier de LFM2

Si quelqu’un veut refaire les tests sur **LFM2-8B-A1B-Q4_K_M.gguf**, il faut noter deux points pratiques importants :

1. **le modèle a besoin de `--jinja`** ;
2. il faut **supprimer les réglages `q8` sur K et V** quand ils bloquent le lancement du serveur.

En clair : ne pas supposer que la configuration KV utilisée ailleurs s’applique telle quelle à LFM2 côté `llama-server`.

### 10.2 Ne pas surinterpréter les tests “à vide”

Un benchmark réussi sur `pp1024` et `tg128` sans profondeur ne suffit pas. Il faut toujours vérifier au minimum :

- `d1024`
- `d2048`
- `d4096`

Sans cela, on peut facilement garder un modèle qui paraît excellent mais se dégrade trop vite en usage réel.

### 10.3 Toujours séparer vitesse et qualité

Le protocole du rapport confirme qu’il faut garder deux couches d’évaluation :

- **bench technique rapide** pour filtrer ;
- **bench capacité court mais identique** pour arbitrer.

C’est cette combinaison qui a permis d’éviter plusieurs faux bons candidats et de faire ressortir les vrais profils rentables.

---

## 11. Ce que ce travail apporte vraiment

Au-delà du classement ponctuel, cette campagne apporte surtout de la connaissance réutilisable.

### 11.1 Elle montre une logique de sélection

Pour une vieille machine CPU-only, la bonne stratégie n’est pas “prendre le plus gros modèle qui démarre”, ni “prendre un 8B par réflexe”.

La logique la plus saine est plutôt :

1. filtrer par **coût machine réel** ;
2. observer la **tenue en contexte** ;
3. ne garder que les modèles qui ont ensuite une **vraie utilité qualitative**.

### 11.2 Elle clarifie le paysage des modèles locaux

Le rapport montre clairement trois familles utiles :

- les **MoE à faible actif**, souvent surprenants sur CPU ;
- les **petits denses bien entraînés**, très utiles pour l’agilité ;
- les **gros généralistes qualitatifs**, à réserver quand on accepte leur poids.

### 11.3 Elle évite plusieurs erreurs classiques

Ce travail aide à éviter des impasses fréquentes :

- croire qu’un modèle plus gros est automatiquement meilleur choix ;
- croire qu’un score brut à vide suffit ;
- croire que le milieu de gamme dense sera naturellement optimal ;
- oublier que la qualité réelle peut casser une belle impression de vitesse.

---

## 12. Conclusion générale

Sur cette machine **i7-7700 / 32 Go / sans GPU**, plusieurs enseignements ressortent avec netteté.

### Leçon 1

Les **MoE à faible nombre de paramètres actifs** sont la meilleure surprise de la campagne. Ils peuvent offrir des performances franchement impressionnantes sur CPU-only.

### Leçon 2

Le **meilleur compromis global** observé est probablement **LFM2-8B-A1B-Q4_K_M** : rapide, stable en profondeur, et réellement utile.

### Leçon 3

Si l’on veut la **meilleure qualité générale**, les deux noms à retenir sont **Gemma-4-26B-A4B** et **Qwen3.6-35B-A3B**.

### Leçon 4

Les **petits modèles denses** gardent une vraie place, mais surtout comme outils rapides, légers et ciblés, pas comme remplaçants universels.

### Leçon 5

La vraie question n’est jamais seulement “combien de t/s ?”, mais plutôt :

> **combien de t/s pour quel niveau de qualité, et avec quelle stabilité quand le contexte devient réel ?**

C’est précisément ce que ces expérimentations ont permis de rendre visible.

---

## 13. Recommandation finale en une ligne

Pour quelqu’un qui veut continuer les tests localement sans se disperser :

- garder **LFM2** comme référence “compromis intelligent” ;
- garder **Gemma-4-26B-A4B** et **Qwen3.6-35B-A3B** comme références “qualité” ;
- garder **BailingMoe2** comme référence “vitesse spectaculaire” ;
- garder **SmolLM3 3B** comme référence “petit dense utile”.

C’est un noyau de comparaison cohérent, lisible, et riche en enseignements.

---
