## Le principe

Toute la classe travaille sur un même dépôt GitHub. Ce dépôt contient un fichier `vainqueur.txt` qui ne comporte qu'une seule ligne :

```text
Et le vainqueur est ...
```

Le but du jeu est d'y inscrire son prénom et de pousser cette modification sur le dépôt distant :

```text
Et le vainqueur est Léa !
```

Quand l'enseignant annonce la fin de la partie, le vainqueur est **le dernier élève dont le push a été accepté par GitHub**.

Comme tout le monde modifie la même ligne en même temps, les pushs vont être refusés et les conflits vont s'enchaîner. Gagner, c'est donc savoir résoudre un conflit vite et sans erreur. C'est pour cela que la séance commence par une phase d'entraînement individuelle.

La séance se déroule en deux phases :

1. **L'entraînement** : chacun s'exerce seul à la résolution de conflit, dans un fichier qui lui est réservé.
2. **Le jeu** : tout le monde s'affronte sur le fichier `vainqueur.txt`.

## Avant de commencer

### Vérifications à faire par chaque élève

Sous Windows, toutes les commandes de ce document se tapent dans **Git Bash**, pas dans PowerShell ni dans l'invite de commandes. Dans PowerShell, `mkdir -p` n'existe pas et `echo "..." > fichier` peut écrire le fichier dans un encodage (UTF-16) que Git traite comme un fichier binaire, ce qui empêche l'affichage des conflits. Sous Linux et macOS, on utilise le terminal habituel.

On vérifie que son identité Git est configurée :

```bash
git config --global user.name
git config --global user.email
```

Si une des deux commandes n'affiche rien, on la configure :

```bash
git config --global user.name "Prénom Nom"
git config --global user.email "adresse@exemple.fr"
```

On indique à Git qu'un `git pull` doit faire une fusion (merge). Sans ce réglage, certaines versions de Git s'arrêtent et demandent de choisir une stratégie au moment du pull :

```bash
git config --global pull.rebase false
```

On vérifie qu'on sait pousser sur GitHub depuis son poste (clé SSH, jeton d'accès personnel ou Git Credential Manager). Le mot de passe du compte GitHub n'est pas accepté pour les opérations Git.

On accepte l'invitation à collaborer sur le dépôt du jeu. Elle arrive par e-mail et elle est aussi visible à l'adresse `https://github.com/<proprietaire>/boulet-d-or/invitations`. Sans cette invitation acceptée, tous les pushs sont refusés.

### Pourquoi on ajoute toujours `--no-edit`

Quand Git crée un commit de fusion, il ouvre un éditeur de texte dans le terminal (souvent Vim) pour faire valider le message. L'option `--no-edit` accepte directement le message proposé par Git. On l'utilise sur `git pull` et sur `git commit` pendant toute la séance pour ne pas se retrouver bloqué dans l'éditeur.

Si on se retrouve malgré tout dans Vim, on tape `Échap`, puis `:wq`, puis `Entrée` pour enregistrer et quitter.

## Phase 1 : l'entraînement

### Règle de l'entraînement

Pendant l'entraînement, **on ne touche pas au fichier `vainqueur.txt`**. Chacun travaille uniquement dans son propre fichier `entrainement/<prenom>.txt`.

Le nom du fichier est son prénom en minuscules, sans accent et sans espace : `lea.txt`, `jean-marc.txt`. Si deux élèves ont le même prénom, on ajoute l'initiale du nom : `lea-d.txt`, `lea-m.txt`.

### Le dispositif : un élève, deux postes

Pour provoquer un conflit, il faut deux personnes qui modifient la même ligne. On va simuler ces deux personnes en clonant **deux fois** le dépôt sur son ordinateur, dans deux dossiers différents. Chaque dossier joue le rôle d'un développeur distinct : le « poste A » et le « poste B ».

On se place dans le dossier où on range ses projets, puis on clone le dépôt deux fois :

```bash
git clone <url-du-depot> boulet-poste-a
git clone <url-du-depot> boulet-poste-b
```

On ouvre deux terminaux : un dans `boulet-poste-a`, un dans `boulet-poste-b`. Dans la suite, chaque bloc de commandes indique dans quel poste on se trouve.

### Étape 1 : créer et pousser son fichier

Dans le poste A, on crée son fichier avec une seule ligne :

```bash
mkdir -p entrainement
echo "Entraînement de Léa : version 1" > entrainement/lea.txt
git add entrainement/lea.txt
git commit -m "Ajout du fichier d'entraînement de Léa"
git push
```

Deux cas sont possibles.

**Le push est accepté** : personne n'avait poussé depuis notre clone. On passe à l'étape 2.

**Le push est refusé** avec ce message :

```text
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to '...'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally.
```

C'est normal : des camarades ont poussé leur propre fichier entre-temps. Le dépôt distant contient des commits que notre dépôt local n'a pas, et Git refuse d'écraser leur travail. On récupère d'abord leurs commits :

```bash
git pull --no-edit
```

