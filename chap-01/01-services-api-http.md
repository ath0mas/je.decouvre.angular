#Angular, http et services

Dans cette partie nous allons voir comment faire interagir le "front" Angular avec des services Json fournis par Express.

##Rapidement, la partie serveur

Normalement (si vous avez suivi) vous devez avoir un projet qui ressemble à ça :

    votre_projet/
    ├── public/
    |   ├── bower_components/ /* angular est par ici */
    |   ├── js/      
    |   |    └── main.js
    |   └── index.html
    ├── bower.json
    └── .bowerrc

Nous avons besoin d'installer [Express](http://expressjs.com/), pour la base de données nous utiliserons [NeDB](https://github.com/louischatriot/nedb) qui est une petite base de données NoSQL légère à base de "fichiers" qui fonctionne avec la même logique que [MongoDb](http://www.mongodb.org/), vous pourrez donc porter votre code facilement par la suite.

*Pour plus d'informations, vous pouvez lire mon article: [http://k33g.github.io/2014/05/11/NEDB.html](http://k33g.github.io/2014/05/11/NEDB.html)*

###Installation Express + NeDB

A la racine de votre projet, créez un fichier `package.json` avec ce contenu:

    {
      "name": "books",
      "description" : "books",
      "version": "0.0.0",
      "dependencies": {
        "express": "4.1.x",
        "body-parser": "1.0.2",
        "nedb": "0.10.5"
      }
    }

Tapez dans une console la commande (à la racine de votre projet) :

    npm install

Maintenant vous devriez avoir cette structure :

    votre_projet/
    ├── node_modules/ /* <-- Express et NeDB sont ici */    
    ├── public/
    |   ├── bower_components/ /* angular est par ici */
    |   ├── js/      
    |   |    └── main.js
    |   └── index.html
    ├── package.json    
    ├── bower.json
    └── .bowerrc

###Services JSON

Pour le moment je n'ai prévu que 3 types de services possibles :

- Créer un livre
- Retrouver un livre par son id
- Récupérer la liste de tous les livres

Donc, créer à la racine de votre projet, un fichier `app.js` avec le contenu suivant:

```javascript
var express = require('express')
  , http = require('http')
  , bodyParser = require('body-parser')
  , DataStore = require('nedb')
  , app = express()
  , http_port = 3000
  , booksDb = new DataStore({ filename: 'booksDb.nedb' });

app.use(express.static(__dirname + '/public'));
app.use(bodyParser());

// Liste de tous les livres
app.get("/books", function(req, res) {
  booksDb.find({}, function (err, docs) {
    res.send(docs);
  });
});

// Obtenir un livre par son id
app.get("/books/:id", function(req, res) {
  booksDb.findOne({ _id: req.params.id }, function (err, doc) {
    res.send(doc)
  });
});

// Ajouter un livre
app.post("/books", function(req, res) {
  var book = req.body;

  booksDb.insert(book, function (err, newDoc) {
    res.statusCode = 301;
    res.header("location", "/books/"+newDoc._id).end();
  });
});

// Lancer l'application une fois la base chargée
booksDb.loadDatabase(function (err) {
  app.listen(http_port);
  console.log("Listening on " + http_port);
});
```

Nous avons donc ceci:

    votre_projet/
    ├── node_modules/ /* <-- Express et NeDB sont ici */    
    ├── public/
    |   ├── bower_components/ /* angular est par ici */
    |   ├── js/      
    |   |    └── main.js
    |   └── index.html
    ├── package.json    
    ├── bower.json
    ├── .bowerrc
    └── app.js

**Et nous sommes donc prêts pour la suite.**

##API `$http`

Angular propose une API "ajax" par le biais du "helper" `$http` qui permet donc de faire des requêtes vers votre serveur.

Nous avons vu dans la partie précédente comment sortir nos modèles du contrôleur avec une factory, nous avions ceci:

```javascript
/*--- factory ---*/
booksApp.factory("Models", function() {
  var books = function() {
    return [
      {title:"Backbone c'est de la balle", description:"tutorial bb", level:"très bon"}
      , {title:"React ça dépote", description:"se perfectionner avec React", level:"bon"}
      , {title:"J'apprends Angular", description:"from scratch", level:"débutant"}
    ]
  };
  var levels = function() {
    return [
      "très bon", "bon", "débutant"
    ];
  }

  return {
    books : books, levels : levels
  }
});
```

Par la suite, cela va être le serveur qui va nous fournir les livres (pour le moment je laisse les niveaux (levels) "en dur" côté front). Donc, modifions notre factory `Models` de la façon suivante:

```javascript
/*--- factory ---*/
booksApp.factory("Models", function() {

  var levels = function() {
    return [
      "très bon", "bon", "débutant"
    ];
  }

  return {
    levels : levels
  }
});
```

Et modifiez le contrôleur de cette manière:

