# Git rebase

Ce tuto a pour but de présenter l’utilisation de la command git rebase. La commande git rebase permet de rejouer les commits d’une branche sur une autre. Un rebase peut notamment être utilisé afin d’effectuer de la réécriture d’historique de commit et de conserver une arborescence du repo la plus continue et lisible possible.

##Préambule

Avant de dérouler ce tuto, quelques points sur la forme :

* Je n’affiche pas dans le document les retours de console pour éviter de trop surcharger le texte sauf quand cela me semble pertinent pour le sujet traité. 
* Des commentaires précédés d’un # sont présent dans le texte pour faciliter la compréhension des opérations.

##Partie 1

Avant de montrer l’utilisation de git rebase, nous allons dérouler un exemple très simple de merge. Pour cela, se placer dans un répertoire vide de votre choix.

\# initialisation du repo

	git init

A l’initialisation du repo, vous devriez être positionné par défaut sur master.

Un premier commit sur master

\# création d’un fichier file.txt avec un ligne de text ‘Hello world’

	echo 'Hello world' > file.txt

\# ajout du fichier à l’index du repo et commit

	git add file.txt

	git commit -m 'my new file.txt'
	[master (root-commit) 755bfd8] my new file.txt
	1 file changed, 1 insertion(+)
 	create mode 100644 file.txt


Une premier fichier file.txt avec le contenu (hautement original :)) ‘Hello world’  a été commité sur master

Création et switch sur une branche A

	git checkout -b A
	Switched to a new branch 'A'

\#Ajout d’une ligne à file.txt

	echo 'Hello Paris' >> file.txt
	git commit -am 'first change in file.txt'
	96d5486] first change in file.txt
 	1 file changed, 1 insertion(+)

\#Ajout d’une deuxième ligne

	echo 'Hello Tokyo' >> file.txt

\#cat file.txt

\#Hello world

\#Hello Paris

\#Hello Tokyo

	git commit -am 'second change in file.txt'

Deux  commits ont été réalisés sur la branche A altérant le contenu du fichier file.txt.

###3. Second commit sur master

	git checkout master

###création d’un nouveau fichier sur master
	echo 'other things' > other.txt

	git add other.txt

###commit du nouveau
	git commit -m 'add one file'



Etat de l’arborescence de notre repo jusqu’à présent:

Pour visualiser l’aborescence de votre repo, vous pouvez faire un 

	git log --graph --decorate --all

Mais pour ce tuto j’utilise GitExtensions (https://code.google.com/p/gitextensions/) :

###Merging time

On va merger le contenu de A dans Master

	git checkout master

	git merge A
	Merge made by the 'recursive' strategy.
	 file.txt | 2 ++
	 1 file changed, 2 insertions(+)

\#On peut supprimer A
	git branch -d A
	Deleted branch A (was 73d828f).

Dans le cas présent, l’exemple est assez simple, mais on peut facilement s’imaginer que notre repo git puisse rapidement devenir une autoroute à n voies qui finissent par s’entrecroiser les unes les autres. 

Autre point, imaginons que master soit notre branche d’échange/intégration avec d’autres développeurs. Les autres développeurs ne sont peut être pas intéressés par les n commit effectuée sur la branche A pour arrivée à la fonctionnalité attendue méritant d’être mergée et publiée sur master.

### Partie 2

Idéalement, avant de dérouler la suite du tuto, recréer un repo dans un répertoire vide et dérouler les étapes 1,2 et 3 de la partie 1.

L’arborescence de votre repo devrait être la suivante


Rebase in action

#On se remet sur A

	git checkout A

#On lance le rebase de A à partir de master
	git rebase master
	First, rewinding head to replay your work on top of it...
	Applying: first change in file.txt
	Applying: second change in file.txt

Vision de l’arborescence git


Les commit de A ont été rejoués sur master à partir du commit de tête de la branche master.  Nous obtenons donc un arbre linéaire, comme si la branche A avait été initialement tirée à partir du second commit sur master. On peut donc effectuer un merge en fast-forward de A dans master.

	git checkout master

	git merge A
	Updating e3d6710..be78444
	Fast-forward
	file.txt | 2 ++
	1 file changed, 2 insertions(+)



### Les risques

Il est important de comprendre que les commits préalablement sur A n’ont pas été déplacés vers master mais plutôt que le contenu des commits de A a été rejoués en tant que nouveaux commits à partir du commit de tête de master. D’ailleurs si l’on compare le sha1 des commits sur A pré et post rebase, on se rendra compte que ce ne sont pas les mêmes.

En conséquence, c’est pourquoi il ne faut jamais effectuer un rebase d’un branche vers une autre si les commits concernés par le rebase ont déjà été rendu public aux autres développeurs. Prenons deux développeurs A et B. A effectue des commits sur sa branches et les publie. B récupère le travail de A et base la suite de son travail à la suite des commits de A. A rebase ses commits et les publie de nouveaux. Lorsque B va de nouveaux récupérer le travail de A, il va récupérer de nouveaux tout le contenu des commits de A puisque ceux-ci seront vu comme de nouveaux commits (+ tous les conflits de merge qui iront avec...) 

### Interactive rebase

Une fonctionnalité particulièrement intéressant du rebase est son option interactive rebase (-i) qui va nous permettre de réécrire l’historique des commit.

	git checkout A
	Switched to branch 'A'

	echo 'one more line 1' >> file.txt

	git commit -am 'add one more line'
	4ed28f5] add one more line
	 1 file changed, 1 insertion(+)

	echo 'one more line 2' >> file.txt

	git commit -am 'add one more line'
	035c2f5] add one more line
	 1 file changed, 1 insertion(+)

	echo 'one more line 3' >> file.txt

	git commit -am 'add one more line'
	d8331a1] add one more line
	 1 file changed, 1 insertion(+)


