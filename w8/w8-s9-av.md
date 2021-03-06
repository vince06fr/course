# w8-s9

## Résumé de la vidéo

À ce stade nous avons vu les concepts
principaux de la programmation
asynchrone en python, à savoir les
coroutines, les boucles d'événements, et
nos avons survolé le contenu de la
librairie.

Dans cette vidéo, nous allons surtout
nous intéresser aux principaux écueils
que l'on rencontre, surtout quand on
commence à travailler avec ce nouveau
paradigme qui, c'est vrai, peut paraître
un peu déroutant au début.

========== [fragment coroutine fonction
/ coroutine]

Le premier, on l'a déjà signalé mais ça
ira mieux en le répétant, c'est qu'une
fonction coroutine, comme ici

========== [run async def foo()]

la fonction foo, renvoie quand elle est
appelée un *objet* coroutine; et donc ça
ne fait rien,

========== [run foo()]

exactement d'ailleurs comme une fonction
génératrice qui lorsqu'on l'appelle

========== [fragment]

retourne un générateur qui est un itérateur.
Je n'ai pas choisi cet exemple au
hasard, la ressemblance entre ces deux
objets est très profonde.

========== [slide tous les scénarios]

Quoi qu'il en soit, il est donc très
important d'utiliser à bon escient les
fonctions standard et les fonction coroutines
dans le bon contexte.

========== [fragment def foo()]
Voici un résumé du cas où on est dans
une fonction normale:

* (1) On peut bien sûr appeler une fonction
  normale
* (2) En général ce n'est pas une bonne idée
  d'appeler une fonction asynchrone,
  c'est ce qu'on vient de voir, ça
  retourne un objet coroutine mais ça
  n'en fait rien

========== [run def foo()]

* (3) et (4) Pour les deux autres formes, où on
  essaierait de faire un await, là c'est
  simple: le langage considère ça comme
  une erreur de syntaxe, au moins comme
  ça c'est réglé; ce sont les deux seuls
  cas qui sont exclus aussi tôt dans le
  développement

========== [fragment async def afoo()]
Depuis une fonction asynchone
maintenant;

* (5) on peut bien sûr toujours appeler
  une fonction synchrone, mais la
  coroutine va bloquer en attendant le
  retour donc c'est réservé à des choses
  brèves;

* (6) on peut aussi faire un await d'une
  coroutine, ce sont les usages normaux

* (7) par contre comme tout à l'heure le
  langage nous laisse appeler sans await
  une fonction coroutine, mais pareil,
  ce n'est sans doute pas une bonne idée.

* (8) si on fait await sur une fonction
  synchrone, ce qui se passe en détail,
  c'est que la fonction est évaluée et
  son résultat doit être un awaitable.

Je vais revenir sur les 3 cas
problématiques, donc 2, 7 et 8.

