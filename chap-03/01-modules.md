#Organisation de projet (utilisation du pattern module)

Bon, notre projet devient vite un peu "pénible" à maintenir d'un point de vue "javascript", tout est dans `main.js`.
Nous allons donc "splitter" ça en plusieurs fichiers. Nous n'allons pas utiliser **Require.js** (soit dit en pensant, vu les possibilité de modularisation d'Angular, je ne suis pas convaincu du tout d'avoir besoin de Require).
Alors, la structure que je vais vous proposer n'est pas parfaite, il y a d'autre moyens de faire, à vous de décliner en fonction de vos règles projets, sensibilités, habitudes, etc. ...

Une piste serait d'organiser d'une manière plus "fonctionnelle", mais c'est un autre sujet (que j'aborderais certainement).

##Préparation

Apportons quelques modifications à l'arborescence de notre projet:

Dans le répertoire `js`, nous allons créer un répertoire `controllers` avec 4 fichiers javascript:

    js/
     └── controllers/
         ├── ActualitesCtrl.js
         ├── AdditionCtrl.js
         ├── CoupsDeCoeurCtrl.js
         └── MainCtrl.js

Ensuite, toujours dans le répertoire `js`, nous allons créer un répertoire `factories` avec 2 fichiers javascript:

       js/
        └── factories/
            ├── Book.js
            └── Models.js

Nous devrions donc avoir la structure suivante:

    votre_projet/
    ├── public/
    |   ├── bower_components/ /* angular est par ici */
    |   ├── js/
    |   |    ├── controllers/
    |   |    |   ├── ActualitesCtrl.js
    |   |    |   ├── AdditionCtrl.js
    |   |    |   ├── CoupsDeCoeurCtrl.js
    |   |    |   └── MainCtrl.js
    |   |    ├── factories/
    |   |    |   ├── Book.js
    |   |    |   └── Models.js
    |   |    └── main.js
    |   ├── books/
    |   |    ├── bookForm.html
    |   |    └── booksList.html
    |   ├── templates/
    |   |    ├── actualites.html
    |   |    ├── addition.html
    |   |    └── coupsdecoeur.html
    |   └── index.html
    ├── bower.json
    └── .bowerrc

##Refactoring, c'est parti!

Maintenant nous allons passer dans chacun des nouveaux fichiers.

###`Models.js`

Nous allons extraire le code concernant les modèles dans `main.js` pour le "transférer" dans `Model.js`.

**Avant (dans `main.js`)**:

```javascript
booksApp.factory("Models", function() {
  var levels = function() {
    return [
      "", "très bon", "bon", "débutant"
    ];
  }

  return {
    levels : levels
  }
});

```

**Après (dans `/factories/Models.js`)**:

La modification n'est pas énorme.

- Il faut bien penser à encapsuler le code dans `(function () { //foo }());` (module "pattern", allez lire ceci [http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)).
- Et déclarer notre `Models` dans le module principal : `angular.module("booksApp").factory("Models", Models);`

```javascript
(function () {

  var Models = function () {
    var levels = function() {
      return [
        "", "très bon", "bon", "débutant"
      ];
    }

    return {
      levels : levels
    }
  };

  //Models.$inject = []; pas d'injection

  angular.module("booksApp").factory("Models", Models);

}());
```


###`Book.js`

Nous allons extraire le code dans `main.js` pour le "transférer" dans `Book.js`, sur le même principe que précédemment:

**Avant (dans `main.js`)**:

```javascript
booksApp.factory("Book", function($resource) {
  return $resource("/books/:id", {id: '@id'},{
    update: { method: 'PUT' , params: {id: '@id'}, isArray: false }
  });
})
```

**Après (dans `/factories/Book.js`)**:

- Il faut bien penser à encapsuler le code dans `(function () { //foo }());`
- Attention, il faut passer la dépendance : `$resource`, alors `Book.$inject = ["$resource"]` n'est pas forcément obligatoire, mais si vous minifiez votre code, ça peut vous rendre service
- Et déclarer notre factory `Book` dans le module principal : `angular.module("booksApp").factory("Book", Book);`

```javascript
(function () {

  var Book = function ($resource) {
    return $resource("/books/:id", {id: '@id'},{
      update: { method: 'PUT' , params: {id: '@id'}, isArray: false }
    });
  };

  Book.$inject = ["$resource"];

  angular.module("booksApp").factory("Book", Book);

}());
```

Nous allons utiliser le même principe pour les contrôleurs :

###`ActualitesCtrl.js`

**Après (dans `/controllers/ActualitesCtrl.js`)**:

```javascript
(function () {

  var ActualitesCtrl = function ($scope) {
    $scope.title = "Actualités";
    $scope.news = [
      { title:"Un nouveau Stephen King à venir?" }
      , { title:"Le Kindle paperwhite est juste une turie" }
      , { title:"Le format epub 3 en détail" }
    ];
  };

  ActualitesCtrl.$inject = ["$scope"];

  angular.module("booksApp").controller("ActualitesCtrl", ActualitesCtrl);

}());
```

