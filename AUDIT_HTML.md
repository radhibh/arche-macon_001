# Audit HTML & conformité scientifique

Fichier audité : `arche-mason-v17-double-arc (5).html`

## Résumé exécutif

- La structure applicative est cohérente (UI + moteur Python embarqué via Pyodide).
- Les formules principales affichées dans le code sont globalement alignées avec les hypothèses classiques (SETRA/Heyman/Coulomb/Jaky).
- **Une non-conformité physique importante a été identifiée et corrigée** : pour le glissement Coulomb, le code pouvait retourner `τ_gliss = 0` quand `N → 0` et `V > 0`, ce qui est non conservatif.
- Des incohérences de versionnement restent présentes (v8/v14/v15/v16).

## Méthode d'audit

1. Revue statique du moteur de calcul embarqué (bloc Python dans le HTML).
2. Vérification des équations implémentées vs hypothèses mécaniques explicites dans les commentaires du code.
3. Contrôle syntaxique JS des scripts inline.
4. Tentative de lecture des PDF de référence fournis dans le dépôt.

## Conformité scientifique (constats)

### 1) Poussée des terres — coefficients K

Implémentation :
- Au repos : `K0 = 1 - sin(φ)` (Jaky)
- Active : `Ka = tan²(π/4 - φ/2)` (Rankine)
- Passive : `Kp = tan²(π/4 + φ/2)` (Rankine)

✅ Ces expressions sont conformes aux relations usuelles en géotechnique (sous hypothèses standard de Rankine/Jaky).

### 2) Vérification flexion-compression (NTR)

Le moteur applique :
- noyau central : `σ_max = (N/S)·(1 + 6e/t)` et `σ_min = (N/S)·(1 - 6e/t)`
- hors noyau : zone comprimée réduite `b_c = 3·(t/2 - e)` puis `σ_max = 2N/(B·b_c)`
- excentricité admissible : `e_adm = h·(1 - τ_c)` avec `τ_c = N/(σ0·S)`

✅ Ces formulations sont cohérentes avec un modèle section rectangulaire NTR en compression (approche type SETRA / résistance des matériaux simplifiée).

### 3) Glissement Coulomb

Modèle utilisé : `|V| <= N·tan(φ)`.

⚠️ **Erreur détectée (corrigée)** : le code traitait le cas `V_resist <= tol` avec `τ_gliss = 0`, même si `V > 0`.
- Physiquement, si `N = 0` et `V > 0`, la résistance de frottement est nulle ⇒ taux de glissement doit être **infini** (ou au minimum non valide).
- Correction appliquée :
  - `τ_gliss = Infinity` si `V_resist <= tol` et `|V| > tol`
  - `τ_gliss = 0` seulement si `V ≈ 0`.

## Conformité documentaire (PDF du dépôt)

Les documents PDF présents (`DELBEQ`, `STABLON`, `AFGC`, `clubOA`) n'ont pas pu être exploités automatiquement dans cet environnement car :
- pas d'outil OCR/texte PDF installé (`pdftotext` absent),
- modules Python PDF non disponibles et installation bloquée (proxy),
- au moins un PDF semble principalement scanné (flux image/JBIG2).

➡️ Conclusion : la vérification « théorie vs documents PDF locaux » est **partielle** ici ; la vérification a donc été faite principalement contre la théorie mécanique explicitée dans le code.

## Incohérences annexes à corriger

### Versionning hétérogène

Le même fichier affiche plusieurs versions (`v16.0`, `v14.0`, `v8.0`, `15.0` dans les exports).

Impact : traçabilité affaiblie, ambiguïté import/export.

Recommandation : centraliser une constante unique `APP_VERSION` réutilisée dans title/footer/logs/exports.

## Vérifications réalisées

- Scripts inline extraits puis vérifiés avec `node --check` : **OK**.
- Recherche ciblée des équations/versions avec `rg` : **OK**.
- Vérification de la correction glissement (`N=0, V>0`) via test de logique ajouté : **OK**.
- Tentatives d'outillage PDF (`pdftotext`, `pypdf`) : **non disponibles / bloquées**.
