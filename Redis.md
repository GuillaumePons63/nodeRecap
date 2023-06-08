

# Redis

Un véritable [couteau suisse](https://redis.io/docs/getting-started/) qui permet de gérer :

 - Une base de données
 - De la mise en cache
 - Un système de messagerie pub/sub
 - Une file d'attente
 - [Et bien plus encore](https://kourou.oclock.io/ressources/fiche-recap/presentation-de-redis/)

## Base de données
Utiliser la BDD de Redis n'est pas pour utiliser une BDD physique comme avec PosgreSQL ou MySQL.
Ici, on n'est plus dans un système de table avec des relations entre elles, on fait tout avec un système clé/valeur qui permet d'être beaucoup plus rapide. Les données ne sont plus stockées de manière physique sur le disque, mais en mémoire. Avec Redis, on ne requête plus les données avec du SQL, on fait du NoSQL (comme avec MongoDB) 🚀
Exemple :
```js
const redis = require('redis');
const client = redis.createClient();

function addTask(userId, task) {
  // stocke la tâche dans un hachage
  client.hset(`tasks:${userId}`, task.id, JSON.stringify(task), (err, reply) => {
    if (err) {
      console.error(err);
    } else {
      console.log('Tâche ajoutée avec succès');
    }
  });
}

const task1 = {
  id: '1',
  title: 'Faire les courses',
  description: 'Acheter des fruits, du pain, du lait',
  completed: false
};

addTask('user1', task1);
```
[Pour plus d'exemples](https://redis.io/docs/clients/nodejs/)

## Pub/Sub
Et parmi ses nombreuses facettes nous avons découvert le paradigme publisher/subscriber, qui n'est pas propre à Redis 😉
Son fonctionnement repose sur 3 acteurs principaux (cf schema-pub-sub) :

 - Le publisher -> celui qui va transmettre les données
 - Le subscriber -> celui qui va recevoir les données
 - Le channel -> le canal de communication par lequel transite les données

Ainsi, en se connectant sur le bon canal, plusieurs subscribers peuvent recevoir les données transmises par un publisher. Attention cependant, si le subscriber n'est pas présent dans le canal au moment de l'envoi de la donnée, celle-ci sera perdue !

Et comment on met en place tout ce beau monde ? 😋

Déjà, s'assurer que Redis est installé sur le server et pour l'application :
`npm install redis`

Voici comment créer un publisher et un suscriber avec Redis de manière basique :
```js
const { createClient } = require("redis");
const client= createClient();
client.on("error", (err) => console.log("Redis Client Error", err));
client.connect().then(() => console.log("Redis Client connected"));

// On clone le client :
const publisher = client;
const subscriber = await client.duplicate();
await subscriber.connect();

await subscriber.subscribe("message", (message) => {
    console.log("Message received :", message); // Should log the message sent
});
    
await publisher.publish(
    "message",
    "This is the best message I've ever sent !"
);
```

## Server Sent Events

En l'état ça fonctionne mais ce n'est pas très utile.
Pour que ce soit sympa, on pourrait imaginer faire un server capable d'envoyer des notifications en temps réel et de les intercepter côté client.
Ca tombe bien, c'est tout le concept de l'atelier "jobs.atelier" et plus particulièrement des Servers Sent Events 😇

Dans cet atelier, on a vu qu'il était possible de créer des routes sur Express qui s'exécutent à l'infini grâce à ces headers de réponse. Pratique pour mettre en place un canal de communication en temps réel :
```js
 const headers = {
    'Content-Type': 'text/event-stream',
    Connection: 'keep-alive',
    'Cache-Control': 'no-cache',
};

res.writeHead(200, headers);
```

Puis qu'on était capable via JS d'écouter une route en continu grâce à ce petit bout de code :
```js
events = new EventSource(
    `http://localhost:5000/dashboardNotifications`
);
// Le publisher a envoyé quelque chose !
events.onmessage = event => {
    const parsedData = JSON.parse(event.data);
};
```

Dans le cas de l'atelier, cela permet de trigger un événement côté front, du type "un job a été liké".
Une route côté server sera donc appelée, et s'occupera de mettre en place un publisher avec une donnée à transmettre.
La route dashboardNotifications contiendra un subscriber, qui recevra la donnée du publisher.
Le client pourra récupérer cette donnée grâce à son `events.onmessage`, sans jamais devoir recharger la page.
La boucle est bouclée 😎
