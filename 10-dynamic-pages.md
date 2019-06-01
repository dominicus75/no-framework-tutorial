[<< előző fejezet](09-templating.md) | [következő fejezet >>](11-page-menu.md)

### Dinamikus oldalak

Eddig csupán egy minimális funkcionalitású statikus oldallal lettünk gazdagabbak. Lépjünk is túl a nem kimondottan fantáziadús hello world példán, és adjunk némi valódi funkcionalitást az alkalmazásunknak.

Első funkciónk [markdown](https://szit.hu/doku.php?id=oktatas:web:markdown) állományokból történő dinamikus oldalak generálása lesz (*Fordítói megjegyzés: a szerző az **oldal** kifejezést a Drupalból vagy a WordPress-ből ismert [jelentéstartalommal](http://wphu.org/konyv/2-2-oldalak/) használja, tehát egy tartalomtípust ért alatta, nem úgy általában egy weboldalt*).

Ehhez hozzuk létre a `Page` vezérlőt az alábbi tartalommal:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

class Page
{
  public function show($params)
  {
    var_dump($params);
  }
}
```
Ha ezzel megvagyunk, adjunk hozzá egy új route-ot is az `src/Routes.php` állományban:

```php
['GET', '/{slug}', ['Example\Controllers\Page', 'show']],
```

Most próbáljuk ki a művünket és hívjuk meg néhány tetszőlegesen kiválasztott url-el az alkalmazásunkat, például a `http://localhost:8000/test` vagy a `http://localhost:8000/hello` éppen megfelel. Amint láthatjuk, minden alkalommal a `Page` vezérlőt hívtuk meg, a `$params` tömbben átadva neki a az oldal "[szép url](http://webmestertanfolyam.hu/webmester-blog/szep-url-generalas)"-jét (*A fordító megjegyzése: az eredeti szövegben itt a **slug** kifejezés szerepel, aminek a jelentését – elterjedt fordítás hiányában – szép vagy keresőbarát url-ként lehet legjobban visszaadni, ha esetleg a Gugli Fordító által felajánlott **meztelen csiga** nem nyeri el a tetszésünket...*).

Kezdetnek hozzunk létre néhány oldalt. Még nem fogunk adatbázist használni, ezért most létrehozunk egy új mappát a projektünk gyökerében, amit `pages`-nek fogunk hívni. Ebben a könyvtárban fogjuk elhelyezni az `.md` kiterjesztésű szöveges állományainkat, amelyek oldalaink tartalmát lesznek hivatva tárolni. Először hozzunk létre egy `page-one.md` elnevezésű fájlt, az alábbi kifejező tartalommal:

```
Ez itt egy oldal.
```

Most meg kell még írnunk néhány kódot, hogy azokat a megfelelő fájl olvasni tudja a tartalom megjelenítéséhez. Csábítónak tűnhet beleönteni az összes kódot a `Page` vezérlőnkbe, de ne feledkezzünk meg a [vonatkozások szétválasztásáról (SoC)](http://www.clean-code-developer.hu/separation-of-concerns-soc/) sem. Jó esélyünk van ugyanis arra, hogy ezeket az oldalakat az alkalmazás más felületeiről (pl. admin) is olvasnunk vagy esetleg írnunk kell.

Fentiekre figyelemmel helyezzük el a funkcionalitást egy külön osztályba. Később akár az is előfordulhat, hogy a most használt `.md`-fájlokról valamilyen adatbázisra váltunk, ezért nem árt, ha ide beszúrunk egy absztrakciós réteget (interfészt), hogy oldalolvasónkat leválasszuk az aktuális implementációról.

Az 'src' mappánkban hozzunk létre egy `Page` névre hallgató alkönyvtárat, ahol az oldalakkal kapcsolatos osztályokat fogjuk tartani. Ebben a mappában fogjuk elhelyezni a `PageReaderInterface.php` nevű állományunkat, az alábbi tartalommal:

```php
<?php declare(strict_types = 1);

namespace Example\Page;

interface PageReaderInterface
{
  public function readBySlug(string $slug) : string;
}
```

A fenti interfészt az ugyanitt létrehozandó `FilePageReader.php` állományban fogjuk implementálni, ami valahogy így fog kinézni:

```php
<?php declare(strict_types = 1);

namespace Example\Page;

use InvalidArgumentException;

class FilePageReader implements PageReaderInterface
{
  private $pageFolder;

  public function __construct(string $pageFolder)
  {
    $this->pageFolder = $pageFolder;
  }

  public function readBySlug(string $slug) : string
  {
    return 'Én egy helyőrző vagyok.';
  }
}
```

Mint a fenti kódban látható, az oldalakat tartalmazó mappa (pageFolder) elérési útját a konstruktorban kell átadnunk. Ez a megoldás rugalmassá teszi az osztályunkat, mert ha később úgy döntünk, hogy áthelyezzük a fájlokat, vagy esetleg egységtesztet írunk az osztályhoz, könnyen tudunk alkalmazkodni a változásokhoz, a konstruktornak paraméterként átadott elérési út megváltoztatásával.

Az oldalhoz kapcsolódó (általános, nem alkalmazás-specifikus feladatot ellátó) osztályokat saját csomagba is lehet rendezni és különböző alkalmazásokhoz újrahasznosítani. Mivel ezek nem kapcsolódnak szorosan egymáshoz (csak egy absztrakciós rétegen keresztül), ezért könnyen lecserélhetők.

Hozzunk létre egy `Page.html` nevű sablon-fájlt oldalainkhoz a `templates` könyvtárban, majd helyezzük el benne ezt a sort: `{{ content }}`. Ez után az `src/Dependencies.php` állományunkat egészítsük ki az alábbi kódrészlettel, hogy alkalmazásunk tudomására hozzuk, új interfészünk melyik implementációját akarjuk befecskendezni. Itt kell beállítanunk a `pageFolder` értékét is (a már ismert módon, a `define()` metódus használatával).

```php
$injector->alias('Example\Page\PageReaderInterface', 'Example\Page\FilePageReader');
$injector->share('Example\Page\FilePageReader');
$injector->define('Example\Page\FilePageReader', [
  ':pageFolder' => __DIR__ . '/../pages',
]);
```

Ha mindezt megcselekedtük, akkor térjünk vissza a `Page` vezérlőnkhöz és módosítsuk annak `show` metódusát a következőképpen:

```php
public function show($params)
{
  $slug = $params['slug'];
  $data['content'] = $this->pageReader->readBySlug($slug);
  $html = $this->renderer->render('Page', $data);
  $this->response->setContent($html);
}
```

Befejezésként a konstruktoron keresztül be kell fecskendeznünk oldalkezelőnkbe a `Response`, a `Renderer` és a `PageReaderInterface` egy-egy korábban (a `src/Dependencies.php` állományban) beállított implementációját, a `Homepage` vezérlőben alkalmazottak szerint. Ha elkészültünk mindezzel, `Page` vezérlőnk eleje hasonlóan fog festeni:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

use Http\Response;
use Example\Template\RendererInterface;
use Example\Page\PageReaderInterface;
use Example\Page\InvalidPageException;

class Page
{
    private $response;
    private $renderer;
    private $pageReader;

    public function __construct(
      Response $response,
      RendererInterface $renderer,
      PageReaderInterface $pageReader
    ) {
      $this->response = $response;
      $this->renderer = $renderer;
      $this->pageReader = $pageReader;
    }
...
```

Ha eddig minden rendben, akkor most bírjuk `FilePageReader` osztályunkat tényleges munkára. Többek közt képesnek kell lennie arra is, hogy jelezze, ha egy kért oldal esetleg nem található. Ehhez (a beépített `Exception` osztály kibővítésével) létrehozhatunk egy egyéni kivételt, amelyet a hívó kódban majd elkaphatunk.

Az `src/Page` mappában hozzuk is létre az `InvalidPageException.php` állományt a következő tartalommal:

```php
<?php declare(strict_types = 1);

namespace Example\Page;

use Exception;

class InvalidPageException extends Exception
{
  public function __construct($slug, $code = 0, Exception $previous = null)
  {
    $message = "No page with the slug `$slug` was found";
    parent::__construct($message, $code, $previous);
  }
}
```

Majd a `FilePageReader` osztály `readBySlug` metódusát cseréljük le erre:

```php
public function readBySlug(string $slug) : string
{

  $path = "$this->pageFolder/$slug.md";

  if (!file_exists($path)) {
    throw new InvalidPageException($slug);
  }

  return file_get_contents($path);

}
```

Ezután, ha olyan lapra navigálunk, ami történetesen nem létezik, akkor egy `InvalidPageException`-t kell kapnunk, létező lap esetén pedig annak tartalmát kellene látnunk, ha mindent jól csináltunk.

Természetesen annak, hogy az érvénytelen URL-t bepötyögő felhasználóknak különféle `Exception`-okat hajigálunk, nem sok értelme van. Ezért a `Page` vezérlőnkben kapjuk el a `readBySlug` metódus által dobott kivételt, és a felhasználónak már egy 404-es hibaoldalt mutassunk helyette.

Nyissuk meg tehát a `Page` vezérlőt és írjuk újra annak `show` metódusát az alábbiak szerint:

```php
public function show($params)
{
  $slug = $params['slug'];

  try {
    $data['content'] = $this->pageReader->readBySlug($slug);
  } catch (InvalidPageException $e) {
    $this->response->setStatusCode(404);
    return $this->response->setContent('404 - Page not found');
  }

  $html = $this->renderer->render('Page', $data);
  $this->response->setContent($html);
}
```

Győződjünk meg róla, hogy a `use Example\Page\InvalidPageException` utasítás szerepel-e a `Page` vezérlő állomány felső részében, majd próbálkozzunk különféle URL-ek meghívásával, hogy meggyőződjünk róla: művünk helyesen működik. Ha nem, akkor irány bogarászni...

Ahogy mindig, most se felejtsünk el commitolni!

[<< előző fejezet](09-templating.md) | [következő fejezet >>](11-page-menu.md)