Git affiche `Merge made by the 'ort' strategy.` : il a fusionné tout seul, **sans conflit**, parce que les camarades ont modifié d'autres fichiers que le nôtre. On peut maintenant pousser :

```bash
git push
```

Si le push est encore refusé, c'est qu'un autre camarade a poussé pendant ce temps : on recommence `git pull --no-edit` puis `git push`.

À retenir : **un push refusé ne veut pas dire qu'il y a un conflit**. Il y a conflit seulement quand deux commits modifient la même ligne d'un même fichier.

### Étape 2 : provoquer un conflit

Dans le poste B, on récupère d'abord son fichier, puisque ce clone a été fait avant qu'on le pousse :

```bash
git pull --no-edit
```

Dans le poste A, on modifie la ligne, on commite et on pousse :

```bash
echo "Entraînement de Léa : version poste A" > entrainement/lea.txt
git add entrainement/lea.txt
git commit -m "Modification depuis le poste A"
git push
```

Si ce push est refusé à cause des camarades, on fait `git pull --no-edit` puis `git push`, comme à l'étape 1.

Dans le poste B, **sans faire de pull**, on modifie la même ligne de façon différente, on commite et on pousse :

```bash
echo "Entraînement de Léa : version poste B" > entrainement/lea.txt
git add entrainement/lea.txt
git commit -m "Modification depuis le poste B"
git push
```

Le push est refusé : le dépôt distant contient le commit du poste A. On récupère ce commit :

```bash
git pull --no-edit
```

Cette fois, Git ne peut pas fusionner seul :

