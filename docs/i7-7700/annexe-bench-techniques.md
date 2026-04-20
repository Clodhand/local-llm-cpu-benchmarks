## Annexe B — Tableau récapitulatif des tests techniques par modèle

Les valeurs ci-dessous correspondent au protocole de comparaison retenu sur cette machine, avec lecture des points suivants :

- `pp1024` : préfill ;
- `tg128` : génération ;
- `d1024`, `d2048`, `d4096` : profondeur de contexte.

| Modèle | Type | Taille | pp1024 | tg128 | pp1024 @ d1024 | tg128 @ d1024 | pp1024 @ d2048 | tg128 @ d2048 | pp1024 @ d4096 | tg128 @ d4096 |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| BailingMoe2 16B.A1B Q4_K_M | MoE | 9.22 GiB | 128.28 | 35.10 | 113.67 | 30.08 | 100.62 | 25.66 | 82.31 | 18.02 |
| LFM2-8B-A1B-Q4_K_M | MoE | 4.70 GiB | 88.36 | 28.90 | 85.43 | 27.78 | 82.26 | 25.97 | 77.30 | 22.68 |
| Qwen2 3B Q4_K | Dense | 1.79 GiB | 47.36 | 14.67 | 35.82 | 13.40 | 28.98 | 12.28 | 21.11 | 10.51 |
| SmolLM3 3B Q4_K_M | Dense | 1.78 GiB | 46.48 | 14.85 | 34.88 | 13.30 | 28.20 | 12.21 | 20.52 | 10.30 |
| SmolLM2 1.7B (`llama-1.7B-Instruct-Q4_K_M`) | Dense | 1005 MiB | 72.92 | 26.46 | 47.34 | 20.34 | 34.78 | 15.75 | 19.50 | 9.16 |
| Qwen3-8B-Q4_K_M | Dense | 4.68 GiB | 19.79 | 6.22 | 15.37 | 5.55 | 12.54 | 4.95 | 9.05 | 3.54 |
| Qwen3-Coder-30B-A3B Q4_K_M | MoE | 17.28 GiB | 31.55 | 14.32 | 19.69 | 10.96 | 14.60 | 8.91 | 9.47 | 6.36 |
| Gemma-4-26B-A4B | MoE | 15.70 GiB | 31.65 | 7.18 | 29.62 | 6.87 | 28.57 | 6.35 | 26.56 | 5.59 |
| Qwen3.6-35B-A3B | MoE | 19.91 GiB | 23.74 | 8.51 | 30.17 | 9.90 | 28.27 | 9.91 | 24.97 | 9.27 |

### Lecture rapide de l’annexe technique

Quelques tendances sautent immédiatement aux yeux :

- **BailingMoe2** et **LFM2** dominent très nettement en débit brut.
- **LFM2** est remarquable par sa **stabilité en profondeur**.
- **Qwen3.6-35B** est beaucoup plus stable en long contexte que ce que sa taille pourrait laisser craindre.
- **Qwen3-Coder-30B** reste fort, mais sa décroissance avec le contexte est bien plus visible.
- **Gemma-4-26B-A4B** n’est pas le plus rapide, mais garde un profil très propre et régulier.
- Les **petits modèles denses** restent utiles, mais leur plafond qualitatif apparaît plus vite.
