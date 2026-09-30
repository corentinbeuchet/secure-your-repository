# 🛡️ Exercice 2 – Mettre en place un workflow GitHub sécurisé (CI + revue de code)

> 🎯 **Priorités** : les parties 1 à 3 et les tests 1, 2, 3 et 5 de la partie 4 sont l'essentiel, ce que vous devrez savoir refaire seul à l'évaluation finale. Le test 4 est pour aller plus loin.

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
- **Faire une revue utile** : repérer ce qui rend un code difficile à lire, à tester ou dangereux, et le dire avec des commentaires précis et constructifs.

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

### Test 5 – Une vraie revue de code
Jusqu'ici, votre binôme a approuvé un `hello.txt`. Une revue sert à autre chose : relire un **vrai** code et trouver ce qui pose problème **avant** le merge.

`[CHACUN]` Partez d'un `main` à jour :
```bash
git switch main
git pull
git switch -c feature/total-panier
```
Ajoutez ces deux fichiers (le code est volontairement mauvais, **ne le corrigez pas avant la revue**), puis ouvrez une PR avec pour description : « Calcul du total du panier ».

`src/Calc.java`
```java
import java.util.List;

public class Calc {

    // t : 1 = client normal, 2 = client fidèle
    public double doIt(List<String[]> l, int t) {
        double r = 0;
        for (String[] x : l) {
            if (t == 1) {
                r = r + Double.parseDouble(x[1]) * 1.2;
            } else if (t == 2) {
                r = r + Double.parseDouble(x[1]) * 1.2 * 0.9;
            } else {
                r = r + Double.parseDouble(x[1]) * 1.2;
            }
        }
        try {
            Notifier.send("https://api.boutique.example/total?token=abc123", r);
        } catch (Exception e) {
        }
        return r;
    }
}
```

`test/CalcTest.java`
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertNotNull;

class CalcTest {
    @Test
    void test1() {
        assertNotNull(new Calc());
    }
}
```

`[BINÔME]` Faites la revue (**Files changed**) :
1. Laissez **au moins 4 commentaires** sur des lignes précises. Pour chacun : le problème, pourquoi c'est un problème, une proposition.
2. Aidez-vous de ces questions : le code fait-il ce que dit la description ? Est-il testé, et le test peut-il échouer ? Un autre développeur le comprend-il sans vous ? Y a-t-il un risque de sécurité ?
3. Terminez par **Review changes → Request changes** (pas « Approve »).

`[AUTEUR]` Corrigez en tenant compte des commentaires, poussez, répondez à chaque commentaire (« corrigé dans … » ou « non, parce que… »), puis redemandez une revue.
Résultat attendu : le binôme approuve la **nouvelle** version, et vous pouvez merger.

> 💡 Un bon commentaire de revue vise le code, pas la personne : « ce nom ne dit pas ce que la méthode calcule, `totalTtc` ? » plutôt que « c'est illisible ». Les questions de forme (indentation, espaces) sont le travail d'un outil (linter) dans la CI, pas du relecteur : vous le mettrez en place à l'exercice 3.

---

## 🏁 Conclusion
Ce TP illustre un workflow DevOps moderne combinant CI, revue de code et protection de branche.

## ❓ Questions de réflexion
1. Ouvrez l'onglet **Actions** du dépôt [manage-security](https://github.com/corentinbeuchet/manage-security/actions) : c'est le pipeline que vous construirez aux exercices 3 et 4 (l'exercice 5 y ajoutera la sécurité). Quelles étapes reconnaissez-vous ? Lesquelles vous semblent nouvelles, et à quoi servent-elles d'après vous ?
2. Êtes-vous convaincu de l'utilité de la mise en place d'un workflow CI/CD ?
3. Pensez-vous que cette protection de la branche `main` suffit à garantir la qualité du code ? Qu'est-ce que la revue du test 5 a trouvé qu'aucun réglage ne pouvait trouver ?
4. Dans le test 3, la CI « teste » avec un simple `echo`. Que faudrait-il pour que ce check ait vraiment de la valeur ? (→ exercice suivant : `automated-tests`)