```text
Auto-merging entrainement/lea.txt
CONFLICT (content): Merge conflict in entrainement/lea.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Les deux postes ont modifié la même ligne du même fichier. Git ne sait pas quelle version garder et nous demande de décider.

### Étape 3 : résoudre le conflit

On regarde l'état du dépôt :

```bash
git status
```

Git indique `You have unmerged paths` et liste le fichier en conflit avec la mention `both modified`.

On ouvre `entrainement/lea.txt` dans son éditeur. Git y a écrit les deux versions, encadrées par des marqueurs :

```text
<<<<<<< HEAD
Entraînement de Léa : version poste B
=======
Entraînement de Léa : version poste A
>>>>>>> 3a06ca498e04de7aefb3b2c1aaca4c1c7a975a48
```

Voici comment lire ces marqueurs :

| Marqueur             | Signification                                                                                            |
| -------------------- | -------------------------------------------------------------------------------------------------------- |
| `<<<<<<< HEAD`       | Début de notre version, celle du dépôt local (ici le poste B)                                            |
| `=======`            | Séparation entre les deux versions                                                                       |
| `>>>>>>> 3a06ca4...` | Fin de la version qui arrive du dépôt distant (ici le poste A), identifiée par l'empreinte de son commit |

Résoudre le conflit, c'est remplacer tout ce bloc par le contenu final voulu. On peut garder une des deux versions, ou écrire une ligne qui combine les deux. Ici, on décide d'écrire :

```text
Entraînement de Léa : conflit résolu
```

Le fichier ne doit plus contenir aucun marqueur `<<<<<<<`, `=======` ou `>>>>>>>`. On enregistre le fichier.

Dans VS Code, le fichier en conflit affiche au-dessus du bloc les boutons « Accept Current Change », « Accept Incoming Change » et « Accept Both Changes ». Ils font la même chose que l'édition manuelle. On peut les utiliser, à condition de vérifier le résultat avant de continuer.

On signale à Git que le conflit est résolu, on termine la fusion et on pousse :

```bash
git add entrainement/lea.txt
git commit --no-edit
git push
```

Si le push est refusé à cause des camarades, on fait `git pull --no-edit` puis `git push`.

### Étape 4 : resynchroniser le poste A et observer l'historique

Dans le poste A, on récupère la résolution :

```bash
git pull --no-edit
cat entrainement/lea.txt
```

Le fichier contient la ligne `Entraînement de Léa : conflit résolu`. On observe l'historique sous forme de graphe :

```bash
git log --oneline --graph -20
```

On voit les deux lignes de développement (poste A et poste B) qui se rejoignent dans le commit de fusion. Comme toute la classe pousse sur le même dépôt, les commits des camarades apparaissent aussi dans le graphe : on repère les siens grâce à leurs messages « Modification depuis le poste A » et « Modification depuis le poste B ».

### Étape 5 : recommencer en inversant les rôles

On refait les étapes 2 à 4 en inversant les postes : cette fois, c'est le poste B qui pousse en premier et le poste A qui rencontre le conflit. À chaque tour, on change le texte de la ligne pour que les deux postes aient bien des versions différentes.

On doit avoir résolu **au moins deux conflits** (un dans chaque sens) avant de montrer à l'enseignant, dans le résultat de `git log --oneline --graph -20`, les deux commits de fusion qui relient ses commits des postes A et B. L'enseignant valide alors l'entraînement.

### Si ça coince

**Le pull est refusé avec `Your local changes to the following files would be overwritten by merge`.** On a modifié le fichier mais on ne l'a pas commité. Git refuse de faire la fusion pour ne pas perdre ces modifications. On fait `git add` puis `git commit`, et on relance `git pull --no-edit`.

**On ne sait plus où on en est.** On tape `git status` : Git indique toujours l'étape en cours et les commandes possibles.

**On veut annuler une fusion en conflit et revenir à l'état d'avant le pull.** On tape :

```bash
git merge --abort
```

## Phase 2 : le jeu

### Les règles

1. On joue uniquement depuis le poste A (le dossier `boulet-poste-a`). Le poste B n'est plus utilisé.
2. On ne modifie que le fichier `vainqueur.txt`. Aucun autre fichier ne doit être touché pendant le jeu.
3. Le fichier `vainqueur.txt` contient toujours **une seule ligne**, au format exact : `Et le vainqueur est Prénom !`
4. Quand on résout un conflit, on garde sa propre ligne et on supprime celle du camarade.
5. Le `git push --force` est interdit. Il est de toute façon bloqué par une règle de protection sur le dépôt.
6. Personne ne touche au fichier avant le signal de départ.
7. À l'annonce de la fin, on retire les mains du clavier. Tout push arrivé sur GitHub après le signal de fin est annulé.
8. Le vainqueur est **le dernier élève dont le push a été accepté par GitHub** avant le signal de fin.

### Au signal de départ

On récupère l'état le plus récent du dépôt :

```bash
git pull --no-edit
```

### La boucle de jeu

**1. On écrit son prénom** dans `vainqueur.txt`, soit dans l'éditeur, soit avec la commande :

```bash
echo "Et le vainqueur est Léa !" > vainqueur.txt
```

**2. On commite et on pousse :**

```bash
git add vainqueur.txt
git commit -m "Léa prend la tête"
git push
```

**3a. Le push est accepté** : on est en tête. On passe à la section « Reprendre la tête ».

**3b. Le push est refusé** : quelqu'un a poussé avant nous. On récupère son commit :

```bash
git pull --no-edit
```

Git signale un conflit sur `vainqueur.txt`. On ouvre le fichier, on supprime les marqueurs et la ligne du camarade pour ne garder que :

```text
Et le vainqueur est Léa !
```

On enregistre, puis on termine la fusion et on pousse :

```bash
git add vainqueur.txt
git commit --no-edit
git push
```

Si le push est encore refusé, un autre camarade a été plus rapide : on revient au début de l'étape 3b.

### Reprendre la tête

Être en tête ne suffit pas : seul compte le dernier push avant le signal de fin. Les autres vont donc nous dépasser. Pour savoir si c'est le cas, on interroge le dépôt distant :

```bash
git fetch
git status
```

Si Git indique `Your branch is behind 'origin/main'`, quelqu'un a poussé après nous. On récupère son commit, puis on repart à l'étape 1 de la boucle :

```bash
git pull --no-edit
```

Ce pull ne provoque pas de conflit : on n'a aucun commit local en attente, Git avance simplement notre branche jusqu'au commit du camarade (une « avance rapide », ou fast-forward). Le fichier contient alors son prénom, on le remplace par le nôtre.

### La fin de la partie

Au signal de fin, l'élève propriétaire du dépôt active la règle qui gèle la branche `main` : plus aucun push n'est accepté.

On désigne ensuite le vainqueur ensemble, en projetant la page **Activity** du dépôt (`https://github.com/<proprietaire>/boulet-d-or/activity`). Elle liste chaque push avec le compte GitHub qui l'a effectué et l'heure à laquelle GitHub l'a reçu. Le vainqueur est l'auteur du dernier push reçu avant le signal de fin.

Ce n'est ni le prénom écrit dans le fichier (on peut y écrire n'importe quel nom), ni la date affichée par `git log` (c'est la date de création du commit, fournie par l'horloge de l'ordinateur de l'élève, pas la date du push) qui font foi.

## Aide-mémoire

| Ce que Git affiche                                     | Ce que ça veut dire                                 | Ce qu'on fait                                                           |
| ------------------------------------------------------ | --------------------------------------------------- | ----------------------------------------------------------------------- |
| `! [rejected] main -> main (fetch first)`              | Le dépôt distant contient des commits qu'on n'a pas | `git pull --no-edit`                                                    |
| `Merge made by the 'ort' strategy.`                    | Fusion automatique réussie, pas de conflit          | `git push`                                                              |
| `CONFLICT (content): Merge conflict in ...`            | Deux commits ont modifié la même ligne              | On édite le fichier, puis `git add`, `git commit --no-edit`, `git push` |
| `Your local changes ... would be overwritten by merge` | Des modifications ne sont pas commitées             | `git add`, `git commit`, puis `git pull --no-edit`                      |
| `Your branch is behind 'origin/main'`                  | Quelqu'un a poussé après nous                       | `git pull --no-edit`                                                    |
| `You have unmerged paths`                              | Une fusion en conflit n'est pas terminée            | On résout le conflit, puis `git add` et `git commit --no-edit`          |
