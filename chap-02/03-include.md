#La directive include

Alors une partie très courte, mais néanmoins elle pourra peut-être changer votre vie de développeur front (ok, j'y vais un peu fort, mais ...). Combien de fois j'ai pesté parce que mon code html devenait trop "touffu", illisible, ... Dans ces moments là, il ya plusieurs solutions : bricolage avec jquery pour charger les templates, appel au plugin text de require, grunt, ... mais toujours une gymnastique à force pénible pour arriver à modulariser ses page html.

Avec Angular c'est simple, en 1 seule directive vous pouvez régler tous vos problèmes.

##Restructuration de notre projet (côté front)

Actuellement nous avons une structure **front** qui doit ressembler à ceci:


    votre_projet/
    ├── public/
    |   ├── bower_components/ /* angular est par ici */
    |   ├── js/
    |   |    └── main.js
    |   └── index.html
    ├── bower.json
    └── .bowerrc

Créez donc dans `/public` un répertoire `books` avec 2 fichiers html à l'intérieur : `bookForm.html` et `booksList.html`, comme ceci:

    votre_projet/
    ├── public/
    |   ├── bower_components/ /* angular est par ici */
    |   ├── js/
    |   |    └── main.js
    |   ├── books/
    |   |    ├── bookForm.html
    |   |    └── booksList.html
    |   └── index.html
    ├── bower.json
    └── .bowerrc

Dans votre page `index.html` extrayez complètement le code correspondant à votre formulaire pour le coller dans `bookForm.html`. Le contenu de `bookForm.html` devrait être celui-ci:

{% highlight html %}
<h3 class="uk-panel-title">Formulaire Livre</h3>

<form class="uk-form" name="bookForm" novalidate>
  <input ng-model="book.title" name="title" placeholder="titre du film" required>

  <span ng-show="bookForm.title.$error.required" class="uk-text-warning">Champs obligatoire</span>

  <input ng-model="book.description" placeholder="description du film">
  <select ng-model="book.level"
          ng-options="level for level in levels">
  </select>
  <p ng-model="book">{% raw %}{{book._id}}{% endraw %}</p>

  <hr>
  <button class="uk-button" ng-click="newBook()">Nouveau</button> |
  <button class="uk-button uk-button-primary" ng-disabled="bookForm.$invalid" ng-click="saveBook(book)">Valider (ajout ou modification)</button> |
  <button class="uk-button uk-button-danger" ng-click="deleteBook(book._id)">Supprimer</button> |
  <hr>

</form>
{% endhighlight %}

Ensuite, toujours de votre page `index.html` extrayez complètement le code correspondant à votre liste de livre pour le coller dans `booksList.html`. Le contenu de `booksList.html` devrait être celui-ci:

{% highlight html %}
<h3 class="uk-panel-title">Liste des Livres</h3>

<form class="uk-form">
  <input ng-model="search.$" placeholder="chercher un livre">
  <input ng-model="search.title" placeholder="par titre">
  <input ng-model="search.description" placeholder="par description">
  <select ng-model="search.level"
          ng-options="level for level in levels">
  </select>
</form>

<table class="uk-table uk-table-hover">
  <thead>
  <tr>
    <th>Titre</th>
    <th>Description</th>
    <th>Niveau</th>
    <th>id</th>
  </tr>
  </thead>

  <tbody ng-repeat="book in books | filter:search:strict" ng-click="selectBook(book)">

  <tr>
    <td>{% raw %}{{book.title}}{% endraw %}</td>
    <td>{% raw %}{{book.description}}{% endraw %}</td>
    <td>{% raw %}{{book.level | stars}}{% endraw %}</td>
    <td>{% raw %}{{book._id}}{% endraw %}</td>
  </tr>
  </tbody>
</table>
{% endhighlight %}

Du coup, votre page `index.html` devrait ressembler à ceci:

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>livres</title>
    <link rel="stylesheet" href="bower_components/uikit/dist/css/uikit.almost-flat.min.css" />
  </head>

  <body ng-app="booksApp" style="padding: 10px">

    <div class="uk-container uk-container-center">

      <div class="uk-grid" ng-controller="MainCtrl">
        <div class="uk-width-4-10">

          <div class="uk-panel">
            <!-- ici il y avait la saisie des livres -->
          </div>

        </div>

        <div class="uk-width-6-10">

          <div class="uk-panel">
            <!-- ici il y avait la liste des livres -->
          </div>

        </div>
      </div>
    </div>

    <script src="bower_components/angular/angular.min.js"></script>
    <script src="bower_components/angular-resource/angular-resource.min.js"></script>

    <script src="js/main.js"></script>

  </body>
</html>
{% endhighlight %}

Alors c'est tout de suite plus propre, mais ça marche forcément beaucoup moins bien ...

##La "magie" avec Angular

Nous devons juste nous contenter d'expliquer à Angular où sont `booksList.html` et `booksList.html`. Et pour cela la directive `ng-include` va grandement nous faciliter la vie. Pour cela,

Remplacez:

{% highlight html %}
<div class="uk-panel">
  <!-- ici il y avait la saisie des livres -->
</div>
{% endhighlight %}

par:

{% highlight html %}
<div class="uk-panel" ng-include="'books/bookForm.html'" onload="whenFormIsLoaded()"></div>
{% endhighlight %}

Angular à partir de `ng-include` ira charger `books/bookForm.html`et lancera grâce à `onload` la méthode `whenFormIsLoaded()`du contrôleur (il faudra créer la méthode, mais `onload` n'est pas obligatoire).

Puis remplacez:

{% highlight html %}
<div class="uk-panel">
  <!-- ici il y avait la liste des livres -->
</div>
{% endhighlight %}

par:

{% highlight html %}
<div class="uk-panel" ng-include="'books/booksList.html'" onload="whenListIsLoaded()"></div>
{% endhighlight %}

Angular à partir de `ng-include` ira charger `books/booksList.html`et lancera grâce à `onload` la méthode `whenListIsLoaded()`du contrôleur (il faudra créer la méthode, mais `onload` n'est pas obligatoire).

Donc le code définitif de `index.html` devrait être le suivant:

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>livres</title>
    <link rel="stylesheet" href="bower_components/uikit/dist/css/uikit.almost-flat.min.css" />
  </head>

  <body ng-app="booksApp" style="padding: 10px">

    <div class="uk-container uk-container-center">

      <div class="uk-grid" ng-controller="MainCtrl">
        <div class="uk-width-4-10">

          <div class="uk-panel" ng-include="'books/bookForm.html'" onload="whenFormIsLoaded()"></div>

        </div>

        <div class="uk-width-6-10">

          <div class="uk-panel" ng-include="'books/booksList.html'" onload="whenListIsLoaded()"></div>

        </div>
      </div>
    </div>

    <script src="bower_components/angular/angular.min.js"></script>
    <script src="bower_components/angular-resource/angular-resource.min.js"></script>

    <script src="js/main.js"></script>

  </body>
</html>
{% endhighlight %}

Il ne vous reste plus qu'à tester. Je vous laisse imaginer comment cela va être facile de modulariser vos projets et de vous en faciliter la maintenance.
