[<< előző fejezet](03-error-handler.md) | [következő fejezet >>](05-router.md)

### HTTP

A PHP rendelkezik jó néhány olyan eszközzel, amelyek könnyebbé teszik a HTTP-vel való munkát. Ilyenek például az előre definiált vagy [szuperglobális](http://php.net/manual/en/language.variables.superglobals.php) asszociatív tömbök, amelyek tartalmazzák a szerver és a kérelem adatait. Ezek a beépített változók bárhonnan elérhetők, a `global` kulcsszó használata nélkül.

#### $_SERVER ####
A szerver és a futtatási környezet változói, fejléceket, útvonalakat és a futó scripre vonatkozó információkat tartalmaz. A `QUERY_STRING` és `REQUEST_URI` kulccsal azonosított elemei az [RFC 3986 ajánlás](http://www.faqs.org/rfcs/rfc3986.html) szerint [url-kódolva](https://www.php.net/manual/en/function.rawurlencode.php) vannak (a kódolást a böngészőnk végzi, a kérelem elküldése előtt). Ez azt jelenti, hogy minden nem ascii karakter (pl. magyar ékezetes betűk) helyén egy százalékjel és egy kétjegyű hexadecimális szám található. E kódolt karakterláncból a PHP [rawurldecode()](https://www.php.net/manual/en/function.rawurldecode.php) függvénye segítségével nyerhetjük ki az eredeti szöveget.
#### $_GET ####
A HTTP GET metódussal érkező adatok (vagyis a kulcs-érték párokra bontott `QUERY_STRING`) lelőhelye. A $_SERVER tömbbel ellentétben az adatok itt már url-dekódolva vannak.
#### $_POST ####
A HTTP POST metódussal érkező, `application/x-www-form-urlencoded` vagy `multipart/form-data` típusú adatokat tárolja.
#### $_REQUEST ####
Alapértelmezés szerint tartalmazza a $_GET, $_POST és a $_COOKIE tömbök tartalmát.
#### $_SESSION ####
Az aktuális script által hozzáférhető, munkamenet-változókat tartalmazza. A [munkamenet-kezelés](http://webprogramozas.inf.elte.hu/tananyag/wf2/lecke17_lap1.html#hiv1) a HTTP protokoll állapotmentességéből fakadó problémákat hivatott orvosolni, lehetővé téve a kliensek megkülönböztetését és a kliensenkénti adattárolást a szerveren.
#### $_COOKIE ####
A [setcookie()](https://www.php.net/manual/en/function.setcookie.php) függvény által beállított sütiket hivatott tárolni.
#### $_FILES ####
A felhasználók által a HTTP POST metódussal feltöltött állományok adatait tartalmazza.

Ezek a változók (kellő óvatosság mellett, mert zömében a felhasználótól származó adatokat tartalmaznak, amelyekben nem bízhatunk) hasznosak lehetnek ha csak egy egyszerűbb scriptet szeretnénk kapni, amit nem túl nehéz karbantartani. Ha azonban [tiszta](https://hup.hu/node/143736), könnyen karbantartható, [SOLID](https://www.refaktor.hu/tiszta-kod-5-resz-a-s-o-l-i-d-alapelvek/) kód a célunk, akkor egy szép, átlátható, objektumorientált felületet kell létrehoznunk.

Ehhez nem kell feltétlenül újra feltalálnunk a kereket (ha nem akarjuk), elég a megfelelő csomagot telepíteni. Ebben az útmutatóban a [PatrickLouys/http](https://github.com/PatrickLouys/http) csomagot fogjuk használni, de ha tudsz jobbat, nyugodtan használd azt. Íme néhány alternatíva: [Symfony HttpFoundation](https://github.com/symfony/HttpFoundation), [Nette HTTP Component](https://github.com/nette/http), [Aura Web](https://github.com/auraphp/Aura.Web), [sabre/http](https://github.com/fruux/sabre-http)

Adjuk is hozzá a kiválasztott HTTP komponenst a `composer.json` függőségeihez, majd futtassuk a `composer update` parancsot:

```json
  "require": {
    "php": ">=7.0.0",
    "filp/whoops": "~2.1",
    "patricklouys/http": "~1.4"
  },
```

Ha ez megvan, akkor az előző fejezetben megírt hibakezelő kód alá írjuk a következőket a `Bootstrap.php` állományba (és ne felejtsük eltávolítani a tesztelési célzattal beszúrt Exception-t):

```php
$request = new \Http\HttpRequest($_GET, $_POST, $_COOKIE, $_FILES, $_SERVER);
$response = new \Http\HttpResponse;
```

Ezzel beállítottuk azokat a `Request` és `Response` objektumokat, amelyeket alkalmazásunk más osztályai használhatnak a kérelem adatainak kinyerésére és a válasz elküldésére a böngészőnek. A válasz elküldéséhez adjuk hozzá a következő kódrészletet a `Bootstrap.php` végéhez:

```php
foreach ($response->getHeaders() as $header) {
    header($header, false);
}

echo $response->getContent();
```

Ez fogja elküldeni válaszunkat a felhasználó böngészőjének. Ha ezt nem tesszük meg, akkor nem fog történni semmi, mert a `Response` objektum csak eltárolja az adatokat és a megfelelő függvény meghívása nélkül esze ágában sem lesz elküldeni őket. Ezt a legtöbb HTTP komponens másképpen kezeli és mintegy mellékhatásként küld adatokat a böngészőnek. Ezt tartsuk szem előtt, ha másik komponenst használunk és a meglepetések elkerülése végett olvassuk el figyelmesen az adott csomag dokumentációját.

A `header()` [függvény](https://www.php.net/manual/en/function.header.php) második paramétere a kódban azért `false`, mert így több azonos típusú (de eltérő tartalmú) fejléc sort is elküldhetünk (ellenkező esetben a függvény felülírja a meglévő fejléceket).

`Response` objektumunk most csupán egy üres választ küld a böngészőnek, az aktuális HTTP státusz kóddal (200). Ha azt szeretnénk, hogy üzenettörzse is legyen a válaszunknak, szúrjuk be a következő kódot a `foreach` kifejezés fölé:

```php
$content = '<h1>Hello World</h1>';
$response->setContent($content);
```

Ha a közkedvelt 404-el szeretnénk megörvendeztetni a klienst, akkor használjuk az alábbi kódot:

```php
$response->setContent('404 - Page not found');
$response->setStatusCode(404);
```

Ne feledjük, hogy az objektumpéldány csak adatokat tárol, így a válasz elküldése előtt több állapotkódot is beállíthatunk rajta, de csak az utolsót fogja elküldeni.

Azon HTTP megoldások, amelyek a [PSR-7 szabványt](https://www.php-fig.org/psr/psr-7/) implementálják, mint a [Slim/Http/Response](https://github.com/slimphp/Slim/blob/3.x/Slim/Http/Response.php), a HTTP objektumokra a PSR által megkövetelt [immutable](https://www.refaktor.hu/megvaltoztathatatlan-objektumok/) tervezési mintát alkalmazzák, amely nem teszi lehetővé a `Response` objektumpéldányunk állapotának megváltoztatását. A PSR-7-ben a setterek helyett (pl. a példánkban látott `setStatusCode()`) olyan metódusok állnak rendelkezésünkre, amelyek a megváltoztatni kívánt tulajdonságot egy másik (klónozott) objektumpéldányon állítják be (ilyen a `Slim/Http/Response::withStatus()` metódusa), míg az eredeti példányt változatlanul hagyják (innen a tervezési minta neve).

A későbbi részekben bemutatjuk, hogy lehet használni az egyes komponensek különböző funkcióit. Részletesebb információkért lapozzunk bele a [dokumentációba](https://github.com/PatrickLouys/http), vagy böngésszük a forráskódot, ha szeretnénk mélyebben megérteni az összetevő működését, nem csak használni azt.

[<< előző fejezet](03-error-handler.md) | [következő fejezet >>](05-router.md)
