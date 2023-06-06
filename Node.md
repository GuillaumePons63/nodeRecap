

# NodeJS

C'est un environnement d'exécution JavaScript côté serveur. Ce qui veut dire qu'avec [NodeJS](https://kourou.oclock.io/ressources/fiche-recap/node-js/), on peut exécuter du Javascript en dehors du navigateur, comme un certain PHP. 😎
Cet environnement fonctionne sur le principe de l'***asynchrone***. Cela signifie que l'on peut lancer plusieurs connexions en parallèle sans bloquer l'application, idéal pour un serveur web ou une application de tchat !
PHP possède *Composer* pour gérer les dépendances d'un projet, sur NodeJS ce sera *npm* (Node Package Manager).

## Express

Parmi ces packages, on retrouve [Express](https://kourou.oclock.io/ressources/fiche-recap/express/), un framework bien utile pour créer un server web. Il permet de mettre en place des routes, des middlewares ainsi que des templates avec l'appui d'une bonne documentation.

Exemple de route :
```js
const express = require('express')
const app = express()

// respond with "hello world" when a GET request is made to the homepage
app.get('/', (req, res) => {
  res.send('hello world')
})
```

## Middleware

Ce joli mot que l'on peut traduire littéralement par "vaisselle du milieu" est en fait une fonction intermédiaire qui peut être ajouté sur nos routes. *(cf schema-middleware)*
Les intérêts pour l'utiliser sont multiples :

 - Modification de l'objet de requête (request) : Un middleware peut ajouter des informations supplémentaires à l'objet de requête, par exemple, en analysant les en-têtes de requête ou en ajoutant des paramètres personnalisés. Cela permet de rendre ces informations disponibles pour les routes ultérieures.
 - Modification de l'objet de réponse (response) : Un middleware peut modifier la réponse avant qu'elle ne soit renvoyée au client. Cela peut inclure l'ajout d'en-têtes supplémentaires, la modification du corps de la réponse ou l'ajout de métadonnées.
 - Exécution de tâches communes : Les middlewares peuvent être utilisés pour effectuer des tâches courantes telles que l'authentification, la validation des données, la gestion des erreurs, la compression de réponse, le suivi des journaux, etc. Cela permet de centraliser ces fonctionnalités et de les réutiliser facilement dans différentes parties de l'application.
 - Contrôle du flux de la chaîne de traitement : Les middlewares peuvent également contrôler le flux de la chaîne de traitement en décidant s'il faut passer à l'étape suivante ou arrêter le traitement en envoyant une réponse au client. Cela peut être utile, par exemple, pour effectuer une vérification d'autorisation avant de traiter une requête.

Et comment ça s'utilise sur Express ?

```js
// On déclare un middleware qui fait des trucs
let myLogger = function (req, res, next) {
  console.log('LOGGED');
  // On passe à la route suivante
  next();
};

// On l'utilise avant d'aller sur une méthode d'un controller
app.get('/', myLogger, appController.home);
``` 
Plein d'exemples de middlewares se cachent dans le projet *react-express.ACL*.

## Sequelize

Pour interagir avec une BDD, on peut tout faire à la main, ce qui est une hérésie pour tout dev qui se respecte.
Où l'on peut utiliser [Sequelize](https://sequelize.org/docs/v6/getting-started/) quand on dev sur NodeJS 🔥
C'est un ORM qui permet de faire des choses assez sympathiques :

 - Modéliser les données en définissant les tables/colonnes/contraintes via des fichiers, qu'on place en général dans un dossier `Models`
 - Requêter les données avec un CRUD déjà tout fait pour vos modèles
 - Valider les données en définissant des contraintes
 - Gérer les relations entre les tables (One-to-One, Many-to-Many, One-to-Many)
 - Gérer les migrations de BDD

Tout ce beau monde est compatible à partir du moment où votre BDD est sur MySQL, PostgreSQL, SQLite ou Microsoft SQL Server. On est zen 😇

Exemple de modèle pour une table User :
```js
const { Model, DataTypes, literal } = require('sequelize');
const connexion =  new Sequelize(
    process.env.DB_NAME,
    process.env.DB_USERNAME,
    process.env.DB_PASSWD,
    {
        define: {
            createdAt: 'created_at',
            updatedAt: 'updated_at',
        },
        host: process.env.DB_HOST,
        dialect: process.env.DB_ENV,
        logging: false,
    }
);

class User extends Model {}

User.init(
    {
        id: {
            type: DataTypes.INTEGER,
            unique: true,
            autoIncrement: true,
            primaryKey: true,
        },
        email: {
            type: DataTypes.TEXT,
            unique: true,
            allowNull: false,
        },
        firstname: {
            type: DataTypes.STRING,
        },
        lastname: {
            type: DataTypes.STRING,
        },
        role_id: {
            type: DataTypes.INTEGER,
        },
        created_at: {
            type: DataTypes.DATE,
            defaultValue: literal('CURRENT_TIMESTAMP'),
            allowNull: false,
        },
        updated_at: {
            type: DataTypes.DATE,
            allowNull: true,
        },
    },
    {
        // instance de la connexion obligatoire
        sequelize: connexion,
        modelName: 'User',
        tableName: 'users',
    }
);

module.exports = User;
```

Exemple de relation entre les tables :
```js
const User = require('./User');
const Role = require('./Role');

// Définition d'associations entre modèles
Role.hasMany(User, {
    foreignKey: 'role_id',
    as: 'users',
});

User.belongsTo(Role, {
    foreignKey: 'role_id',
    as: 'role',
});

module.exports = { User, Role };
```

Une fois que ces fichiers sont écrits, c'est parti pour écrire vos requêtes dans les controllers à base de find, de query, d'update et autres joyeusetés ! 🤩

## PostgreSQL

A l'instar de MySQL, c'est un système de gestion de base de données relationnelles (SGDBR).
Il possède plein d'avantages :

 - Mise en place de contrainte de clé étrangère
 - Conforme aux normes SQL, pour ne pas être dépaysé de MySQL
 - Extensible car on peut créer ses propres types de données, ses fonctions, ses opérateurs.. on peut presque tout personnaliser
 - Performant et fonctionnel même avec des charges de travail à grande échelle
 - Beaucoup de développeurs l'utilisent !

Voici quelques fiches récap sur ce SGDBR :

 - [PostgreSQL](https://kourou.oclock.io/ressources/fiche-recap/postgresql/)
 - [Se connecter à une BDD PostgreSQL depuis Node (module pg)](https://kourou.oclock.io/ressources/fiche-recap/se-connecter-a-une-bdd-postgresql-depuis-node-module-pg/)
 - [Rôles PostgreSQL](https://kourou.oclock.io/ressources/fiche-recap/roles-postgresql/)

## Tests unitaires

A l'aube du développement, les stagiaires étaient désignés pour vérifier si l'évolution qui vient d'être mise en place ne cassait rien sur le site.
Puis, un stagiaire très paresseux (donc très bon) s'est dit que ce serait pas mal d'automatiser la vérification d'un programme. Ainsi naquit les test unitaires 💥
> Les tests unitaires sont des fonctions qui vérifient si d'autres
> fonctions fonctionnent.

Ces petits tests permettent de s'assurer que chaque fonctionnalité répond correctement au fur et à mesure que le développement avance. Aussi, il permet de détecter bien plus précisément les bugs et donc de les corriger plus vite.

### Supertest

[Supertest](https://www.npmjs.com/package/supertest) est une librairie permettant de tester directement des routes HTTP, sans avoir besoin de cliquer sur un navigateur !
```js
const request = require('supertest');
const express = require('express');

const app = express();

app.get('/user', function(req, res) {
  res.status(200).json({ name: 'john' });
});

// Ce qu'on est censé obtenir en allant sur cette route
request(app)
  .get('/user')
  .expect('Content-Type', /json/)
  .expect('Content-Length', '15')
  .expect(200)
  .end(function(err, res) {
    // Y a eu un soucis
    if (err) throw err;
  });
```

### JEST
[Jest](https://jestjs.io/) est quant à lui un framework dédié aux tests.
Il permet de créer des tests pour n'importe quel code Javascript, que ce soit des fonctions, des modules ou même des composants React.
Aussi, il permet d'automatiser des tests, ce qui permet de faire de l'intégration continue (**CI**) et aussi de la livraison continue (**CD**). Ce qui veut dire que [les tests sont lancés lors d'un git push](https://medium.com/@trevorjperez1/add-jest-to-your-ci-cd-pipeline-with-github-actions-b369c0079173), royal 🤩

### TDD
Le test driven development est une méthode consistant à d'abord écrire tous ses tests avant même de commencer à coder !
Une fois fait, il ne reste plus qu'à lancer les tests pour voir les erreurs et les rectifier une à une.
Cette pratique possède de nombreux avantages :

 - Code fiable dès le départ
 - Encourage à développer proprement
 - Limite les bugs
 - Phase de développement beaucoup plus rapide
 - Favorise la bonne collaboration entre les développeurs

Evidemment tout n'est pas rose, le fait d'écrire tous les tests est coûteux en temps, mais suivant les cas ça peut être plus rapide de suivre cette méthode 😇