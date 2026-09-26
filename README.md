# 🛡️ Exercice 2 – Mettre en place un workflow GitHub sécurisé (CI + revue de code)

## 📌 Contexte général
Vous travaillez sur un projet hébergé sur GitHub.
La branche `main` représente la version stable du projet.

Afin de garantir la qualité du code et de limiter les erreurs, l'équipe décide de mettre en place :
- des branches de fonctionnalité
- une intégration continue (CI)
- une protection stricte de la branche `main`
- une revue de code obligatoire avant chaque merge

---

## 🎯 Ce que vous devez comprendre et savoir faire
À la fin de cet exercice, vous devez être capable de :
- **Expliquer à quoi sert une CI** : chaque proposition de changement est vérifiée automatiquement, de la même façon pour tout le monde, avant d'arriver sur `main`.
- **Écrire un workflow GitHub Actions** simple : quand il se déclenche (`on:`), ce qu'il exécute (`jobs:` / `steps:`), et où lire son résultat.
- **Protéger `main`** pour que personne, pas même vous, ne puisse y pousser directement.
- **Rendre la CI bloquante** : une PR dont la CI est rouge ne peut pas être mergée.
- **Exiger une revue de code**, et expliquer pourquoi une approbation doit être refaite après un nouveau commit.
- **Dire ce que vous engagez en approuvant une PR** : relire, comprendre, questionner, pas juste cliquer sur « Approve ».

---

## ⚠️ Avant de commencer
- **Travaillez en binôme (A et B).** Chacun crée son propre dépôt et invite l'autre : GitHub n'autorise pas l'auteur d'une pull request à l'approuver lui-même, il vous faut donc un relecteur.
- **Créez un dépôt public.** Sur un compte GitHub gratuit, la protection de branche ne s'applique qu'aux dépôts publics. Un dépôt privé ne fonctionne qu'avec GitHub Pro (gratuit via GitHub Education, si vous l'avez activé).

---

## 🧩 PARTIE 1 – Initialisation du projet GitHub

### 1. Création du dépôt `[CHACUN]`
1. Créez un nouveau dépôt GitHub **public** appelé `secure-your-repository`.
2. Cochez **Add a README file** pour l'initialiser.
3. Vérifiez que la branche par défaut s'appelle `main`.
4. Dans **Settings → Collaborators**, invitez votre binôme. Acceptez l'invitation que vous recevez de votre côté.

### 2. Création d'une branche de fonctionnalité
Récupérez votre dépôt en local, puis créez une branche :
```bash
git clone https://github.com/<votre-compte>/secure-your-repository.git
cd secure-your-repository
git switch -c feature/hello
```

Créez un fichier `hello.txt` contenant :
```
Hello DevOps!
```

> Ne poussez pas encore : on ajoute d'abord la CI dans la partie 2.

---

## 🧩 PARTIE 2 – Mise en place de l'intégration continue (CI)

### 3. Création du workflow GitHub Actions
Sur la même branche, créez le fichier :
```
.github/workflows/ci.yml
```

Contenu :
```yaml
name: CI

on:
  pull_request:
    branches: [ "main" ]

jobs:
  test:
    runs-on: ubuntu-26.04
    steps:
      - uses: actions/checkout@v7
      - name: Run tests
        run: |
          echo "Running tests..."
          echo "Tests OK"
```

> `runs-on: ubuntu-26.04` : la machine qui exécute le job, ici la dernière version LTS d'Ubuntu. On fixe la version plutôt que d'écrire `ubuntu-latest`, qui change de version sans prévenir (et casse parfois un pipeline qui marchait la veille).

Puis committez et poussez les deux fichiers :
```bash
git add hello.txt .github/workflows/ci.yml
git commit -m "feat: add hello feature and CI workflow"
git push -u origin feature/hello
```

### 4. Vérification de la CI
1. Sur GitHub, ouvrez une **Pull Request** de `feature/hello` vers `main`. **Ne la mergez pas.**
2. Vérifiez que la CI se lance automatiquement (onglet **Checks** de la PR, ou onglet **Actions**) et qu'elle passe au vert.

> 📌 Cette première exécution est indispensable : GitHub ne propose le check `test` dans les règles de protection (partie 3) qu'après l'avoir vu tourner au moins une fois.

---

## 🧩 PARTIE 3 – Protection de la branche `main`

Dans **Settings → Branches**, cliquez sur **Add classic branch protection rule** avec le motif `main`, puis activez les options ci-dessous.

### 5. Blocage des push directs sur `main`
- ✅ **Require a pull request before merging**

### 6. Revue de code obligatoire
Dans la même option :
- ✅ **Require approvals** (minimum : 1)
- ✅ **Dismiss stale pull request approvals when new commits are pushed**

### 7. CI bloquante
- ✅ **Require status checks to pass before merging**
- Dans la recherche, sélectionnez le check **`test`** (le nom du job dans `ci.yml`).

### 8. Pas d'exception pour l'administrateur
- ✅ **Do not allow bypassing the above settings**

> ⚠️ Sans cette option, la règle **ne s'applique pas à vous** : vous êtes administrateur de votre propre dépôt, et vos push directs sur `main` passeraient quand même.

Enregistrez la règle (**Create**).

---

## 🧩 PARTIE 4 – Vérifications

### Test 1 – Push direct sur `main`
```bash
git switch main
git pull
git commit --allow-empty -m "test: direct push"
git push origin main
```
Résultat attendu : **push refusé**, avec un message du type `GH006: Protected branch update failed`.
Annulez ensuite ce commit local, qui ne sera jamais accepté :
```bash
git reset --hard origin/main
```

> Si le push passe, vérifiez l'étape 8.

### Test 2 – Merge sans review
Retournez sur votre Pull Request `feature/hello`.
Résultat attendu : **merge bloqué**, avec la mention *Review required*.

Demandez à votre binôme de relire la PR (**Files changed → Review changes**) : un commentaire, puis **Approve**.
Le merge devient possible : mergez.

### Test 3 – CI en échec
```bash
git switch main
git pull
git switch -c feature/ci-rouge
```
Dans `.github/workflows/ci.yml`, remplacez les deux lignes `echo` par `exit 1`, puis committez, poussez et ouvrez une PR.

Résultat attendu : la CI passe au **rouge** et le **merge reste bloqué**, même après l'approbation de votre binôme.

### Test 4 – Approbation périmée (bonus)
Sur une PR déjà approuvée, poussez un nouveau commit.
Résultat attendu : **l'approbation disparaît**, il faut une nouvelle revue.

---

## 🏁 Conclusion
Ce TP illustre un workflow DevOps moderne combinant CI, revue de code et protection de branche.

## ❓ Questions de réflexion
1. Suite à cet exercice, retournez voir le contenu de [test-driven-development](https://github.com/corentinbeuchet/test-driven-development), ou du code que vous aviez récupéré. Que constatez-vous en lisant le contenu du fichier [ci.yml](https://github.com/corentinbeuchet/test-driven-development/blob/main/.github/workflows/ci.yml) ?
2. Êtes-vous convaincu de l'utilité de la mise en place d'un workflow CI/CD ?
3. Pensez-vous que cette protection de la branche `main` suffit à garantir la qualité du code ?
4. Dans le test 3, la CI « teste » avec un simple `echo`. Que faudrait-il pour que ce check ait vraiment de la valeur ? (→ exercice suivant : `automated-tests`)