###`CoupsDeCoeurCtrl.js`

**Après (dans `/factories/CoupsDeCoeurCtrl.js`)**:

```javascript
(function () {
  var CoupsDeCoeurCtrl = function ($scope) {
    $scope.title = "Coups de Coeur";
    $scope.items = [
      { title:"Game of Thrones" }
      , { title:"Bilbo le Hobbit" }
      , { title:"Le sorcier de TerreMère" }
    ];
  };

  CoupsDeCoeurCtrl.$inject = ["$scope"];

  angular.module("booksApp").controller("CoupsDeCoeurCtrl", CoupsDeCoeurCtrl);
}());

```

###`AdditionCtrl.js`

**Après (dans `/factories/AdditionCtrl.js`)**:

```javascript
(function () {

  var AdditionCtrl = function ($scope, $routeParams) {
    $scope.result = parseInt($routeParams['a']) + parseInt($routeParams['b']);
  };

  AdditionCtrl.$inject = ["$scope", "$routeParams"];

  angular.module("booksApp").controller("AdditionCtrl", AdditionCtrl);

}());
```

Et enfin :

###`MainCtrl.js`

**Après (dans `/factories/MainCtrl.js`)**:

```javascript
(function () {

  var MainCtrl = function ($scope, Models, Book) {

    $scope.whenFormIsLoaded = function() { console.log("Form is loaded ..."); }
    $scope.whenListIsLoaded = function() { console.log("List is loaded ..."); }

    $scope.books = null;
    $scope.levels = Models.levels();

    $scope.selectBook = function(book) {
      $scope.book = book;
    }

    $scope.getAllBooks = function() {
      Book.query().$promise.then(
        function(data) {
          $scope.books = data;
        },
        function(error) {
          console.log("getAllBooks", error);
        }
      )
    }

    $scope.saveBook = function(book) {
      book._id !== undefined ? $scope.updateBook(book) : $scope.createBook(book);
    }

    $scope.createBook = function(book) {
      Book.save(book).$promise.then(
        function(data) {
          $scope.getAllBooks();
          $scope.newBook();
        },
        function(error) {
          console.log("createBook", error);
        }
      )
    }

    $scope.newBook = function() {
      $scope.book = {};
    }

    $scope.updateBook = function(book) {
      Book.update({id: book._id}, book).$promise.then(
        function(data) {
          $scope.newBook();
          $scope.getAllBooks();
        },
        function(error) {
          console.log("updateBook", error);
        }
      )
    }

    $scope.deleteBook = function(id) {
      Book.delete({id: id}).$promise.then(
        function(data) {
          $scope.getAllBooks();
          $scope.newBook();
        },
        function(error) {
          console.log("book delete ERROR", error);
        }
      )
    }


    $scope.getBook = function(id) {
      Book.get({id:id}).$promise.then(
        function(data) {
          console.log(data);
          $scope.book = data;
        },
        function(error) {
          console.log("getBook", error);
        }
      )
    }

    $scope.getAllBooks()
  };

  MainCtrl.$inject = ["$scope", "Models", "Book"];

  angular.module("booksApp").controller("MainCtrl", MainCtrl);

}());
```

##Encore quelques modifications

###`main.js`

Là aussi utilisons le module "pattern". Maintenant, votre code doit ressembler à ceci:

```javascript
(function () {
  var booksApp = angular.module("booksApp", ["ngResource", "ngRoute"]);

  booksApp.config(['$routeProvider',
    function($routeProvider) {

      $routeProvider.
        when('/actualites', {
          templateUrl: "templates/actualites.html",
          controller: "ActualitesCtrl"
        })
        .when('/coupsdecoeur', {
          templateUrl: "templates/coupsdecoeur.html",
          controller: "CoupsDeCoeurCtrl"
        })
        .when('/addition/:a/:b', {
          templateUrl: "templates/addition.html",
          controller: "AdditionCtrl"
        })
        .otherwise({
          redirectTo: "/"
        })

    }
  ]);

  booksApp.filter("stars", function() {
    return function(data) {
      switch (data) {
        case "très bon":
          return "***";
          break;
        case "bon":
          return "**";
          break;
        case "débutant":
          return "*";
          break;
        default:
          return "";
      }
    }
  });

})();
```

Bien sûr, rien ne vous empêche d'externaliser `filter` et `config`

Et finalement:

###`index.html`

Il ne faut bien sûr pas oublier de référencer tous nos scripts: *(il existe des solutions simples pour faciliter la gestion de ces références mais c'est un autre sujet)*.

```html
<script src="js/main.js"></script>

<script src="js/factories/Models.js"></script>
<script src="js/factories/Book.js"></script>

<script src="js/controllers/AdditionCtrl.js"></script>
<script src="js/controllers/ActualitesCtrl.js"></script>
<script src="js/controllers/CoupsDeCoeurCtrl.js"></script>

<script src="js/controllers/MainCtrl.js"></script>
```

ET hop, c'est fini! Vous pouvez tout relancer, cela fonctionne comme avant et votre projet est plus facile à maintenir.