========== [slide cas #2]

Le cas #2, c'est le cas où on appelle
une coroutine depuis du code synchrone,
mais sans await.

Ici j'atteins un peu les limites de la
technologie des notebooks, je vais
utiliser le ! pour lancer en fait des
commandes comme si j'étais dans un
terminal.

Ici par exemple avec cat

========== [run !cat calls2.py]

je vous montre juste le contenu du
fichier calls2.py

et si je l'exécute

========== [run !python calls2.py]

vous voyez que python est tout de même
assez malin pour me signaler le souci

========== [slide cas #7]

J'ai reproduit presque exactement le
même exemple,

========== [run !cat calls7.py]

sauf que cette fois la coroutine est
appelée - et pas awaitée - depuis du
code asynchrone

========== [slide python calls7.py]

et en fait comme vous le voyez on obtient
le même message d'avertissement.

Ce qui, bien entendu, est rassurant

bon cela dit on aimerait sans doute
pouvoir détecter ça avant de faire
tourner le code; j'ai fait quelques
recherches mais actuellement c'est à
dire en 2017, je n'ai rien trouvé dans
pylint par exemple qui puisse détecter
ça par une analyse statique, je ne doute
pas que la situation va s'améliorer dans
le futur.

========== [slide cas #8]
Je passe enfin au cas numéro 8,
c'est à dire un await sur le résultat
d'une fonction normale.

---------- fragment 2 bullets

Si vous vous souvenez la vidéo sur les
objets awaitables, ce code en toute
rigueur peut être légitime si la
fonction synchro retourne un awaitable

mais une fois qu'on a dit ça, dans un
code asynchrone - on va dire normal -
ce code-là est très très suspect.

Et dans ce cas là vous recevrez un
message à run-time vous disant que vous
ne pouvez pas faire await sur l'objet.

---------- fragment inspect.isawaitable

Dans mon cas par exemple je peux voir
avec inspect que l'objet retourné par
synchro - c'est à dire None - n'est pas
awaitable et donc que je ne peux pas
faire await dessus.

---------- slide (texte sans support)

Le premier écueil donc, c'est de se
mélanger entre les appels normaux et les
appels de coroutines avec await. Je vous
rassure, avec un tout petit peu
d'habitude, ça passe tout seul. 

En cas de besoin vous pouvez utiliser
cet utilitaire dans le module inspect,
mais les documentations sont de plus en
plus lisibles par rapport à ce point qui
est en effet très important.


========== [slide écueil #2]

Le second écueil est lorsqu'on appelle
au milieu d'une coroutine du code
bloquant qui garde la main trop
longtemps.

Quand ça se produit, les autres tâches
dans la boucle ne peuvent plus avoir la
main, et on perd totalement la
réactivité de l'ensemble.

Si on parle d'un gros calcul gourmand en
cycles, il est toujours possible
d'utiliser un vrai thread. Maintenant si
ce n'est pas opportun, je vais vous
montrer comment on peut simplement faire
respirer le code bloquant, en ajoutant
simplement des await factices.

Je vous montre une toute petite
simulation, avec à gauche un code
bien asynchrone,
========== [run def countdown]
et à droite un code qui
calcule;
========== [run def compute]
l'appel à time.sleep est
bloquant, ça me permet de simuler la
phase de calcul.

Si je fais tourner ce code, j'observe
évidemment que dès que la phase de
calcul prend la main, comme elle ne fait
aucun await, elle garde la main et
empêche le code asynchrone de réagir.

========== [slide faites respirer ...]

Dans notre cas, une façon de résoudre le
problème consiste à faire respirer le
code synchrone en injectant des await;
puisque dans notre cas on fait du
calcul, on n'a rien de pertinent à
attendre, on peut toujours attendre
`asyncio.sleep(0)` qui permet simplement
de faire respirer la coroutine à cet
endroit-là. Bien sûr il n'est pas non
plus question de faire ça toutes les
nano-secondes..

========== [run all 3 cells]

Avec ce code modifié vous pouvez voir
qu'on a réglé le problème de famine et
les deux tâches sont plus correctement
mélangées.


Ce genre de situation arrive plus
souvent qu'on ne voudrait, et pas
toujours avec du code gourmand en cycle,
mais simplement du code conçu avant
asyncio.

C'est pourquoi d'ailleurs on dit que la
programmation asynchrone est
contagieuse, dans ce sens que tous les
traitements qui ont tendance à prendre
un temps un peu trop long doivent être
revus, et transformés en coroutine.

[[AL: est-ce que cette technique n'est pas un emplâtre sur une jambe
de bois ? est-ce que c'est vraiment
utilisé en pratique ?]]
[[TP: apparemment je suis pas clair, je
veux dire si tu fais du web, les
librairies synchrones comme requests,
c'est mort, il FAUT utiliser oaihttp
sinon tu as des bouts de code bloquants
qui ne niquent toute ta boucle.  du coup
ton crawler web par exemple, bin il faut
le récrire presque entièrement, et
transformer tes anciennes fonctions
synchrones en coroutines, etc...]]

========== [slide ecueil 3]

Enfin je vous signale le cas des
exceptions non lues. Vous vous souvenez
de l'animation qu'on a vue dans une
vidéo précédente, si dans une de vos
tâches il y a une exception qui est
levée, on mémorise ça dans l'objet
tâche.

C'est une source d'erreurs car si
personne ne va lire cette information
dans la tâche, cela signifie qu'une
erreur est passée totalement inaperçue.

Je le signale surtout parce que j'ai eu
personnellement des soucis avec ça dans
des versions antérieures de python, j'ai
le sentiment que les choses se sont
améliorées en terme de confort
d'utilisation dans les versions
récentes.

Voici ce qu'on obtient sur un petit
exemple avec la 3.6.

========== [run 2 cells]
========== [interrupt]

========== slide [bonnes pratiques ...]

Ce qui m'amène à conclure cette vidéo en
vous parlant de cette page dans la
documentation, qui vous donne des
indications sur les bonnes pratiques en
cas de souci avec du code qui ne fait
pas ce qu'on veut.

J'attire notamment votre attention sur
la variable d'environnement que vous
voyez ici, qui est sans doute le moyen
le plus simple pour activer ce debug.

========== slide [résumé]

Pour résumer ce qu'on a vu dans cette
vidéo, je vous invite à retenir 4
points:

* assurez vous de bien appeler les
  coroutines avec `await`, c'est la plus
  grosse source d'erreurs surtout chez
  les débutants, mais croyez-moi, pas
  seulement :)

* il est tout à fait légal d'appeler du
  code synchrone à partir d'une
  coroutine mais gardez à l'esprit que
  ça doit faire des choses brèves, sinon
  votre boucle et donc votre application
  perd en réactivité et en performances

* c'est moins pathologique, mais
  n'oubliez pas que vos tâches peuvent
  lever des exceptions et qu'elles ne
  vous sont pas notifiées immédiatement,
  c'est à vous de les lire, ou de les
  attraper à l'intérieur de vos tâches
  si vous préférez

* enfin pensez à activer le mode debug
  en cas de soucis


À ce stade vous devriez commencer à vous
sentir raisonnablement à l'aise avec ce
paradigme un peu space, n'hésitez pas à
vous lancer et bon codage.
