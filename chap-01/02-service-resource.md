#Angular, "Effleurer les ressources"

Les ressources (`$resource`) sont un module très puissant d'Angular. Nous allons aujourd'hui juste effleurer leurs possibilités, mais ça sera largement suffisant pour faire pas mal de chose en termes de "discussions" avec notre serveur http. On peut considérer `$resource` comme un helper "autour" de `$http` pour faciliter des appels à des API "RESTful".

A la fin de ce paragraphe nous aurons un service capable de faire les requêtes suivantes:

- GET : [/books](/books)
- GET : [/books/{id}](/books/{id})
- DELETE : [/books/{id}](/books/{id})
- POST : [/books](/books)
- PUT : [/books/{id}](/books/{id})

##Pré-requis

Nous avons besoin du module `$resource` présent dans `angular-resource`. Veuillez donc ajouter dans votre fichier `bower.json` la ligne `"angular-resource" : "1.2.16"`, nous avons donc normalement :

    {
      "name": "books",
      "version": "0.0.0",
      "dependencies": {
        "angular" : "1.2.16",
        "angular-resource" : "1.2.16"
      }
    }

Faite ensuite en mode commande, dans le répertoire du projet un petit:

    bower update

Et, allez ajouter dans `index.html` (juste après le tag `<script>` d'Angular) ceci:

```html
<script src="bower_components/angular-resource/angular-resource.min.js"></script>
```

Nous sommes bon pour commencer.

##Modification de `BooksServices` : `query` et `save`

Nous allons complètement modifier notre service `BooksServices`, vous pouvez tout supprimer à l'intérieur, comme ceci, tout en injectant en paramètre `$resource` :

```javascript
booksApp.service("BooksServices", function($resource) {
  
});
```

Ensuite nous allons définir une ressource au sens Angular :

```javascript
booksApp.service("BooksServices", function($resource) {
  var books = $resource("/books/:id", {id: '@id'},{});  
});
```

La simple définition de cette ressource va nous permettre de re-écrire simplement nos méthodes `getAll` et `create` :

```javascript
booksApp.service("BooksServices", function($resource) {
  var books = $resource("/books/:id", {id: '@id'},{});  

  this.getAll = function(callback) {
    books.query().$promise.then(function(data) {
      callback(data);
    }, function(error) {});
  }

  this.create = function(book, callback) {
    books.save(book).$promise.then(function(data){
      callback(data);
    }, function(error) {});
  }

});
```

**Remarque**: il est aussi possible d'utiliser cette notation:

```javascript
booksApp.service("BooksServices", function($resource) {
  var books = $resource("/books/:id", {id: '@id'},{});  

  this.getAll = function(callback) {
    books.query().success(function(data) {
      callback(data);
    });
  }

  this.create = function(book, callback) {
    books.save(book).success(function(data){
      callback(data);
    })
  }

});
```

Vous pouvez tester à nouveau votre code, vous verrez qu'il fonctionne comme avant.

##Création des services `update`, `delete`, `getOneById`

Si on se limite au paragraphe précédent, on pourrait penser que `$resource` n'a que peu d'intérêt. Ce serait une erreur, voyons donc ce que cela peut nous apporter sans trop d'effort. *Je vous engage à lire la documentation et les tutoriaux sur le sujet si vous souhaitez aller plus loin, mais les bases permettent de faire pas mal de choses.*

###Ajout des services `getOneById` et `deleteOneById`

Dans notre `BooksServices` ajoutons déjà 2 méthodes:

- `getOneById` : permet de retrouver un modèle par son id
- `deleteOneById` : permet de supprimer un modèle par son id

```javascript
  this.getOneById = function(id, callback) {
    books.get({id:id}).$promise.then(function(data){
      callback(data);
    }, function(error) {});
  }

  this.deleteOneById = function(id, callback) {
    books.delete({id:id}).$promise.then(function(data){
      callback(data);
    }, function(error) {});
  }
```

###Modification du contrôleur

Ajoutons 2 méthodes à notre contrôleur:

- `getBook`
- `deleteBook`

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

  $scope.getBook = function(id) {
    BooksServices.getOneById(id, function(data) {
      //TODO
      console.log("get one book", data)
    })
  }

  $scope.deleteBook = function(id) {
    BooksServices.deleteOneById(id, function(data) {
      //TODO
      console.log("delete one book", data)
    })
  }

  $scope.getAllBooks()
});
```


###Services Express correspondant à ajouter

Allons ajouter le code des services JSON dans l'application Express en ajoutant ceci dans le fichier `app.js` :

```javascript
// Obtenir un livre par son id
app.get("/books/:id", function(req, res) {
  booksDb.findOne({ _id: req.params.id }, function (err, doc) {
    res.send(doc)
  });
});

// Supprimer un livre par son id
app.delete("/books/:id", function(req, res) {
  booksDb.remove({ _id: req.params.id }, {}, function (err, numRemoved) {
    res.send(numRemoved);
  });
});

```

Vu que nous n'avons pas modifié l'IHM pour tenir compte de ces services, vous pouvez juste ajouter **provisoirement** ceci à la fin de votre contrôleur Angular:

```javascript
window.getBook = $scope.getBook;
window.deleteBook = $scope.deleteBook
```

Cela vous permettra "d'essayer" vos services dans la console de votre navigateur en passant des commandes de ce type:

    getBook({ title: 'Hello World!', description: 'le chemin du programmeur', _id: '0HC0iIQLD3HJ1YOL' })

ou bien:

    deleteBook({ _id: '0HC0iIQLD3HJ1YOL' })

###Il en manque un! `update`

Souvent les API Web (serveur) attendent une requête de type PUT avec un format comme celui-ci: [URL/object/ID](URL/object/ID). Avec Angular, pour ce cas-là, il faut être plus explicite (si quelqu'un a une autre solution je suis preneur) et définir explicitement une méthode `update` dans notre ressource, de la manière suivante:

Nous modifions la ressource dans `BooksServices`:

```javascript
var books = $resource("/books/:id", {id: '@id'},{
  update: { method: 'PUT' }
});
```

ce qui nous permet de définir le service suivant dans `BooksServices`:

```javascript
this.update = function(book, callback) {
  books.update({id:book._id}, book).$promise.then(function(data){
    callback(data);
  }, function(error) {});
}
```

###Modification du contrôleur

Ajoutons 1 méthode à notre contrôleur:

- `updateBook`

```javascript
$scope.updateBook = function(book) {
  BooksServices.update(book, function(data) {
    $scope.getAllBooks();
  })
}
```

###Services Express correspondant à ajouter

Allons ajouter le code du services JSON de mise à jour de modèle dans l'application Express en ajoutant ceci dans le fichier `app.js` :

```javascript
// Modifier un livre
app.put("/books/:id", function(req, res) {
  booksDb.update({_id:req.params.id}, req.body, {}, function (err, numReplaced) {
    res.send(numReplaced);
  })
});
```

Là aussi que nous n'avons pas modifié l'IHM pour tenir compte de ce service, vous pouvez juste ajouter **provisoirement** ceci à la fin de votre contrôleur Angular:

```javascript
window.updateBook = $scope.updateBook
```

Cela vous permettra "d'essayer" le service de mise à jour dans la console de votre navigateur en passant cette commande:

    updateBook({ title: 'Hello World!', description: 'le chemin du programmeur', level: 'bon' _id: '0HC0iIQLD3HJ1YOL' })

C'est tout pour cette partie, c'est relativement simple. Dans une prochaine partie, nous ferons un peu de "cosmétique" pour rendre notre application plus agréable à utiliser avant de passer à des sujet "plus techniques"