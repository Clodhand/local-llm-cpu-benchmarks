## Annexe A — Les 10 tests de capacité

Ces prompts ont servi de base commune pour comparer la qualité utile des modèles. L’intérêt de les conserver ici est double :

- rendre la méthode de notation plus transparente ;
- permettre à quelqu’un de rejouer exactement le même mini-benchmark.

### A. Raisonnement

**Prompt 1 — logique simple**

> Tu as 3 interrupteurs en bas et une ampoule à l’étage. Tu peux manipuler les interrupteurs comme tu veux, mais tu ne peux monter voir l’ampoule qu’une seule fois. Comment déterminer avec certitude quel interrupteur commande l’ampoule ? Réponds de manière courte et claire.

**Prompt 2 — déduction**

> Un serveur web renvoie parfois des erreurs 502 uniquement quand le trafic augmente. La RAM semble correcte, le CPU n’est pas saturé, mais les timeouts montent côté backend. Donne 3 hypothèses plausibles classées par probabilité, puis explique comment les vérifier rapidement.

**Prompt 3 — plan d’action**

> J’ai un vieux PC CPU-only avec 32 Go de RAM, sans GPU, et je veux choisir un LLM local. Donne une méthode simple et pragmatique pour filtrer rapidement les modèles intéressants sans passer 48 heures de benchmark.

### B. Code

**Prompt 4 — écriture de fonction**

> Écris une fonction Python `chunk_text(text, size, overlap)` qui découpe un texte en morceaux de taille fixe avec chevauchement. Gère les cas invalides proprement. Ajoute un petit exemple d’utilisation.

**Prompt 5 — correction de bug**

> Voici un code Python :
>
> ```python
> def dedup(items):
>     seen = []
>     for x in items:
>         if x not in seen:
>             seen.append(x)
>     return sorted(seen, key=lambda x: items.index
> ```
>
> Corrige-le pour qu’il fonctionne correctement et efficacement, puis explique le bug.

**Prompt 6 — shell / parsing**

> Écris une commande shell robuste pour compter le nombre de fichiers `.md` dans un dossier et tous ses sous-dossiers, en évitant les erreurs liées aux espaces dans les noms de fichiers. Donne aussi une variante qui compte les `.pdf`.

### C. Compréhension / synthèse

**Prompt 7 — résumé fidèle**

> Résume ce texte en 5 points maximum, sans inventer d’informations :
>
> « Les modèles MoE peuvent être très intéressants sur CPU-only quand le nombre de paramètres actifs par token reste faible. En revanche, les modèles denses intermédiaires, par exemple entre 7B et 14B, donnent parfois un mauvais compromis sur les machines anciennes : trop lourds pour être rapides, mais pas assez supérieurs qualitativement pour compenser. Les petits modèles bien entraînés, autour de 1B à 3B, peuvent au contraire offrir une très bonne réactivité pour des tâches simples ou modérées. Enfin, la vitesse brute ne suffit pas : il faut aussi regarder la stabilité quand la profondeur de contexte augmente. »

**Prompt 8 — comparaison**

> Compare deux stratégies pour une vieille machine CPU-only :
>
> - prendre un petit modèle dense très rapide ;
> - prendre un MoE plus gros mais avec peu de paramètres actifs.
>
> Fais une réponse structurée avec :
>
> - avantages ;
> - inconvénients ;
> - cas d’usage idéaux ;
> - recommandation finale.

### D. Usage réel

**Prompt 9 — aide pratique**

> Je veux formaliser une méthodologie simple pour benchmarker des LLM en local sur une vieille machine CPU-only. Donne-moi une méthode en 3 phases maximum, claire, réaliste et orientée gain de temps.

**Prompt 10 — message / synthèse humaine**

> Rédige un message court et naturel à envoyer à un ami passionné d’IA pour lui expliquer en quoi les modèles MoE à faible actif sont une bonne surprise sur les vieilles machines CPU-only.

---
