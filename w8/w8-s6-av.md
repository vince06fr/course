# w8-s6

## Résumé de la vidéo

Maintenant qu'on a vu les extensions
asynchrones du langage, nous allons
pouvoir nous intéresser d'un peu plus
près aux boucles d'événements.


Jusqu'ici nous avons toujours utilisé
nos boucles avec la méthode
run_until_complete.
========== [fragment run_complete]

Cette méthode attend en argument
exactement un objet; du coup nous avons
utilisé gather pour montrer des exemples
avec des traitements concurrents.

========== [fragment gather]


Mais bon, on ne peut pas toujours faire
ça aussi simplement; il y a des cas bien
entendu où il faut ajouter des
traitements à la volée, et voyons
comment ça se présente alors.

========== [slide ajout de traitements]

La librairie propose deux autres
mécanismes de base qui sont

* ensure_future, pour ajouter un
  traitement dans la boucle, avant ou
  après qu'elle a commencé

==========[fragment run_forever]

* et aussi run_forever, qui fait tourner
  une boucle d'événements indéfiniment;
  cette méthode sur les boucles ne prend
  aucun argument, ce qui suppose qu'on a
  donc rempli la boucle au préalable
  avec, justement, ensure_future

[[AL: pourquoi au préalable, on peut pas ajouter ensuite de nouvelles
coroutines avec ensure future]]
[[TP: parce qu'il faut qu'il y ait au
moins une tâche dans la boucle quand on
fait run_forever(), qui ne prend aucun
argument]]

On va voir ça sur plusieurs exemples