- modifier la propriété `$scope.books = Models.books();`
- remplacer la méthode `$scope.createBook` 
- ajouter la méthode `$scope.getAllBooks`
- ajouter dans le constructeur du contrôleur un appel à `getAllBooks`
- ajouter la référence à l'api `$http` dans les paramètres du contrôleur: `booksApp.controller("MainCtrl", function($scope, $http, Models) {})`
- supprimez `$scope.selectedBook ` et `selectBook()` (nous n'en avons plus besoin pour le moment)

```javascript
var MainCtrl = booksApp.controller("MainCtrl", function($scope, $http, Models) {
  
  $scope.books = null;
  $scope.levels = Models.levels();

  $scope.getAllBooks = function() {
    $http.get("books").success(function(data) {
      $scope.books = data;
    })
  }

  $scope.createBook = function(book) {
    $http.post("books", book).success(function(data){
      $scope.getAllBooks();
      $scope.book = null;
    });
  }
  // charger les livres
  $scope.getAllBooks()

});
```

Nous utilisons l'API `$http` dans la méthode `$scope.getAllBooks` pour faire une requête de type `GET` sur l'uri `books`, ce qui aura pour effet d'appeler la route Express définie plus haut par :

```javascript
// Liste de tous les livres
app.get("/books", function(req, res) {
  booksDb.find({}, function (err, docs) {
    res.send(docs);
  });
});
```

Et nous utilisons l'API `$http` dans la méthode `$scope.createBook` pour faire une requête de type `GET` sur l'uri `books`, ce qui aura pour effet d'appeler la route Express définie plus haut par :

```javascript
// Ajouter un livre
app.post("/books", function(req, res) {
  var book = req.body;

  booksDb.insert(book, function (err, newDoc) {
    res.statusCode = 301;
    res.header("location", "/books/"+newDoc._id).end();
  });
});
```

**Notez bien** que la méthode `$scope.createBook` prend maintenant `book` en paramètre.

Avant d'essayer, nous allons apporter une petite modification à notre vue HTML, souvenez vous nous avions un bouton pour créer un livre "vierge":

```html
<hr>
<a href="#" ng-click="createBook()">Ajouter un livre</a>
<hr>
```

Modifiez comme ceci: *(nous passons le modèle `book` en paramètre de la méthode)*

```html
<hr>
<a href="#" ng-click="createBook(book)">Ajouter un livre</a>
<hr>
```

Remonter cette partie, juste en dessous du formulaire de saisie (c'est plus ergonomique) et **modifiez les directives `model` en remplaçant `selectedBook` par `book`**:

```html
<div>
  <input ng-model="book.title">
  <input ng-model="book.description">
  <div>
    <select id="inputType"
            ng-model="book.level"
            ng-options="level for level in levels"></select>
  </div>
  <!-- j'ai mis mon bouton ici, c'est plus ergonomique -->
  <hr>
  <a href="#" ng-click="createBook(book)">Ajouter un livre</a>
  <hr>
</div>
```


###Essayons

Donc, pour cela, en mode commande, à la racine, lancez la commande `node app.js` puis ouvrez dans votre navigateur [http://localhost:3000](http://localhost:3000). Cette fois-ci, vous allez obtenir une page vous permettant de saisir un nouveau livre, avec une liste vide (normal), maintenant essayez de créer des livres. 

Si tout va bien ça fonctionne (sinon "pinguez" moi).

##Toujours plus propre : on découple

Nous avons déjà vu comment séparer les responsabilités avec la factory et les `Models`, voyons maintenant comment utiliser la notion de **services** d'Angular, pour enlever la "notion" d'API `$http` du contrôleur (on découple).

###Création du service `BooksServices`

Pour créer un service, il suffit d'utiliser la méthode `service` de notre module angular `booksApp`, de la manière suivante:

```javascript
booksApp.service("BooksServices", function($http) {

  this.getAll = function(callback) {
    $http.get("books").success(function(data) {
      callback(data);
    })
  }

  this.create = function(book, callback) {
    $http.post("books", book).success(function(data){
      callback(data);
    });
  }
})
```

Le raisonnement est un peu différent de la factory qui retournait une instance d'objet/fonction, là pensez un peu comme si vous aviez une classe avec des méthodes statiques.

Nous avons donc créé `BooksServices` qui propose 2 méthodes: `getAll` et `create`, **remarquez** le passage de `$http` en paramètre: `booksApp.service("BooksServices", function($http) {})`

###Modification du contrôleur `MainCtrl`

Apportons quelques modifications à notre contrôleur:

- on ne passe plus `$http` en paramètre mais `BooksServices`: dites vous que si vous voyez `$http` dans un contrôleur c'est qu'il y a peut-être un souci ;)
- nous modifions `getAllBooks` et `createBook` pour utiliser `BooksServices`

```javascript
var MainCtrl = booksApp.controller("MainCtrl", function($scope, Models, BooksServices) {

  $scope.books = null;
  $scope.levels = Models.levels();

  $scope.getAllBooks = function() {
    BooksServices.getAll(function(data) {
      $scope.books = data;
    })
  }

  $scope.createBook = function(book) {
    BooksServices.create(book, function(data) {
      $scope.getAllBooks();
    })
  }

  $scope.getAllBooks()
});
```

Il n'y a plus qu'à tester pour vérifier ... Et bien sûr cela fonctionne `\o/`

**Nous avons donc pu voir comment rendre notre code plus propre et plus facilement maintenable.**

Nous allons ensuite voir la factory `$resource` pour être complètement "RESTful".