
# Authentification

C'est cool de pouvoir gérer des utilisateurs sur son site, mais il ne faut pas laisser n'importe qui faire n'importe quoi, [il y a un pro pour ça](https://www.youtube.com/@nqtv/videos).

## JWT (JSON Web Token)

L'une des méthodes pour authentifier correctement un utilisateur sur une application web est d'utiliser [JWT](https://kourou.oclock.io/ressources/fiche-recap/le-cas-jwt/).
Un JWT est composé de trois parties distinctes : l'en-tête (header), la charge utile (payload) et la signature. L'en-tête spécifie le type de token et l'algorithme de hachage utilisé pour la signature. La charge utile contient les informations supplémentaires, telles que l'identité de l'utilisateur et les autorisations accordées. La signature permet de vérifier l'intégrité du token.
C'est pour ça que le token ressemble à quelque chose du genre : `xxxxx.yyyyy.zzzzz`
En bref, c'est un jeton magique qui contient tout ce qu'on veut pour savoir si un utilisateur est des nôtres. 👀

Lorsqu'un utilisateur veut accéder à une route protégée, il doit transmettre son token dans le header de la requête.
Ainsi, le server peut récupérer le token et vérifier si les informations correspondent, comme ceci :
```js
const jwt = require('jsonwebtoken');

function auth(req, res, next) {
    const token = req.header('x-auth-token');
    if (!token) {
        const error = new Error('Encore une catastrophe');
        error.status = 401;
        return next(error);
    }

    try {
	    // On vérifie si la signature du token correspond avec celle du server
        const decoded = jwt.verify(token, process.env.APP_SECRET);
        req.user = decoded.user;
        next();
    } catch (e) {
        next(e);
    }
}
```

## ACL

Bon c'est chouette on peut dire qu'un utilisateur est bien de chez nous, mais c'est pas forcément pour ça que je veux qu'il accède à ma super page en l'honneur de [Christopher Walken](https://www.youtube.com/watch?v=wCDIYvFmgW8) 😱
Pour n'autoriser que l'élite des utilisateurs à accéder au graal, les développeur ont mis au point le concept des **A**cces **C**ontrol **L**ist.
Pour faire simple, c'est un système qui permet de définir des règles spécifiques pour chaque utilisateur ou groupe d'utilisateurs, afin de déterminer ce à quoi ils ont le droit d'accéder et quelles actions ils peuvent effectuer sur ces ressources.
Ces ACL peuvent être mis en place sur des systèmes d'exploitation, dans des pares-feux ou bien sur une application web qui utilise NodeJS. 🥳

```js
// route
router.get(
    '/awesome-walken',
    [
        auth,
        authorize(['elite']),
    ]
);

// middleware
function authorize(actions) {
    return async (req, res, next) => {
        const { id } = req.user;

        const user = await User.findByPk(id, {
            include: { all: true, nested: true },
        });

        for (const action of actions) {
            if (user.can(action)) {
                return next();
            }
        }

        const error = new Error("Un truc bizarre s'est produit");
        error.status = 403;
        return next(error);
    };
}

// Méthode can du model User :
can(action) {
    let ok = false;

    this['roles'].forEach(role => {
        role['permissions'].forEach(ability => {
            if (ability.name === action) {
                ok = true;
            }
        });
    });

    return ok;
}
```

Dans ce petit exemple, on ne va autoriser l'accès à la route qu'aux utilisateurs qui ont la permission "elite".
C'est un peu comme un badge, si tu l'as pas, tu rentres pas.

> [Vous avez le formulaire bleu ?](https://www.youtube.com/watch?v=FdfIuw9o7Ys)
