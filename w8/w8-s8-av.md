# w8-s8

## Résumé de la vidéo

Nous avons vu les coroutines et les
boucles d'événements, il est temps de
survoler le contenu de la librairie de base
asyncio. Ce sera un survol, j'aimerais
vous donner une idée de ce qui est
disponible, à vous ensuite de creuser en
fonction de vos besoins.

Pour commencer je rappelle son contenu,
du moins à l'heure actuelle; il y a

==========[fragment]
* la boucle d'événements, on l'a déjà
 évoquée

==========[fragment]
* des objets de base utiles à la
synchronisation

==========[fragment]
* un objet subprocess pour interagir
avec les processus

==========[fragment]
* et enfin des classes abstraites, pour
les interactions avec les protocoles
réseau et/ou les data streams.


Je ne vais bien sûr pas pouvoir tout
vous montrer, mais on va en dire
quelques mots, et voir quelques exemples
concrets.

========== [slide utilitaire]

Je me définis un petit utilitaire pour
pouvoir réexécuter chacun de mes
exemples avec une boucle vierge.

========== [slide queue]

Nous allons commencer avec l'objet Queue
Une queue c'est simplement une file de
messages, qu'on va pouvoir utiliser pour
synchroniser plusieurs traitements
par l'envoi de messages.

========== [run reset_loop]

Je peux créer un objet Queue
comme ceci. Je lui donne une taille finie,
dans mon cas un seul slot suffira.

========== [run création Queue]

Si vous lisez la documentation
vous verrez qu'on peut lui passer un
objet *loop*; de manière générale, on
l'a déjà signalé rapidement, si vous
utilisez a bon escient `set_event_loop`,
il n'est pas utile de le faire, car
get_event_loop() permet toujours de
retrouver la bonne instance.

C'est une bonne habitude à prendre de ne
pas s'encombrer à passer des objets
boucles dans tous les sens.

========== [fragment producer / consumer]

Mon exemple est très simple, il consiste
à avoir une tâche producer, qui va
écrire un message dans la queue toutes
les secondes, et une autre tâche
consumer, qui va être en attente de ces
messages et les imprimer au fur et à
mesure.

Ici j'utilise encore `sleep` dans le
producteur, pour avoir un code aussi
petit que possible, mais notre but c'est
bien sûr que le consumer soit notifié au
fur et à mesure.

On peut lire et écrire dans la queue, un
message à la fois, et dans les deux cas
c'est asynchrone - c'est à dire que get
et put sont des coroutines, comme vous
pouvez voir on les appelle avec await.

Alors pour get c'est assez évident, si
on essaie de lire et qu'il n'y a pas de
message, il faut attendre; pour put
c'est pareil si la file est pleine, il
faut attendre une place. 

========== [run producer]
========== [run consumer]

Je peux ajouter mes coroutines dans la
queue, et exécuter la boucle
indéfiniment

========== [run ensure]
========== [run loop]

Il faut que j'interrompe.

Vous voyez que c'est aussi simple que ça
peut l'être, la seule chose c'est de
bien penser à mettre les 3 instructions
await; deux du coté du producteur pour
put et sleep, et un du coté consumer.

========== [outline ]

Dans tous les cas il faut toujours bien
vérifier qu'on appelle un coroutine avec
await, sinon si on oublie le await,
comme on l'a vu déjà, le code ne fait
pas du tout ce qu'on croit.

[[AL: je n'ai pas souvenir qu'on ait vu un exemple d'await sur autre
chose qu'une coroutine]]
[[TP: non, mais on a vu qu'appeler une
coroutine sans await retourne juste
l'objet coroutine]]

C'est sans doute la source principale de
frustration chez les débutants, on en reparlera.


========== slide [limiter le
parallélisme]

Je vais vous montrer une autre
utilisation de la queue.

========== [run reset_loop]

Si vous avez
disons 10000 pages web à aller chercher,
vous voulez bien le faire en parallèle
mais votre système ne vas pas vouloir
créer 10000 connections TCP en même
temps.

Avec une queue, on peut assez facilement
limiter le nombre de tâches actives,
voyons ça. C'est très simple, on crée
une fenêtre - un objet queue dont la
taille correspond au nombre maximum de
traitements qu'on veut faire en
parallèle.

========== [run window = Queue()]

Ensuite chacune des coroutines qu'on
veut exécuter va simplement commencer
par faire un put dans la queue, pour
matérialiser une sorte de réservation;
puis faire son travail et enfin libérer
son jeton en faisant un get dans la
queue. C'est un peu contrintuitif, parce
qu'on prend un jeton avec un put et on
le rend avec un get, mais c'est parce qu'au
départ la queue est vide...

========== [slide suivant]

Je fais tourner cette idée; on avait
créé une fenêtre de 4; je vais créér une
série de 8 jobs, pour que ce soit
parlant je leur ai donné des tailles
différentes, 1, 2, 3, 1, 2 et ainsi de
suite.

Je vous laisse vous convaincre que ça
fait bien ce qu'on veut.

Je vais en rester là pour la queue; bien
sûr on peut aussi s'en servir comme
d'une queue de messages, mais bon ça
vous donne une idée de ce qu'on peut
faire avec un peu d'imagination.

========== [conclusion]

Nous verrons bien sûr d'autres éléments
de la librairie dans les compléments.

En guise de conclusion, je voudrais tout
de même ajouter un dernier mot sur la
boucle d'événements.

========== [fragment]

On l'a vu, son travail consiste à
choisir en permanence quelle tâche va
avoir la main; il est bien évident que
si elle devait à chaque instant
envisager toutes les tâches, ça lui
donnerait énormément de travail, et ce
serait terriblement inefficace.

Au contraire, la boucle utilise toutes
les ressources que lui offre l'OS, que
ce soit la gestion des interruptions
avec l'appel système signal(), ou
l'appel système select() qui lui permet
de réagir lorsqu'un canal d'entrée
sortie est prêt à être utilisé, ce qui
lui permet d'atteindre des performances
très raisonnables.

========== [fragment]

Donc on peut voir en fin de compte la
librairie asyncio comme un modèle de
programmation unifié pour gérer de
manière uniforme les accès réseau, les
processus externes, et la
synchronisation entre tâche - liste non
exclusive.

========== [slide épilogue]

Enfin, on dira un mot rapidement dans
les compléments des classes abstraites
que sont `Protocol`, `Transport` et
`Stream`, mais sans entrer dans les
détails, car tout cela n'est pas
nécessaire lorsqu'on est utilisateur des
librairies asyncio de haut niveau qui
servent dans la vie de tous les jours.

À ce stade de ce cours vous devriez
avoir les bases pour commencer à écrire
de petits programmes simples.

Je vous conseille néanmoins, avant de
passer à des choses sérieuses, d'étudier
la séquence sur les bonnes pratiques, où
nous allons voir comment éviter les
écueils les plus fréquents avec asyncio.
