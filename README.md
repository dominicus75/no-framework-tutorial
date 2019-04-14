Az alábbi útmutató kedvező fogadtatása arra ösztönözte a [szerzőt](https://github.com/PatrickLouys), hogy könyv formában is közreadja művét. A könyv a jelen útmutató javított és bővített változata, amely az alábbi linkre kattintva megrendelhető (angol nyelven).

### [Professzionális PHP: karbantartható és biztonságos alkalmazások építése](http://patricklouys.com/professional-php/)

![](http://patricklouys.com/img/professional-php-thumb.png){:target="_blank"}

Az útmutató eredeti formájában alább olvasható.

## PHP alkalmazás létrehozása keretrendszer nélkül

### Bevezetés

Ez az útmutató azoknak készült, akik már elsajátították a PHP nyelvet és tisztában vannak az objektum-orientált programozás alapjaival.

Ha még nem hallottál a [SOLID alapelvekről](https://www.refaktor.hu/tiszta-kod-5-resz-a-s-o-l-i-d-alapelvek/){:target="_blank"}, akkor a jelen sorok elolvasása előtt hasznos lenne megismerkedni velük.

Számtalan emberrel találkoztam már a Stack Overflow PHP csevegőszobájában, akik a felől érdeklődtek, hogy melyik keretrendszer lenne számukra a legjobb. A legtöbb esetben a hozzáértők válasza az volt, hogy leginkább egyik sem. Ha nem muszáj, ne [erőltessék a keretrendszereket](http://legyes.hu/hu/post/miert-nem-hasznalok-php-frameworkot/){:target="_blank"}, inkább natív PHP-t használjanak alkalmazásuk felépítéséhez. Sokan azonban nem tudják, hogy is fogjanak hozzá.

Az következő útmutató célja, hogy nekik nyújtson egyszerű, áttekinthető segítséget. A legtöbb esetben a keretrendszerek használatának nincs sok értelme és az alkalmazás nulláról (esetleg harmadik féltől származó csomagok használatával) való felépítése sokkal egyszerűbb, mint azt sokan gondolnák.

**Ez az útmutató a PHP 7.0 vagy újabb változataihoz készült.** Ha régebbi verziót használsz, nem árt frissíteni a [legújabb stabil verzióra](http://php.net/downloads.php){:target="_blank"}, mielőtt hozzáfogsz.

Ha minden készen áll, vágjunk is bele az [első részbe](01-front-controller.md)!

### Tartalomjegyzék

1. [Front Controller](01-front-controller.md){:target="_blank"}
2. [Composer](02-composer.md){:target="_blank"}
3. [Hibakezelés](03-error-handler.md){:target="_blank"}
4. [HTTP](04-http.md){:target="_blank"}
5. [Router](05-router.md){:target="_blank"}
6. [Dispatcher](06-dispatching-to-a-class.md){:target="_blank"}
7. [A vezérlés megfordítása](07-inversion-of-control.md){:target="_blank"}
8. [Függőség befecskendezés](08-dependency-injector.md){:target="_blank"}
9. [Sablonok](09-templating.md){:target="_blank"}
10. [Dinamikus oldalak](10-dynamic-pages.md){:target="_blank"}
11. [Oldalmenü](11-page-menu.md){:target="_blank"}
12. [Frontend](12-frontend.md){:target="_blank"}
