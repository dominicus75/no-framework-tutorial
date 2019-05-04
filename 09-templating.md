[<< előző fejezet](08-dependency-injector.md) | [következő fejezet >>](10-dynamic-pages.md)

### Sablonok

Mióta ember él a Földön, ádáz viták tárgyát képezi a sablonkezelők alkalmazása. Akadnak szép számmal olyanok, akik szerint a sablonmotor nem feltétlenül szükséges egy PHP alkalmazáshoz, mivel maga a PHP is felfogható sablonnyelvként, így a tartalom és a megjelenés szétválasztása PHP-ban is egyszerűen megoldható. A sablonkezelő e mellett – ha nem is látványos mértékben, de – lassítja az alkalmazást, mivel adatainkat egy újabb rétegen kell átvonszolni.

Viszont nem vitás, hogy egy sablonmotor megkönnyítheti például a [védőkarakterek](https://hu.wikipedia.org/wiki/Felold%C3%B3jel_(informatika)) használatát és biztonságosabbá teheti alkalmazásunkat a PHP-kód megszűrésével. E mellett használatát elsősorban akkor érdemes fontolóra venni, ha
- a design-t nem mi készítjük és a dizájnernek esetleg lövése sincs a PHP-ról,
- várható, hogy gyakran változik a megjelenés,
- zavarja a szemünk a HTML-kódban a sok `<?php echo $valtozo; ?>` vagy a [Zend Skeleton Applicationban](https://github.com/zendframework/ZendSkeletonApplication/blob/master/module/Application/view/layout/layout.phtml) is előszeretettel használt `<?= $valtozo ?>` ("rövid echo") vagy `<?= $this->printValami() ?>` kifejezés,
- tisztább határvonalat szeretnénk az alkalmazás-logika (PHP) és a megjelenítés (HTML) között.

Mivel a szerző szeretne hosszú és békés öregkort megérni, ezért nem foglal állást ebben a kardinális kérdésben, csupán ismerteti a szemben álló nézeteket, hogy az olvasó maga dönthesse el, melyik megoldást választja. Ebben segítségünkre lehet többek közt a Sinka Károly: [Template-kezelő rendszerek](https://blog.fps.hu/template-kezelo-rendszerek/) című írása, ahogy Nagy Krisztián: [PHP, mint sablonnyelv](https://deadlime.hu/2006/07/28/php-mint-sablonnyelv/) című munkája is.

Ezen útmutatóban a [Mustache](https://github.com/bobthecow/mustache.php) PHP-változatát fogjuk használni, tehát a folytatás előtt telepítsük ezt (parancssorban: `composer require mustache/mustache`).

Egy jól ismert és széles körben alkalmazott alternatíva: [Twig](http://twig.sensiolabs.org/) (telepítés: `composer require "twig/twig:^2.0"`).

Ha vetünk egy pillantást a  [Mustache_Engine](https://github.com/bobthecow/mustache.php/blob/master/src/Mustache/Engine.php) osztályra, az első, ami feltűnik, hogy nem valósít meg semmilyen interfészt. Csak type hintel egy-egy konkrét osztályt a szetterekben. Ezzel a megoldással az a probléma, hogy egy konkrét implementációhoz köti az alkalmazásunkat, így ellentmond a [függőség megfordítás elvének](https://reiteristvan.wordpress.com/2011/09/17/s-o-l-i-d-objektum-orientlt-tervezsi-elvek-5-dip/). Más szóval, az összes olyan kódunkat, amelyik ezt a sablonmotort használja, összeköti a Mustache csomaggal.

Ha módosítani szeretnénk az implementációnkon, akkor ez gondot okozhat. Lehet, hogy Twig-re szeretnénk cserélni a sablonkezelőt vagy saját osztályt létrehozni, esetleg újabb funkcionalitással bővíteni a motort. Ezt viszont nem tehetjük meg anélkül, hogy az összes olyan kódunkat ne kéne átírni, amelyik szorosan kapcsolódik a Mustache csomaghoz.

Amire szükségünk van, az egy lazább összekapcsolás. Csupán egy interfészt kelljen type hintelni a hívó osztályba, ne konkrét osztályt. Vagyis absztrakciót és ne implementációt. Így, ha esetleg új megvalósításra van szükségünk, akkor olyan új osztályt kell írnunk, amelyik szintén megvalósítja a megadott interfészt, ezért nyugodtan injektálhatjuk a kliens osztályba az eddigi függőség helyett.

A Mustache csomag kódjának babrálása helyett az [illesztő tervezési mintát](https://hu.wikipedia.org/wiki/Illeszt%C5%91_programtervez%C3%A9si_minta) fogjuk használni. Ez jóval bonyolultabbnak hangzik, mint amilyen, úgyhogy vágjunk is bele.

Először definiáljuk a szükséges interfészt. Jusson eszünkbe az [interfész elválasztási elv](http://www.clean-code-developer.hu/interface-segregation-principle-isp/), amely kimondja, hogy több specifikus interfész jobb, mint egy általános. Csak olyan metódusok megvalósítását követeljük meg az interfészünket implementáló osztályoktól, amelyekre valóban szükség lesz az adott feladathoz. Egy osztály több interfészt is használhat, ha szükséges, így nem szükséges mindent egy monstrumba gyömöszölni.

Mit is kellene tudnia a sablonkezelőnknek? Tulajdonképpen csak egy egyszerű `render` metódusra lesz szükségünk.

Hozzunk létre `Template` néven egy új alkönyvtárat az `src/` mappában, ahol a sablonmotorhoz szükséges osztályok fognak majd tanyázni.

Később, ha kódunk már nem csupán néhány osztályból fog állni, sokat segíthet az áttekinthetőségen, ha a PHP állomány elnevezésében is jelezzük, hogy mit tartalmaz. Nem szégyen a nagyoktól tanulni, így nyugodtan alkalmazhatjuk a Zend által is használt elnevezési konvenciókat. Ezek alapján a "mezei" osztályoknak nincs külön jelölése, az absztrakt-osztályok `Abstract`-előtagot kapnak, az interfészek nevéhez pedig az `Interface` utótagot csapjuk hozzá. Arra azonban ügyeljünk, hogy az osztály/interfész névnek és a PHP állomány nevének az autoloader miatt mindig meg kell egyeznie, tehát az `AbstractRequest` osztály csak az `AbstractRequest.php`-ban lakozhat, sehol másutt.

Hozzunk létre tehát egy interfészt az új könyvtárban, a fentiek szem előtt tartásával. Legyen a neve mondjuk `RendererInterface.php`, az alábbi tartalommal:

```php
<?php declare(strict_types = 1);

namespace Example\Template;

interface RendererInterface
{
    public function render($template, $data = []) : string;
}
```

Ha ezzel megvagyunk, készítsünk hozzá egy implementációt a mustache számára. Ugyanabba a könyvtárba hozzuk létre a `MustacheRenderer.php` állományt, majd töltsük meg a következő tartalommal:

```php
<?php declare(strict_types = 1);

namespace Example\Template;

use Mustache_Engine;

class MustacheRenderer implements RendererInterface
{
    private $engine;

    public function __construct(Mustache_Engine $engine)
    {
        $this->engine = $engine;
    }

    public function render($template, $data = []) : string
    {
        return $this->engine->render($template, $data);
    }
}
```

Mint láthatjuk, adapterünk meglehetősen egyszerű szerkezet. Míg az eredeti `Mustache_Engine` rendelkezik egy regiment metódussal, adapterünk csak az általunk imént definiált interfészt valósítja meg.

Természetesen be kell állítanunk egy új álnevet a `Dependencies.php` állományban újdonsült renderer-ünk számára, hogy az injektor is tudja, melyik implementációt kell befecskendeznie, ha valamely osztály a `RendererInterface`-t kéri. Ezért most adjuk hozzá az alábbi sort a `Dependencies.php` megfelelő részéhez:

`$injector->alias('Example\Template\RendererInterface', 'Example\Template\MustacheRenderer');`

Ezután a `Homepage` vezérlőnkben kell beállítani az új függőséget, valahogy így:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

use Http\Request;
use Http\Response;
use Example\Template\RendererInterface;

class Homepage
{
    private $request;
    private $response;
    private $renderer;

    public function __construct(
        Request $request,
        Response $response,
        RendererInterface $renderer
    ) {
        $this->request = $request;
        $this->response = $response;
        $this->renderer = $renderer;
    }


```

A `show` metódust szintén módosítanunk kell. Ne feledjük, hogy amíg mi csak egy szimpla tömböt adunk át, addig a Mustache lehetőséget ad view context objektum átadására is. Erre később még vissza fogunk térni, most azonban ne bonyolítsuk túl, ha nem szükséges. Írjuk át a `Homepage` vezérlőnk `show` metódusát az alábbiak szerint:

```php
    public function show()
    {
        $data = [
            'name' => $this->request->getParameter('name', 'stranger'),
        ];
        $html = $this->renderer->render('Hello {{name}}', $data);
        $this->response->setContent($html);
    }
```

Lessük meg a böngészőnkben, hogy minden megfelelően működik-e. A Mustache alapértelmezésben egy karakterlánc-kezelőt használ. Ha sablonfájlokat szeretnénk e helyett, akkor át kell adnunk az ehhez szükséges opciókat tartalmazó tömböt a `Mustache_Engine` konstruktorának. Nyissuk meg újra a `Dependencies.php` állományt, majd adjuk hozzá a következő kódrészletet:

```php
$injector->define('Mustache_Engine', [
    ':options' => [
        'loader' => new Mustache_Loader_FilesystemLoader(dirname(__DIR__) . '/templates', [
            'extension' => '.html',
        ]),
    ],
]);
```

Átadunk egy adattömböt mert `.html` kiterjesztést szeretnénk használni az alapértelmezett `.mustache` helyett. Miért? Más sablonnyelvek is hasonló szintaxist alkalmaznak, így ha valaha úgy határozunk, hogy másra váltunk, nem kell átnevezni a sablon állományainkat.

Projektünk gyökérmappájában hozzunk létre egy `templates` nevű könyvtárat, majd helyezzünk el benne egy `Homepage.html`névre hallgató új állományt. Ebbe pedig a következő kódot írjuk:

```
<h1>Hello World</h1>
Hello {{ name }}
```

Visszatérve `Homepage` vezérlőnkhöz, keressük meg a `show` metódusban a renderelésért felelős sort (`$html = $this->renderer->render('Hello {{name}}', $data);`) és cseréljük ki erre: `$html = $this->renderer->render('Homepage', $data);`.

Ha végeztünk, nyissuk meg a böngészőben a kezdőoldalt, hogy megbizonyosodjunk róla, minden megfelelően működik. Ha készen vagyunk, akkor ne felejtsünk el commitolni sem.

[<< előző fejezet](08-dependency-injector.md) | [következő fejezet >>](10-dynamic-pages.md)