========== [slide utilitaire #1]

Mais avant, je vais utiliser quelques
utilitaires annexes pour bien visualiser
ce qui se passe.

J'ai écrit dans un module annexe deux
fonctions très simples qui sont start_timer et
show_timer; elles s'utilisent comme ceci

[[AL: n'oublie pas de déplacer asynchelpers.py dans ./w8]]
[[TP: je l'ai fait le temps que tu
relises, mais je devrai le remettre
comme avant si j'utilise asynchelpers
dans les compléments.]]

========== [fragment start_timer]

Avec start_timer, on enregistre un
instant de référence, ce sera pour nous
le début de l'exécution, puis avec
show_timer on peut voir le temps écoulé
depuis cette référence

========== [run cell start_timer + stop_timer]

Je vous laisse à titre d'exercice imaginer comment vous pourriez
écrire ces deux fonctions.

========== [slide utilitaire #2]

Je me définis encore une coroutine
utilitaire - je l'ai appelé sequence -
qui écrit simplement un message avant et
après un certain délai. C'est donc juste
pour montrer le début et la fin d'une
coroutine.

========== [slide boucle sans fin]

Bien. Je peux maintenant vous montrer un  
exemple de base, qui lance en même temps  
deux petits traitements.

========== [fragment avec ensure_future]

à ce stade la boucle n'est pas active
mais les deux coroutines ont été
ajoutées, elles seront exécutées par la
boucle dès qu'elle sera démarrée

========== [slide run_forever]

ce que peux faire avec run_forever.

========== [run cell]

sauf que, comme son nom l'indique
run_forever ne va jamais retourner,
aussi je vais devoir l'interrompre ici
avec une interruption de mon kernel.

========== [run_until_complete vs run_forever]

Voici un résumé de ces deux méthodes, en
gros run_until_complete se prêtera donc
plutôt si vous avez besoin de glisser un
traitement asynchrone au milieu d'une
application par ailleurs essentiellement
synchrone, alors que run_forever se
prêtera mieux pour une application
massivement asynchrone.


========== [slide new_event_loop()]

À ce stade de mon notebook, il est temps
que je vous parle de `get_event_loop` et
similaires. Ce n'est pas forcément le
coté du langage le plus réussi, raison
de plus donc pour l'expliquer.

Vous avez peut-être remarqué qu'on n'a
jamais jusqu'ici eu besoin de créer un
objet loop, on a utilisé
get_event_loop()

Cette méthode est un peu surchargée;
l'idée c'est qu'il peut y avoir
plusieurs boucles dans une application,
mais on veut éviter de devoir propager
l'objet boucle en le passant en
paramètre à toutes vos fonctions, car ça
alourdit considérablement le code. Aussi
le modèle c'est que chaque thread
possède une boucle, à laquelle on peut
accéder de n'importe où dans le code
avec get_event_loop.

Par contre ce qui est un peu troublant,
et assez mal documenté, dans la 3.6 en
tous cas, c'est que get_event_loop
ne crée pas de boucle, sauf dans le cas
particulier du thread principal.

========== [slide {new,set}_event_loop()]

On peut donc avoir à créer une nouvelle
boucle avec la méthode new_event_loop,
et je vous conseille d'enregistrer cette
nouvelle boucle comme étant le défaut,
ce qui nous donne ceci

========== [fragment set_event_loop]

Notez que dans mon cas j'ai vraiment
besoin de faire cela, parce que la
boucle d'événements qu'on a utilisé il y
a un instant avec run_forever, et que
j'ai interrompue, est en réalité dans un
état instable, on verra dans une autre
séquence qu'il aurait fallu la terminer
proprement, mais pour l'instant je me
contente de continuer mon exposé avec
une boucle toute neuve.

========== [slide Ajout de traitements]

À présent, on va revenir à
ensure_future, et  je vais vous montrer
rapidement qu'on peut aussi l'appeler à
l'intérieur du code asynchrone, pour
ajouter à la volée des traitements dans
une boucle. Imaginons que je veuille
implémenter un scénario comme celui-ci

========== [fragment figure fork]

Ici on veut au milieu de la couroutine 
c2 démarrer la coroutine c3, sans s'en 
soucier davantage. 

Pour faire ça, je ne peux pas ajouter
les 3 coroutines au démarrage de ma
boucle, car c3 commencerait trop tôt. Il
faut donc faire quelque chose à
l'endroit du point rouge. Je vous laisse
à titre d'exercice vous convaincre que
ni await ni gather ne font exactement ce
qu'on veut ici. Je vais plutôt tout
simplement ajouter c3 à la boucle avec
ensure_future.

========== slide [async def c1()]
Pour c1 et c3, il n'y a aucun souci

========== fragment [async def c2()]
Pour c2, je me contente d'ajouter c3()
dans la boucle d'événements avec
ensure_future()

========== slide start_timer()

Voici l'exécution, je vous laisse vous
convaincre que c'est ce qu'on veut.

Je vous laisse aussi à titre d'exercice
essayer de lancer la boucle avec
run_until_complete sur un gather de c1
et c2, et de comparer les deux comportements.

<commentaire sur l'intérêt qu'il y
aurait à faire un run_until_complete
pour retourner dans le code synchrone,
faire un truc, et relancer la boucle ensuite>

========== slide Mélange run_*

Dans un dernier exemple je vous montre
rapidement comment une boucle se
comporte quand on mélange
run_until_complete avec
run_forever. Pour ça je me donne
simplement 3 coroutines de longueur 1, 2
et 3.

========== run 


========== slide mélange run*

Alors si j'ajoute la plus courte et la
plus longue dans la boucle avec
ensure_future, et qu'ensuite je fais
run_until_complete sur la moyenne, ce
qui se passe c'est que les 3 coroutines
sont dans la boucle, mais
run_until_complete retourne dès que la
moyenne a terminé.

========== run run_until_complete

Clairement
il lui reste des choses à faire dans
cette boucle, mais elle ne seront
exécutées, naturellement, que lorsqu'on
relance la boucle, par exemple avec un
run_forever.

========== run run_forever

dit autrement, on peut superposer les
appels à run_until_complete pour faire
avancer une boucle en quelque sorte pas
à pas, l'état est naturellement conservé.

========== slide résumé
Nous allons résumer cette séquence, on a
vu que

========== [fragment]
* on peut accéder à la boucle courante -
en fait celle associée au thread courant
- avec get_event_loop()

* on peut avoir besoin de créer une
  boucle, par exemple dans un thread
  secondaire, au travers de
  new_event_loop et set_event_loop

========== [fragment]

* on peut ajouter des traitements dans
  une boucle à tout moment avec ensure_future()

========== [fragment]

* on peut exécuter une boucle pas à pas
  avec run_until_complete, ou
  indéfiniment avec run_forever

==========[slide conclusion]

Enfin je voudrais conclure cette
séquence pour redire que la boucle
d'événements fait partie de la librairie
`asyncio`, du coup tout ce qu'on a vu
dans cette vidéo, et ce sera vrai pour
les suivantes aussi, risque d'être
légèrement différent si vous utilisez
une autre librairie comme curio ou trio.
