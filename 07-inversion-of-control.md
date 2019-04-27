[<< előző fejezet](06-dispatching-to-a-class.md) | [következő fejezet >>](08-dependency-injector.md)

### A vezérlés megfordítása

Az előző fejezetben létrehoztunk egy vezérlő osztályt, majd majd az `echo` parancs segítségével generáltunk egy sokatmondó "Hello world" kimenetet. Ne feledjük azonban, hogy van egy szép, objektumorientált HTTP absztrakciónk is, amit most még nem tudunk elérni a vezérlő osztályunkból.

Az észszerű megoldás erre a [vezérlés megfordítása](https://hu.wikipedia.org/wiki/A_kontroll_megford%C3%ADt%C3%A1sa). Ez esetünkben azt jelenti, hogy az osztályunknak csak kérnie kell azokat az objektumokat, amelyekre szüksége van, ahelyett, hogy létrehozná őket. Ezt [függőség befecskendezéssel (dependency injection)](https://www.refaktor.hu/tiszta-kod-7-resz-mi-a-fene-az-a-dependency-injection-container/) tudjuk megvalósítani.

Ha elsőre egy kicsit bonyolultnak hangzik is, ez ne szegje kedvünket. Csak kövessük az útmutatót lépésről lépésre, és a végére minden értelmet nyer.

Először módosítsuk `Homepage` vezérlőnket a következőképpen:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

use Http\Response;

class Homepage
{
    private $response;

    public function __construct(Response $response)
    {
        $this->response = $response;
    }

    public function show()
    {
        $this->response->setContent('Hello World');
    }
}
```

Figyeljük meg, hogy a fájl elején (csakis az osztálydefiníció előtt, a fájl globális területén, különben hibát dob) a `use` kulcsszó használatával [importáltuk](http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=6#section_3) a `Http\Response` osztályt. Ez azt jelenti, hogy ettől fogva bármikor használhatjuk ebben az állományban. Ezen felül a `use` feloldja a teljesen minősített osztálynevet, így azt már – ahogy a példakódban is látható – nem kell teljesen kiírnunk, elég a `Response` (az osztálynév). Ha az állományunkban később másképp akarunk rá hivatkozni, megadhatunk álnevet (alias) is az `as` kulcsszó használatával, valahogy így:

```php
use Http\Response as AnotherName;
```

Vezérlőnk konstruktora most [kifejezetten megkövetel (type hinting)](https://it-sziget.hu/php-jelentosebb-valtozasai-napjainkig-kezdetektol-a-trait-ekig#n254-php5-oop-typehint) egy `Http\Response`-típusú bemenetet. Esetünkben a kért `Http\Response` nem egy konkrét osztály (implementáció), hanem a [függőség megfordítás elvének](https://reiteristvan.wordpress.com/2011/09/17/s-o-l-i-d-objektum-orientlt-tervezsi-elvek-5-dip/) megfelelően egy [interfész](http://nyelvek.inf.elte.hu/leirasok/PHP/index.php?chapter=10#section_3_14), vagyis absztrakció. Így nem vagyunk egy konkrét `Http\Response` osztályhoz kötve, mivel vezérlőnk minden olyan bemenetet elfogad, ami megvalósítja a type hintingben megadott interfészt.

Kódunk azonban jelen állapotában hibával fog elszállni, mivel még nem fecskendeztük bele a szükséges függőséget. Javítsuk tehát a `Bootstrap.php` azon részét, ami a létező útvonalra vonatkozó átirányításért felelős. A következő kódrészt:

```php
$class = new $className;
$class->$method($vars);
```

cseréljük le erre:

```php
$class = new $className($response);
$class->$method($vars);
```

A `Http\HttpResponse` objektum megvalósítja a `Http\Response` interfészt, így teljesíti a `Homepage` vezérlőnk konstruktorában támasztott követelményt, vagyis felhasználható. A kódunknak ezért működnie kell.

Ha követjük a fenti példát, minden objektumunk amit így példányosítunk ugyanazt a függőséget (jelen esetben a `Http\HttpResponse` objektumot) fogja megkapni. Ezt természetesen nem fogjuk így hagyni, a következő fejezetben erre keresünk majd megoldást.

[<< előző fejezet](06-dispatching-to-a-class.md) | [következő fejezet >>](08-dependency-injector.md)
