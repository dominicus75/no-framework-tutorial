[<< előző fejezet](10-dynamic-pages.md) | [következő fejezet >>](12-frontend.md)

### Navigációs menü

Az előző fejezetben elkészítettünk néhány dinamikus oldalt, viszont senki sem találja őket. Ezt a problémát fogjuk most orvosolni. Ebben a fejezetben ugyanis olyan menüt fogunk készíteni, amely tartalmazza minden oldalunk linkjét.

Ha van egy ilyen menünk, nyilván azt szeretnénk, hogy fel tudjuk használni a kódját az összes oldalunkon. Elhelyezhetjük például egy különálló fájlban, hogy azt illeszthessük majd minden oldalunkba. Ennél azonban van jobb megoldás is.

Praktikusabb, ha olyan sablonokat használunk, amelyek képesek más sablonok képességeinek kibővítésére. Így az elrendezéssel kapcsolatos összes kódot egyetlen állományban tudhatjuk és nem kell a header és footer fájlokat minden sablonba egyenként beillesztenünk.

Ezt a megoldást viszont Mustache-implementációnk nem támogatja. Át tudnánk írni a kódunkat úgy, hogy ezt elhárítsuk, viszont ez egyrészt időbe kerülne, másrészt a hibalehetőségek is megszaporodnának. Mindezt elkerülendő átválthatunk egy olyan csomagra is, amelyik támogatja ezt és emellett megfelelően ki van próbálva, mint például a [Twig](http://twig.sensiolabs.org/).

Talán most csodálkozik a nyájas olvasó, hogy ha ezt eddig is tudtuk, akkor miért nem mindjárt a Twig-et használjuk. A válasz pedagógiai természetű: esetünk remek példa arra, miért hasznos az interfészek használata és az egymáshoz lazán kapcsolódó osztályok írása. A való világhoz hasonlóan a körülmények váratlan hirtelenséggel változhatnak, s kódunknak ehhez alkalmazkodnia kell.

Emlékszünk még, hogyan hoztuk létre a `MustacheRenderer`-t a [9. fejezetben](09-templating.md)? Most hasonlóan fogjuk megalkotni a `TwigRenderer`-t, ami ugyanazt a `RendererInterface`-t fogja megvalósítani, ezért csereszabatos lesz a `MustacheRenderer` osztállyal.

Mielőtt hozzálátnánk, telepítsük a Twig legújabb változatát a Composer segítségével (`composer require "twig/twig:~1.0"`).

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
$injector->delegate('Twig\Environment', function () use ($injector) {
  $loader = new Twig\Loader\FilesystemLoader(dirname(__DIR__) . '/templates');
  $twig = new Twig\Environment($loader);
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
  'menuItems' => [['href' => '/', 'text' => 'Kezdőlap']],
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

A probléma az, hogy csupán a `Homepage` vezérlőben létrehozott `menuItems` tömb elemeit írtuk ki és azokat is csak a nyitólapra. Létrehozhatnánk ezt a tömböt minden más vezérlőben is (pl. `Page`), ahol az ezt is tartalmazó `$data` tömböt használjuk, de ez nagyon macerás lenne és nem is oldaná meg a problémánkat, mivel ezt a későbbiekben minden változtatás esetén újra és újra végig kellene zongorázni az összes vezérlőnkön.

Valami jobbat kell kitalálnunk, olyat amivel nem csinálunk magunknak fölösleges pluszmunkát a későbbiekben.

Létrehozhatunk akár egy globális változót, amelyet az összes sablon használhat, de itt ez sem tűnik túl jó ötletnek. Később ugyanis újabb részeket hozhatunk létre a webhelyhez, mondjuk egy adminisztrációs felületet, s ez valószínűleg egy külön menüvel fog rendelkezni.

Ehelyett egy külön megjelenítőt fogunk használni a frontendhez. Először egy üres interfészt hozunk létre, amelyet a  már létező `RendererInterface`-ből fogunk leörökíteni az alábbiak szerint:

```php
<?php declare(strict_types = 1);

namespace Example\Template;

interface FrontendRendererInteface extends RendererInterface {}

```

A `RendererInterface` kiterjesztésével deklaráljuk, hogy bármely olyan osztály, amely implementálja a `FrontendRendererInteface`-t használható ott, ahol a `RendererInterface` van megkövetelve. De mindez megfordítva nem működik, mert a `FrontendRendererInteface` (mint leszármazott, elméletileg) több funkcionalitással bír, amellett, hogy örökli a `RendererInterface` metódusait is. Ez a megoldás a későbbiekben azért lehet hasznos, mert a `FrontendRendererInteface`-t úgy tudjuk új, frontend-specifikus metódusokkal bővíteni (akár leszármaztatással is), hogy az általánosabb feladatú `RendererInterface`-hez nem nyúlunk, ezért az azt implementáló osztályainkhoz sem kell (jusson itt eszünkbe az Interface elválasztási elv is).

Most természetesen szükségünk lesz egy osztályra is, amelyik implementálja az új interfészünket. Íme:


```php
<?php declare(strict_types = 1);

namespace Example\Template;

class FrontendTwigRenderer implements FrontendRendererInteface
{
  private $renderer;

  public function __construct(RendererInterface $renderer)
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

Ahogy a kódból is kitűnik, osztályunk rendelkezik egy `RendererInterface` függőséggel. Ezért ez az osztály egy burkoló (közkeletűbb nevén: wrapper) lesz a `RendererInterface`-hez, a feladata pedig az, hogy minden `$data` tömbhöz hozzáadja a `menuItems` elemet a megadott tartalommal.

Természetesen a fenti változásokat a `src/Dependencies.php` állományban is meg kell örökítenünk:

```php
$injector->alias('Example\Template\FrontendRendererInteface', 'Example\Template\FrontendTwigRenderer');
```

Ezután az összes vezérlőnkben (`Homepage.php` és `Page.php`) le kell cserélnünk a `RendererInterface`-re való hivatkozásokat a `FrontendRendererInteface`-re. Győződjünk meg arról is, hogy mind a `use` kifejezésben, mind a konstruktorban a `FrontendRendererInteface` szerepeljen az eddigi `RendererInterface` helyett.

Majd szintén töröljük a következő sort a `Homepage` vezérlőből (a `show` metódusban), mivel ezt a műveletet már a `FrontendTwigRenderer` osztályunk azonos nevű metódusa fogja elvégezni:

```php
'menuItems' => [['href' => '/', 'text' => 'Homepage']],
```

Ha mindezzel kész vagyunk, akkor a menüt immár a nyitólapon és az aloldalakon (pl. `/one-page`) is látnunk kell(ene).

Ugyan most már mindennek működnie kell, de igazából még sincs túl sok értelme, hogy a menüt a `FrontendTwigRenderer` osztályban hozzuk létre (a vezérlők helyett). Helyezzük is át egy saját osztályba!

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

class FrontendTwigRenderer implements FrontendRendererInteface
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
