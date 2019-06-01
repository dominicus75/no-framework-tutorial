[<< előző fejezet](10-dynamic-pages.md) | [következő fejezet >>](12-frontend.md)

### Navigációs menü

Az előző fejezetben elkészítettünk néhány dinamikus oldalt, viszont senki sem találja őket. Ezt a problémát fogjuk most orvosolni. Ebben a fejezetben ugyanis olyan menüt fogunk készíteni, amely tartalmazza minden oldalunk linkjét.

Ha van egy ilyen menünk, nyilván azt szeretnénk, hogy fel tudjuk használni a kódját az összes oldalunkon. Elhelyezhetjük például egy különálló fájlban, hogy azt illeszthessük majd minden oldalunkba. Ennél azonban van jobb megoldás is.

Praktikusabb, ha olyan sablonokat használunk, amelyek képesek más sablonok képességeinek kibővítésére. Így az elrendezéssel kapcsolatos összes kódot egyetlen állományban tudhatjuk és nem kell a header és footer fájlokat minden sablonba egyenként beillesztenünk.

Ezt a megoldást viszont Mustache-implementációnk nem támogatja. Át tudnánk írni a kódunkat úgy, hogy ezt elhárítsuk, viszont ez egyrészt időbe kerülne, másrészt a hibalehetőségek is megszaporodnának. Mindezt elkerülendő átválthatunk egy olyan csomagra is, amelyik támogatja ezt és emellett megfelelően ki van próbálva, mint például a [Twig](http://twig.sensiolabs.org/).

Talán most csodálkozik a nyájas olvasó, hogy ha ezt eddig is tudtuk, akkor miért nem mindjárt a Twig-et használjuk. A válasz pedagógiai természetű: esetünk remek példa arra, miért hasznos az interfészek használata és az egymáshoz lazán kapcsolódó osztályok írása. A való világhoz hasonlóan a körülmények váratlan hirtelenséggel változhatnak, s kódunknak ehhez alkalmazkodnia kell.

Emlékszünk még, hogyan hoztuk létre a `MustacheRenderer`-t a [9. fejezetben](09-templating.md)? Most hasonlóan fogjuk megalkotni a `TwigRenderer`-t, ami ugyanazt a `RendererInterface`-t fogja megvalósítani, ezért csereszabatos lesz a `MustacheRenderer` osztállyal.

Mielőtt hozzálátnánk, telepítsük a Twig legújabb változatát a Composer segítségével (`composer require "twig/twig:~2.0"`).

Ezután hozzuk létre a `TwigRenderer.php` állományt az `src/Template` mappánkban, a következő tartalommal:

```php
<?php declare(strict_types = 1);

namespace Example\Template;

use Twig_Environment;

class TwigRenderer implements RendererInterface
{
  private $renderer;

  public function __construct(Twig_Environment $renderer)
  {
    $this->renderer = $renderer;
  }

  public function render($template, $data = []) : string
  {
    return $this->renderer->render("$template.html", $data);
  }
}
```

Mint látható, a `render` metódusban (a `MustacheRenderer` azonos nevű metódusától eltérően) a `.html` kiterjesztést is külön feltüntettük. Erre azért van szükség, mert a Twig alapértelmezésben nem rendel a sablonhoz fájlkiterjesztést, így ezt nekünk kell minden esetben megtennünk. Ezzel a módszerrel ugyanúgy tudjuk használni, mint a Mustache csomagot.

Egészítsük ki a `Dependencies.php` állományunkat a következő kóddal:

```php
$injector->delegate('Twig_Environment', function () use ($injector) {
  $loader = new Twig_Loader_FilesystemLoader(dirname(__DIR__) . '/templates');
  $twig = new Twig_Environment($loader);
  return $twig;
});
```

Ahelyett, hogy az eddig megszokott `define()` metódus segítségével meghatároznánk a függőségeket, ebben a példában a `delegate()` meghívásával egy anonim függvénynek fogjuk átadni a kért osztály létrehozásának feladatát. A jövőben ez még hasznos lesz számunkra.

Most már lecserélhetjük a `RendererInterface`-hez rendelt `MustacheRenderer` implementációt a `TwigRenderer` osztályunkra, hogy ezt használhassuk a Mustache helyett.

Ha megnézzük a webhelyünket a böngészőben, akkor mindennek úgy kell működnie, mint azelőtt. Ha ez így van, akkor hozzá is kezdhetünk a menü elkészítéséhez.

Először is küldjünk egy adattömböt a sablonunknak. Ehhez nyissuk meg a `Homepage` vezérlőnket és cseréljük ki a `$data` tömb tartalmát a következőre:

```php
$data = [
  'name' => $this->request->getParameter('name', 'stranger'),
  'menuItems' => [['href' => '/', 'text' => 'Homepage']],
];
```

Majd a `templates/Homepage.html` elejére szúrjuk be az alábbi kódot:

```php
{% for item in menuItems %}
  <a href="{{ item.href }}">{{ item.text }}</a><br>
{% endfor %}
```

Ha ezután frissítjük a nyitólapunkat a böngészőben, látnunk kell az imént beállított hivatkozást is. A menünk tehát már működik a nyitólapon, viszont mi az összes oldalon látni szeretnénk. Át is másolhatnánk az összes sablon-állományunkba, de ha ezután valamit meg akarunk változtatni, akkor azt az összes kapcsolódó állományban meg kell tennünk.

Ez nem tűnik túl jó ötletnek. Ehelyett egy olyan elrendezés-sablont fogunk használni, amelyhez az összes sablon alkalmazkodhat. Hozzuk is létre a `Layout.html` állományt a `templates` könyvtárba, az alábbi tartalommal:

```php
{% for item in menuItems %}
  <a href="{{ item['href'] }}">{{ item['text'] }}</a><br>
{% endfor %}
<hr>
{% block content %}
{% endblock %}
```

Ezután ugyanitt cseréljük le `Homepage.html` állományunk tartalmát az alábbiakra:

```php
{% extends "Layout.html" %}
{% block content %}
  <h1>Hello World</h1>
  Hello {{ name }}
{% endblock %}
```

Majd a `Page.html` tartalmát erre:

```php
{% extends "Layout.html" %}
{% block content %}
  {{ content }}
{% endblock %}
```

Ha ráfrissítünk a nyitólapra a böngészőben, akkor látnunk kell a menüt. Viszont ha egy aloldalra navigálunk, akkor a menü helyett csak egy `<hr>` sor lesz látható.

The problem is that we are only passing the `menuItems` to the homepage. Doing that over and over again for all pages would be a bit tedious and a lot of work if something changes. So let's fix that in the next step.

We could create a global variable that is usable by all templates, but that is not a good idea here. We will add different parts of the site in the future like an admin area and we will have a different menu there.

So instead we will use a custom renderer for the frontend. First we create an empty interface that extends the existing `Renderer` interface.

```php
<?php declare(strict_types = 1);

namespace Example\Template;

interface FrontendRenderer extends RendererInterface {}
```

By extending it we are saying that any class implementing the `FrontendRenderer` interface can be used where a `Renderer` is required. But not the other way around, because the `FrontendRenderer` can have more functionality as long as it still fulfills the `Renderer` interface.

Now of course we also need a class that implements the new interface.


```php
<?php declare(strict_types = 1);

namespace Example\Template;

class FrontendTwigRenderer implements FrontendRenderer
{
  private $renderer;

  public function __construct(Renderer $renderer)
  {
    $this->renderer = $renderer;
  }

  public function render($template, $data = []) : string
  {
    $data = array_merge($data, [
      'menuItems' => [['href' => '/', 'text' => 'Homepage']],
    ]);
    return $this->renderer->render($template, $data);
  }
}
```

As you can see we have a dependency on a `Renderer` in this class. This class is a wrapper for our `Renderer` and adds the `menuItems` to all `$data` arrays.

Of course we also need to add another alias to the dependencies file.

```php
$injector->alias('Example\Template\FrontendRenderer', 'Example\Template\FrontendTwigRenderer');
```

Now go to your controllers and exchange all references of `Renderer` with `FrontendRenderer`. Make sure you change it in both the `use` statement at the top and in the constructor.

Also delete the following line from the `Homepage` controller:

```php
'menuItems' => [['href' => '/', 'text' => 'Homepage']],
```

Once that is done, you should see the menu on both the homepage and your subpages.

Everything should work now, but it doesn't really make sense that the menu is defined in the `FrontendTwigRenderer`. So let's refactor that and move it into it's own class.

Right now the menu is defined in the array, but it is very likely that this will change in the future. Maybe you want to define it in the database or maybe you even want to generate it dynamically based on the pages available. We don't have this information and things might change in the future.

So let's do the right thing here and start with an interface again. But first, create a new folder in the `src` directory for the menu related things. `Menu` sounds like a reasonable name, doesn't it?

```php
<?php declare(strict_types = 1);

namespace Example\Menu;

interface MenuReader
{
  public function readMenu() : array;
}
```

And our very simple implementation will look like this:

```php
<?php declare(strict_types = 1);

namespace Example\Menu;

class ArrayMenuReader implements MenuReader
{
  public function readMenu() : array
  {
    return [
      ['href' => '/', 'text' => 'Homepage'],
    ];
  }
}
```

This is only a temporary solution to keep things moving forward. We are going to revisit this later.

Before we continue, let's edit the dependencies file to make sure that our application knows which implementation to use when the interface is requested.

Add these lines above the `return` statement:

```php
$injector->alias('Example\Menu\MenuReader', 'Example\Menu\ArrayMenuReader');
$injector->share('Example\Menu\ArrayMenuReader');
```

Now you need to change out the hardcoded array in the `FrontendTwigRenderer` class to make it use our new `MenuReader` instead. Give it a try without looking at the solution below.

Did you finish it or did you get stuck? Or are you just lazy? Doesn't matter, here is a working solution:

```php
<?php declare(strict_types = 1);

namespace Example\Template;

use Example\Menu\MenuReader;

class FrontendTwigRenderer implements FrontendRenderer
{
  private $renderer;
  private $menuReader;

  public function __construct(RendererInterface $renderer, MenuReader $menuReader)
  {
    $this->renderer = $renderer;
    $this->menuReader = $menuReader;
  }

  public function render($template, $data = []) : string
  {
    $data = array_merge($data, [
      'menuItems' => $this->menuReader->readMenu(),
    ]);
    return $this->renderer->render($template, $data);
  }
}
```

Még mindig működik? Remek. Akkor ne felejtsünk el commitolni, mielőtt továbblépnénk a következő fejezetre.

[<< előző fejezet](10-dynamic-pages.md) | [következő fejezet >>](12-frontend.md)
