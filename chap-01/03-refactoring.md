#Pourquoi faire compliqué?

Ce matin en voulant continuer mon "périple Angular", et en repensant au commentaire fait par **Eric Taix**, cf remarque dans la partie précédente "[Service et $resource](chap-01/02-service-resource.md)", je me suis aperçu que mon code n'était pas forcément élégant et que l'on pouvait faire plus simple et plus propre (c'est mon avis, c'est discutable et j'accepte d'en discuter).

##Une factory à la place de mon service

Nous allons remplacer notre `BooksServices` par une factory que nous appellerons `Book`. Donc supprimer complètement le code de `BooksServices` et remplacez le par:

```javascript
booksApp.factory("Book", function($resource) {
  return $resource("/books/:id", {id: '@id'},{
    update: { method: 'PUT' , params: {id: '@id'}, isArray: false }
  });
})
```

Déjà c'est beaucoup plus court ;)

##Modification de nôtre contrôleur

Du coup il va y avoir un peu plus de code dans le contrôleur, mais à peine, et on évite mon mécanisme de callback précédent: 

**Notez bien le passage de paramètres avec `Book` à la place de `BooksServices`**.

```javascript
var MainCtrl = booksApp.controller("MainCtrl", function($scope, Models, Book) {

  $scope.books = null;
  $scope.levels = Models.levels();

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

  $scope.createBook = function(book) {
    Book.save(book).$promise.then(
      function(data) {
        $scope.getAllBooks();
      },
      function(error) {
        console.log("createBook", error);
      }
    )
  }

  $scope.updateBook = function(book) {
    Book.update({id: book._id}, book).$promise.then(
      function(data) {
        $scope.getAllBooks();
      },
      function(error) {
        console.log("updateBook", error);
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

  $scope.deleteBook = function(id) {
    Book.delete({id: id}).$promise.then(
      function(data) {
        $scope.getAllBooks();
      },
      function(error) {
        console.log("book delete ERROR", error);
      }
    )
  }

  $scope.getAllBooks()

});
```
