# local-llm-cpu-benchmarks

Benchmarks et retours d’expérience sur des LLM en local sur une machine ancienne CPU-only.

## Contexte

Ce dépôt rassemble une série d’expérimentations réalisées sur une machine modeste :

- **CPU** : Intel i7-7700
- **RAM** : 32 Go
- **GPU** : aucun
- **OS** : Debian 13 headless

L’objectif est simple : identifier quels modèles GGUF restent réellement intéressants en usage local sur une machine ancienne, sans GPU.

## Contenu

- [Rapport complet](docs/rapport-final.md)
- [Annexe — tests de capacité](docs/i7-7700/annexe-tests-capacite.md)
- [Annexe — benchmarks techniques](docs/annexe-bench-techniques.md)

## Méthode

Les modèles sont comparés selon deux axes :

- **performance technique** : vitesse de préfill et de génération via `llama-bench`
- **capacité utile** : mini benchmark de 10 prompts couvrant raisonnement, code, compréhension et usage réel

## Quelques conclusions

- les **MoE à faible nombre de paramètres actifs** sont une très bonne surprise sur CPU-only
- **LFM2-8B-A1B-Q4_K_M** ressort comme l’un des meilleurs compromis globaux
- **Gemma-4-26B-A4B** et **Qwen3.6-35B-A3B** dominent en qualité générale
- les **petits modèles denses** gardent un vrai intérêt pour les tâches simples et rapides

## Objectif du dépôt

Ce dépôt sert à :

- documenter les tests
- garder une trace des résultats
- partager une méthode de benchmark simple et réutilisable
- aider d’autres personnes à choisir un LLM local sur une machine limitée

## Licence

Ce contenu est publié sous licence **CC BY 4.0**.  
Vous pouvez le partager et le réutiliser à condition de créditer l’auteur.

Texte complet de la licence :
https://creativecommons.org/licenses/by/4.0/
