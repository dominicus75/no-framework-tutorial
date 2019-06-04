[<< előző fejezet](11-page-menu.md) | [Zárszó >>](to-be-continued.md)


### Frontend

Nem tudom, hogy a jámbor olvasó hogy van vele, de én nem szeretek olyan webhellyel dolgozni, amelyik húsz évesnek néz ki. Tegyük is rendbe alkalmazásunk kinézetét.

Mivel jelen írás nem frontend útmutató, ezért nem fogunk most a HTML5 és a CSS3 bugyraiban elmélyedni, csupán beillesztjük alkalmazásunkba a [pure CSS](http://purecss.io/)-t és azt fogjuk használni (aki a némiképp terjedelmesebb [bootstrapre](https://getbootstrap.com/) esküszik, használhatja azt is).

Először a `Layout.html` sablonfájlt kell az alábbiak szerint módosítanunk.


```php
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Example</title>
    <link rel="stylesheet" href="https://unpkg.com/purecss@1.0.0/build/pure-min.css" integrity="sha384-nn4HPE8lTHyVtfCBi5yW9d20FjT8BJwUXyWZT9InLYax14RDjBj46LmSztkmNP9w"
    crossorigin="anonymous">
    <link rel="stylesheet" href="/css/style.css">
  </head>
  <body>
    <div id="layout">
      <div id="menu">
        <div class="pure-menu">
          <ul class="pure-menu-list">
            {% for item in menuItems %}
              <li class="pure-menu-item"><a href="{{ item['href'] }}" class="pure-menu-link">{{ item['text'] }}</a></li>
            {% endfor %}
          </ul>
        </div>
      </div>
      <div id="main">
        <div class="content">
          {% block content %}
          {% endblock %}
        </div>
      </div>
    </div>
  </body>
</html>
```

Szükségünk lesz némi egyéni CSS-kódra is a PureCSS-en felül. Ennek CSS-fájlnak a web felől elérhetőnek kell lennie, ezért a `public` mappában hozunk létre neki egy `public/css` névre hallgató alkönyvtárat.

Újdonsült `public/css` mappánk első lakója a `style.css` állomány lesz, az alábbi tartalommal:

```css
body {
  color: #777;
}

#layout {
  position: relative;
  padding-left: 0;
}

#layout.active #menu {
  left: 150px;
  width: 150px;
}

#layout.active .menu-link {
  left: 150px;
}

.content {
  margin: 0 auto;
  padding: 0 2em;
  max-width: 800px;
  margin-bottom: 50px;
  line-height: 1.6em;
}

.header {
  margin: 0;
  color: #333;
  text-align: center;
  padding: 2.5em 2em 0;
  border-bottom: 1px solid #eee;
}

.header h1 {
  margin: 0.2em 0;
  font-size: 3em;
  font-weight: 300;
}

.header h2 {
  font-weight: 300;
  color: #ccc;
  padding: 0;
  margin-top: 0;
}

#menu {
  margin-left: -150px;
  width: 150px;
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  z-index: 1000;
  background: #191818;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

#menu a {
  color: #999;
  border: none;
  padding: 0.6em 0 0.6em 0.6em;
}

#menu .pure-menu,
#menu .pure-menu ul {
  border: none;
  background: transparent;
}

#menu .pure-menu ul,
#menu .pure-menu .menu-item-divided {
  border-top: 1px solid #333;
}

#menu .pure-menu li a:hover,
#menu .pure-menu li a:focus {
  background: #333;
}

#menu .pure-menu-selected,
#menu .pure-menu-heading {
  background: #1f8dd6;
}

#menu .pure-menu-selected a {
  color: #fff;
}

#menu .pure-menu-heading {
  font-size: 110%;
  color: #fff;
  margin: 0;
}

.header,
.content {
  padding-left: 2em;
  padding-right: 2em;
}

#layout {
  padding-left: 150px; /* left col width "#menu" */
  left: 0;
}

#menu {
  left: 150px;
}

.menu-link {
  position: fixed;
  left: 150px;
  display: none;
}

#layout.active .menu-link {
  left: 150px;
}

```

Ha most még egy pillantást vetünk a webhelyünkre, valamelyest jobban fog kinézni, mint eddig. Ezen a jövőben még bátran javíthatunk, de most térjünk vissza az útmutatóhoz.

Ha a továbbiakban újabb nyilvánosan elérhető fájlokat (pl. az oldalon megjelenített képek) vagy eszközöket (újabb css vagy Javascript állományok) akarunk hozzáadni az alkalmazásunkhoz, akkor ezeket is a `public` mappában kell elhelyeznünk (mondjuk a `public/images` illetve a `public/js` almappákban).

Eddig minden szép és jó (esetleg még működik is), viszont a látogatóink még nem látják, melyik oldalon is tartózkodnak éppen. Ehhez természetesen több menüelemre lesz szükségünk. Mi most a korábban létrehozott `pages/page-one.md` állományt fogjuk használni szemléltetés gyanánt, de bátran hozzáadhatunk újabb lapokat is.

Ezeket az `ArrayMenuReader` osztályban adhatjuk hozzá a `$data` tömbhöz az alábbiak szerint:

```php
return [
  ['href' => '/', 'text' => 'Nyitólap'],
  ['href' => '/page-one', 'text' => 'Első oldal'],
];
```

[<< előző fejezet](11-page-menu.md) | [Zárszó >>](to-be-continued.md)

