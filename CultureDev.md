
# Culture dev

## Logs
Pour faciliter la vie des développeurs, les logs (enregistrements) sont cruciaux.
Ils permettent de traquer automatiquement le comportement d'un projet et de le noter dans des fichiers, des BDD ou même des mails.
Cela permet d'être au courant de la moindre erreur, surtout celles critiques et de rapidement trouver où se situe le bug.

### Try/catch

Que ce soit en JS ou en PHP, il est possible de mettre en place des garde-fous pour empêcher l'application de tomber dans le cas où il y aurait un bug *(fait rarissime je le conçois)*.
L'une de ces méthode est le **try/catch**, qui ressemble à ça en JS :
```js
try {
    // Exemple de traitement de donnée
    let j = await Job.findOne({
        where: { url: req.body.url },
    });
    if (!j) {
         j = await Job.create(req.body);
    }
    res.json({ message: 'ok' });
} catch (e) {
	// Une erreur est survenue, on revient en arrière !
	// On log, on panique pas
    await t.rollback();
    next(e);
}
```

### Throw

Une autre manière de faire est d'avertir l'utilisateur que ce qu'il a demandé n'est pas réalisable via les **Throw**. L'idée est de stopper le programme et transmettre un message d'erreur lorsqu'une route est appelée via de mauvais paramètres, par exemple sur Express :
```js
app.get('/', (req, res) => {
  throw new Error('BROKEN') // Express will catch this on its own.
})
```

### Trigger BDD

Du côté de la base de données aussi il est possible de logger des choses à des moments précis.
On peut notamment faire ça grâce aux [triggers](https://www.enterprisedb.com/postgres-tutorials/everything-you-need-know-about-postgresql-triggers) 😎
Cette fonctionnalité permet d'exécuter du code à partir du moment où un évènement s'est déroulé dans la BDD, comme lors d'un insert, un update ou un delete.
On peut très bien imaginer en SQL de créer un trigger au moment de l'ajout d'un utilisateur et de lancer une fonction SQL à ce moment pour écrire une entrée dans une table "logs", par exemple.
Un exemple de trigger est disponible dans le challenge *atelier-j12*

> Tous ces éléments font partis des bonnes pratiques à adopter pour
> sécuriser son application et éviter de se prendre la tête lors du
> debug 😇

## Le Design Pattern Observer
Ce patron de conception met en jeu deux éléments *(cf schema-observer)* :

 - Le sujet (observable) -> Celui dont on regarde l'état et qui nous notifie à chaque modification en faisant un coucou aux observateurs
 - L'observateur -> Celui qui va recevoir les notifications du sujet et implémenter les méthodes de mise à jour pour le sujet.

> C'est un peu comme un enfant qui demande la permission à ses parents d'avoir une glace. Les parents vérifient si le dit enfant a faim et lui donne la glace, ce n'est pas à l'enfant de se servir tout seul non mais !

Ce pattern permet de séparer les concepts, en mettant le code métier dans les observateurs qui vont réagir à un état du sujet. Le sujet peut avoir autant d'observateurs que nécessaires, cela permet de rester flexible et évolutif.

Un exemple d'observer a été fait dans le challenge *nodejs.observer-pattern-streaming* 😇

## Le Design Pattern Factory
Ce patron de conception permet de créer des objets sans spécifier leur classe. 💥
Voici son fonctionnement *(cf schema-factory)* :

 - Il y a l'interface de fabrique (Factory Interface) qui déclare les méthodes pour créer des objets.
 - Il y a la fabrique concrète (Concrete Factory) qui implémente l'interface et créé réellement les objets.
 - Et le produit, l'objet qui a été créé par la fabrique

Le fait de passer par ces interfaces permet de séparer d'une autre manière les concepts.
Le code client se concentre sur l'utilisation des objets plutôt que la création qui est gérée par la factory.
Cela donne l'avantage de modifier la création des objets sans jamais impacter le code client (et ça c'est chouette).

[Pour plus d'info](https://medium.com/geekculture/node-js-and-factory-pattern-ddabcfe6541c) et aussi le challenge *nodejs.n-tiers-challenge-j11* qui associe à la fois les observers et les factories.