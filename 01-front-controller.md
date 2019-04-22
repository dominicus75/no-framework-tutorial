[következő fejezet >>](02-composer.md)

### Front Controller

Először hozzunk létre egy üres könyvtárat a projektünk gyökérkönyvtárában, mondjuk `public` néven. Tegyünk róla, hogy az Apache `DOCUMENT_ROOT` környezeti változója erre a könyvtárra mutasson, vagyis a web felől csak ezen könyvtár tartalma legyen látható. Ide kerül majd minden olyan dolog, amit a web felül elérhetővé szeretnénk tenni, így a képeket, a felhasználó által feltöltött állományokat, css és js fájlokat tartalmazó alkönyvtárak.

Alkalmazásunk forrásállományait erősen ajánlott a `public` könyvtáron kívül, szintén a projekt gyökérkönyvtárában létrehozott `src` mappában tárolni. Ha [Composert](02-composer.md) használunk, akkor az a `public` és az `src` mellé létre fogja hozni a `vendor` mappát is, ahol a harmadik féltől származó, Composer által kezelt osztályok fognak lakni. Ha a fentiek szerint építjük fel alkalmazásunk könyvtárszerkezetét elérhetjük, hogy forrásállományaink rejtve maradjanak a nem kívánt érdeklődők előtt.

Szükség van ezen felül egy belépési pontra is, ahol az összes beérkező kérelem (request) és kimenő válasz (response) áthalad, ezért a `public` mappában el kell helyezni egy `index.php` állományt is, majd az ugyanitt létrehozott `.htaccess` fájl segítségével minden olyan beérkező kérelmet, ami nem egy létező fájl vagy könyvtár tartalmának elküldését kéri a szervertől, az `index.php`-re kell átirányítani, az alábbiak szerint:

```bash
# Átirányítási szabályok, ha a rewrite modul engedélyezve van
<IfModule mod_rewrite.c>

  RewriteEngine on
  RewriteBase /
  RewriteOptions MaxRedirects=10

  # Minden kérelem átirányítása az index.php-ra
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule ^ index.php [NC,L,QSA]

</IfModule>
```

Ezzel az `index.php` állományt sikeresen ki is neveztük alkalmazásunk [front controllerévé](https://hu.wikipedia.org/wiki/Front_vez%C3%A9rl%C5%91_tervez%C3%A9si_minta), amely egyetlen központosított belépési pontot biztosít a beérkező kérelmek kezeléséhez.

Ha a fentiekkel elkészültünk, a projektünk könyvtárszerkezete valahogy így fog festeni:

```bash
myProject/
├── public/
│   ├── .htaccess
│   ├── index.php
│   ├── css/
│   ├── images/
│   ├── js/
│   └── uploads/
├── src/
└── vendor/
```

Ne feledjük, hogy az `index.php` csak kiindulási pont (most már azt is tudjuk, hogy front controller a becsületes neve), ezért alkalmazásunk logikája nem itt fog lakni. Helyezzük el az állomány elején az alábbi kódot:

```php
<?php declare(strict_types = 1);

require __DIR__ . '/../src/Bootstrap.php';
```

A `__DIR__` egy előre definiált [mágikus konstans](http://php.net/manual/en/language.constants.predefined.php) amely az aktuális könyvtár elérési útját tartalmazza. Használatával biztosíthatjuk, hogy a `require` mindig a tartalmazó fájlhoz (jelen esetben az `index.php`) tartozó relatív elérési utat használja a kért állományok eléréséhez.

A `declare(strict_types = 1);` kód engedélyezi a [szigorú típuskényszerítési módot](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict), amely biztosítja, hogy ha nem a várt típusú paramétereket adjuk át egy függvénynek, akkor a PHP értelmező ne próbálja automatikusan átkonvertálni a kapott értéket, hanem dobjon egy kivételt (Exception). Ez azt jelenti, hogy ha a szigorú mód be van kapcsolva, akkor nem tudunk például egész számot (integer) paraméterként átadni egy olyan metódusnak, amely szöveget (string) vár, mert Exception lesz a jutalmunk.

A `Bootstrap.php` lesz az a fájl, ami összedrótozza az alkalmazásunkat (rendszertöltő). Hamarosan foglalkozni fogunk vele is.

Most navigáljunk az `src` mappába és hozzuk létre a `Bootstrap.php` állományt a következő tartalommal:

```php
<?php declare(strict_types = 1);

echo 'Helló világ!';
```

Most pedig ellenőrizzük, hogy mindent jól csináltunk-e. Nyissuk meg a konzolt/terminált és navigáljunk a `public` mappába (`# cd /path/to/myproject/public`). Itt adjuk ki a `php index.php` parancsot. Ezzel (anélkül, hogy webszervert indítanánk) átadjuk a PHP értelmezőnek `index.php` állományunkat, ami – ha mindent jól csináltunk – lefuttatja azt és kiírja a kimenetre (jelen esetben a terminálra), hogy 'Helló világ!'.

Ha nem azt a kimenetet kapjuk, amire számítottunk ('Helló világ!'), akkor térjünk vissza a megnyitott állományokhoz és keressük meg a hibát (esélyes, hogy elgépeltünk valamit). Ha csupán egy üres képernyő törekvéseink jutalma, ellenőrizzük, hogy telepítve van-e a gépünkre a PHP.

Ha ideáig sikeresen eljutottunk, itt a remek alkalom, hogy megőrizzük az utókornak művünket. Ha még nem használjuk a [Gitet](https://szit.hu/doku.php?id=oktatas:programoz%C3%A1s:verzi%C3%B3kontroll:git), telepítsük sebesen és hozzunk létre egy tárolót a projektünk számára. Jelen írásnak nem célja a Git verziókövető bemutatása, ezért nem megyek bele a részletekbe, az interneten számos kiváló leírás található hozzá, [magyar nyelven](http://math.bme.hu/~balazs/git/) is. Viszont a verziókövetésnek szokásunkká kell válnia, még akkor is, ha csak kisebb bemutató projekteken dolgozunk.

Verziókövető használatakor ne feledjük, hogy néhány szerkesztő és IDE hajlamos rá, hogy saját állományait a projektmappánkban helyezze el. Ennek kivédésére (mivel ezeket a fájlokat nem szeretnénk verziókövetni) hozzunk létre egy `.gitignore` névre hallgató egyszerű szöveges fájlt a projektünk gyökerében és adjuk meg benne azokat a fájlokat és könyvtárakat, amelyeket figyelmen kivül akarunk hagyni a verziókövetésénél. Alább egy példa ([Geany](https://www.geany.org/) használatakor):

```
*.geany
```

[következő fejezet >>](02-composer.md)
