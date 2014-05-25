#Un peu d'ihm et de formulaire

Mon application est plutôt "moche" donc nous allons un peu travailler sur le code HTML.

##Mais d'abord le contrôleur

Pour pouvoir interagir bien mieux avec l'IHM je vous propose d'ajouter quelques méthodes à notre contrôleur.

**`selectBook()`**: je veux pouvoir sélectionner un livre dans la liste

```javascript
$scope.selectBook = function(book) {
  $scope.book = book;
}
```

**`saveBook()`**: je fais un update si mon livre a une propriété `_id` renseignée, sinon c'est une création

```javascript
$scope.saveBook = function(book) {
  book._id !== undefined ? $scope.updateBook(book) : $scope.createBook(book);
}
```

**`newBook()`**: je créée un livre "vide" (cela me servira à vider mon formulaire)

```javascript
$scope.newBook = function() {
  $scope.book = {};
}
```

##On peut passer à l'IHM

Pour une fois, je ne vais pas utiliser **Bootstrap** mais **UIkit** [http://www.getuikit.com/](http://www.getuikit.com/) que je trouve assez joli (et j'avais envie de changer un peu).

Donc dans notre fichier `bower.json`, nous ajoutons une ligne pour UIkit:

    {
      "name": "books",
      "version": "0.0.0",
      "dependencies": {
        "angular" : "1.2.16",
        "angular-resource" : "1.2.16",
        "uikit" : "2.6.0"
      }
    }

Ensuite vous faite un `bower update` dans votre console/terminal. Et vous êtes prêts à refaire la page `index.html` (dans `/public`). Vous pouvez tout supprimer, nous allons reprendre tout le code html.

**Voici la nouvelle structure de votre page**:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>livres</title>
    <link rel="stylesheet" href="bower_components/uikit/dist/css/uikit.almost-flat.min.css" />
  </head>
  <body ng-app="booksApp" style="padding: 10px"> <!-- on n'oublie pas la directive ng-app -->

    <div class="uk-container uk-container-center">

      <div class="uk-grid" ng-controller="MainCtrl"> <!-- on n'oublie pas la directive ng-controller -->

        <div class="uk-width-4-10">
          <!-- FORMULAIRE : Ici nous aurons notre formulaire de saisie -->
        </div>

        <div class="uk-width-6-10">
          <!-- GRILLE : Ici nous aurons la liste des livres -->
        </div>

      </div>

    </div>

    <script src="bower_components/angular/angular.min.js"></script>
    <script src="bower_components/angular-resource/angular-resource.min.js"></script>
    <script src="js/main.js"></script>

  </body>
</html>
```

**Le formulaire**: à l'endroit où il y a le commentaire `<!-- FORMULAIRE : Ici nous aurons notre formulaire de saisie -->`, insérez le code suivant:

```html
<div class="uk-panel">

  <h3 class="uk-panel-title">Formulaire Livre</h3>

    <form class="uk-form" name="bookForm">
      <input ng-model="book.title" name="title" placeholder="titre du film">
      <input ng-model="book.description" placeholder="description du film">
      <select ng-model="book.level"
              ng-options="level for level in levels">
      </select>

      <p ng-model="book">{{book._id}}</p>

      <hr>
      <a href="#" ng-click="newBook()">Nouveau</a> |
      <a href="#" ng-click="saveBook(book)">Valider (ajout ou modification)</a> |
      <a href="#" ng-click="deleteBook(book._id)">Supprimer</a> |
      <hr>

    </form>

</div>
```

Nous retrouvons bien:

- les directives `ng-model` dans les tag `<input>` et `<select>` pour fair le lien avec le modèle `book`
- j'ai ajouté : `<p ng-model="book">{{book._id}}</p>` pour afficher l'id du livre quand il existe (donc quand il a déjà été sauvegardé en base).

Nous avons ensuite une liste de liens:

- `<a href="#" ng-click="newBook()">Nouveau</a>` qui permet de "vider" notre formulaire en appelant la nouvelle méthode `newBook()` de notre contrôleur
- `<a href="#" ng-click="saveBook(book)">Valider (ajout ou modification)</a>` qui permet d'appeler la nouvelle méthode `saveBook(book)` de notre contrôleur en lui passant les valeurs de `book` (lié au formulaire), et qui déterminera en fonction de la présence ou non d'un id si c'est une modification ou un ajout
- `<a href="#" ng-click="deleteBook(book._id)">Supprimer</a>` pour supprimer le livre sélectionné

**La grille (ou table) des livres**: à l'endroit où il y a le commentaire `<!-- GRILLE : Ici nous aurons la liste des livres -->`, insérez le code suivant:

```html
<div class="uk-panel">

  <h3 class="uk-panel-title">Liste des Livres</h3>

  <table class="uk-table uk-table-hover">
    <thead>
    <tr>
      <th>Titre</th>
      <th>Description</th>
      <th>Niveau</th>
      <th>id</th>
    </tr>
    </thead>

    <tbody ng-repeat="book in books" ng-click="selectBook(book)">
    <tr>
      <td>{{book.title}}</td>
      <td>{{book.description}}</td>
      <td>{{book.level}}</td>
      <td>{{book._id}}</td>
    </tr>
    </tbody>
  </table>

</div>
```

Nous avons toujours notre directive `ng-repeat="book in books"`qui va nous permettre d'ajouter autant de lignes(rows) ou `<tr>` dans notre grille ou `<table>`.
Ainsi que `ng-click="selectBook(book)"` qui permettra ainsi d'afficher le livre dans le formulaire si on clique sur la ligne correspondante.

Donc si ce n'est déjà fait, relancez votre application côté serveur et connectez vous sur votre splendide "Single Page Application" de gestion de livre, et jouez avec:

<img src="images/ang-form-01.png" height="70%" width="70%">

Vous allez rapidement vous apercevoir que vous pouvez ajouter des livres "vides", c'est moche!

<img src="images/ang-form-02.png" height="70%" width="70%">

Nous avons donc un prétexte pour regarder comment fonctionne le contrôle de saisie avec Angular (il faudrait aussi modifier côté serveur notre API pour empêcher d'avoir un livre "vide").

Nous allons voir une infime partie des possibilités proposées par Angular, mais déjà bien pratique. Pour aller plus loin, vous pouvez commencer par ceci [https://docs.angularjs.org/api/ng/type/form.FormController](https://docs.angularjs.org/api/ng/type/form.FormController) et [https://docs.angularjs.org/guide/forms](https://docs.angularjs.org/guide/forms).

##Validation

Nous avions le formulaire suivant:

```html
<form class="uk-form" name="bookForm">
  <input ng-model="book.title" name="title" placeholder="titre du film">
  <input ng-model="book.description" placeholder="description du film">
  <select ng-model="book.level"
          ng-options="level for level in levels">
  </select>

  <p ng-model="book">{{book._id}}</p>

  <hr>
  <a href="#" ng-click="newBook()">Nouveau</a> |
  <a href="#" ng-click="saveBook(book)">Valider (ajout ou modification)</a> |
  <a href="#" ng-click="deleteBook(book._id)">Supprimer</a> |
  <hr>

</form>
```

Vous pouvez noter que le formulaire a un nom `name="bookForm"` et que le champ de saisie des titres a lui aussi un nom `name="title"`. Cela va nous servir à tester la validité de notre saisie. Modifions notre formulaire comme suit:

On ajoute l'attribut `novalidate` au tag `<form></form>`:

```html
<form class="uk-form" name="bookForm" novalidate>
```

*Comme cela on n'utilise pas les messages d'erreurs natifs*

On ajoute l'attribut `required` au champ de saisie des titres:

```html
  <input ng-model="book.title" name="title" placeholder="titre du film" required>
```

Ensuite on ajoute un message qui n'apparaîtra que si le champ de saisie de titre est vide:

```html
  <span ng-show="bookForm.title.$error.required" class="uk-text-warning">Champs obligatoire</span>
```

Cette partie là reste inchangée:

```html
  <input ng-model="book.description" placeholder="description du film">
  <select ng-model="book.level"
          ng-options="level for level in levels">
  </select>
  <p ng-model="book">{{book._id}}</p>
```

Remplaçons nos liens par des boutons et précisons pour le bouton de sauvegarde qu'il n'est actif uniquement si les données du formulaire sont valide avec l'attribut : `ng-disabled="bookForm.$invalid"`:

```html
  <hr>
  <button class="uk-button" ng-click="newBook()">Nouveau</button> |
  <button class="uk-button uk-button-primary" ng-disabled="bookForm.$invalid" ng-click="saveBook(book)">Valider (ajout ou modification)</button> |
  <button class="uk-button uk-button-danger" ng-click="deleteBook(book._id)">Supprimer</button> |
  <hr>

</form>
```

Sauvegardez, relancez et vérifiez que cela fonctionne:

<img src="images/ang-form-03.png" height="80%" width="80%">

<img src="images/ang-form-04.png" height="80%" width="80%">

Simple et pratique.