A partir de là, on pourrait très bien faire un merge de A sur master en fast-forward. Cependant pour peu que master soit la branche remote sur laquelle les autres développeurs viennent se synchroniser, les autres développeurs ne sont peut être pas intéressé par le fait que nous ayons fait 3 commit différents pour ajouter 3 lignes à notre fichier file.txt.

	git rebase -i master

L’éditeur s’ouvre avec le contenu suivant

	pick 4ed28f5 add one more line
	pick 035c2f5 add one more line
	pick d8331a1 add one more line

	# Rebase be78444..d8331a1 onto be78444
	#
	# Commands:
	#  p, pick = use commit
	#  r, reword = use commit, but edit the commit message
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit
	#  f, fixup = like "squash", but discard this commit's log message
	#  x, exec = run command (the rest of the line) using shell
	#
	# These lines can be re-ordered; they are executed from top to bottom.
	#
	# If you remove a line here THAT COMMIT WILL BE LOST.
	#
	# However, if you remove everything, the rebase will be aborted.
	#
	# Note that empty commits are commented out

Les 3 commits 4ed28f5, 035c2f5, d8331a1 sont bien présents. 

Nous allons compiler dans le commit 4ed28f5 les commits 035c2f5 et d8331a1 grâce à la commande squash.

	pick 4ed28f5 add one more line
	squash 035c2f5 add one more line
	squash d8331a1 add one more line

	# Rebase be78444..d8331a1 onto be78444
	#
	# Commands:
	#  p, pick = use commit
	#  r, reword = use commit, but edit the commit message
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit
	#  f, fixup = like "squash", but discard this commit's log message
	#  x, exec = run command (the rest of the line) using shell
	#
	# These lines can be re-ordered; they are executed from top to bottom.
	#
	# If you remove a line here THAT COMMIT WILL BE LOST.
	#
	# However, if you remove everything, the rebase will be aborted.
	#
	# Note that empty commits are commented out


git rebase - affiche un historique du haut vers le bas. On pick le premier commit (il en faut au moins 1), et on squash les suivants dans le premier. On enregistre et on valide le rebase.

Une nouvelle fenêtre s’ouvre, cette fois, pour saisir le commentaire du commit. 

	# This is a combination of 3 commits.
	# The first commit's message is:
	add one more line

	# This is the 2nd commit message:

	add one more line

	# This is the 3rd commit message:

	add one more line

	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# rebase in progress; onto be78444
	# You are currently editing a commit while rebasing branch 'A' on 'be78444'.
	#
	# Changes to be committed:
	#	modified:   file.txt
	#


Les commentaires de commit originels sont affichés mais vous pouvez très bien saisir un tout nouveau commentaire en supprimant l’intégralité des lignes affichées.

	\# Vider le fichier et écrire
	add 3 lines

	[detached HEAD b642d6b] add 3 lines
 	1 file changed, 3 insertions(+)
	Successfully rebased and updated refs/heads/A.


L’historique sur A a bien été réécrit et les 3 lignes ajoutées apparaissent sous la forme d’un seul et unique commit.

	git checkout master

	git merge A

	slemoine@SQLI50066-6 /d/SLE/WK/tutogit5 (A)
	$ git checkout master
	Already on 'master'

	slemoine@SQLI50066-6 /d/SLE/WK/tutogit5 (master)
	$ git merge A
	Updating be78444..b642d6b
	Fast-forward
 	file.txt | 3 +++
 	1 file changed, 3 insertions(+)


On peut désormais mettre à dispo le contenu de notre travail aux autres développeur avec un git push origin master.
