# PWA Progressive Web Apps - Perusteet

PWA on tapa toteuttaa Android/iOS mobiili-sovellusten tyyppisiä sovelluksia yksinkertaisesti ja nopeasti HTML ja JavaScript teknologioilla ilman ylimääräisiä kirjastoja tai framework:eja.

PWA sovellukset eivät ole käännettyjä natiiveja sovelluksia vaan tavallisia web-sivustoja eli web app:eja, joita suoritetaan mobiililaitteen selaimessa. Tämän lisäksi:
-	PWA sovellukset näyttävät samalta kuin natiivit mobiili-applikaatiot.
-	PWA sovellukset tukevat offline käyttöä sekä Push-notifikaatioita.
-	PWA sovellukset ovat asennettavissa ja käynnistettävissä puhelimesta samalla tavoin kuin natiivi-applikaatiot eli ne sisältävät käynnistysikonit sekä käynnistyksen aikaisen splash-näytön.

PWA tuotantoasennus ei vaadi mitään erityistä verrattuna normaaliin web-app asennukseen eli esim. Azure tai AWS asennus toimii perusasetuksilla.

PWA sovelluksia voi nyt myydä myös Google Play:n kautta. Mitään apk-tiedostoa ei kuitenkaan asenneta, vaan ainoastaan linkki PWA web-app sivulle + lisenssikoodi.

Jotta mobiililaitteen selain hyväksyy PWA sovelluksen, sen tulee täyttää tietyt kriteerit. Näitä ovat:
-	Sivujen tulee olla responsiivisia eli skaalautua päätelaitteen näytön mukaan.
-	Service worker teknologian avulla sovelluksen tulee tukea offline tilaa.
-	SSL tulee olla käytössä kaikessa tietoliikenteessä.

Tarkempi kuvaus PWA:sta löytyy esim. näiltä kahdelta sivulta:
- https://koodiystava.fi/pwa-suuri-harppaus-web-sovelluksille-170d35bb8d7e
- https://developers.google.com/web/progressive-web-apps/

## Yksinkertaisen PWA sovelluksen toteutus vaiheittain

Alla kuvatussa esimerkissä on käytetty läpi vaiheittain miten yksinkertainen PWA web-sivusto toteutetaan. 

Editorina on VS Code, johon on asennettuna [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) lisäosa. IDE-valinnalla ei  ole  merkitystä PWA järjestelmän toteutuksessa, olennaista on että IDE sopii hyvin web-sovellusten kehitykseen. 

- Luo jokin tyhjä kansio ja avaa VS Code tässä kansiossa, tai käynnistä VS Code ja valitse kyseinen kansio.
- Lisää projektiin tyhjä `index.html`.
- Lisää projektiin tyhjä `styles.css`.
- Lisää projektiin tyhjä `manifest.json`.
- Lisää projektiin tyhjä `sw.js`.
- Lisää `manifest.json` sisällöksi esim.
```
{
  "name": "PWA Sample Project",
  "short_name": "PWA-Sample",
  "description": "This is my PWA Sample Project",
  "Scope": "/",
  "start_url": "/",
  "splash_pages": null,
  "theme_color": "#2196f3",
  "background_color": "#2196f3",
  "display": "standalone",
  "icons": [
    {
      "src": "/images/logo-192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "/images/logo-512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ]
}  
```

- Lisää `images` kansio ja sinne kolme esimerkki-kuvaa:
  - logo-192.png
  - logo-512.png
  - favicon.ico
- Kuvien tulee olla 192x192px ja 512x512px kokoisia. 
- `favicon.ico` tiedosto voi olla jokin esimerkki favicon, joita löytyy paljon Googlen avulla.

- **Huom!** `manifest.json` tiedoston sekä eri kokoiset ikonit voi generoida esim. tällä generaattorilla https://app-manifest.firebaseapp.com/ 
- Lisätietoja `manifest.json` tiedoston parametreistä ym. löytyy täältä https://developers.google.com/web/fundamentals/web-app-manifest/

Kansiorakenteen tulisi nyt näyttää tältä:

<img src="https://nsdwww.azurewebsites.net/github-images/image000.png" width="50%">

- Lisää `index.html`:n viittaus `manifest.json` tiedostoon.
```
<link rel="manifest" href="/manifest.json">
```
- Lisää Service Worker esimerkkikoodi `sw.js` tiedoston sisällöksi:

```
// Sample to cache files for off-line use

var cacheName = 'PWA-SAMPLE-CACHE';

// Add below files to cache
var filesToCache = [
    '/',                // index.html
    '/sw.js',
    '/styles.css'
];

self.addEventListener('install', function (event) {
    event.waitUntil(
        caches.open(cacheName)
        .then(function (cache) {
            console.info('[sw.js] cached all files');
            return cache.addAll(filesToCache);
        })
    );
});

self.addEventListener('fetch', function (event) {
    event.respondWith(
        caches.match(event.request)
        .then(function (response) {
            if (response) {
                return response
            }
            else {
                // clone request stream
                // as stream once consumed, can not be used again
                var reqCopy = event.request.clone();
                return fetch(reqCopy, { credentials: 'include' }) // reqCopy stream consumed
                .then(function (response) {
                    // bad response
                    // response.type !== 'basic' means third party origin request
                    if (!response || response.status !== 200 || response.type !== 'basic') {
                        return response; // response stream consumed
                    }
                    // clone response stream
                    // as stream once consumed, can not be used again
                    var resCopy = response.clone();
                    // ================== IN BACKGROUND ===================== //
                    // add response to cache and return response
                    caches.open(cacheName)
                    .then(function (cache) {
                        return cache.put(reqCopy, resCopy); // reqCopy, resCopy streams consumed
                    });
                    return response; // response stream consumed
                })
            }
        })
    );
});
```
- Yllä kuvattu  `sw.js` cachettaa halutut tiedostot offline käyttöä varten. Cachetettava tiedostot määritetään `sw.js` tiedoston alussa olevalla listalla.
- Lisätietoja Service Worker avulla toteutettavista toiminnoista ja parametreista löytyy täältä https://serviceworke.rs/
- Lisää `index.html` tiedostoon vielä Service Worker:in (`sw.js`) kutsu, jonka jälkeen `index.html` näyttää tältä:
```
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" />
    <meta charset="utf-8" />
    <link rel="manifest" href="/manifest.json">
    <link rel="icon" type="image/png" href="/images/favicon.ico">
    <link href="styles.css" rel="stylesheet" />
    <meta name="theme-color" content="#317EFB" />
</head>
<body>
    <h1>PWA Sample page 1</h1>
    <script>
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker
                .register('/sw.js')
                .then(function () { console.log('Service Worker Registered'); });
        }
    </script>
</body>
</html>
```

Yllä kuvattu koodi löytyy kokonaisuudessaan täältä: https://github.com/nsdnwe/Pwa-Sample

### Huomioitavaa

- Varsinainen PWA sovellus tulee olla responsiivinen, joten mukaan kannattaa ottaa lisäksi esim. [Bootstrap kirjastot](https://getbootstrap.com/).

## PWA sovelluksen testaus Chromessa

- Avaa Chrome Dev Tools eli paina F12.
- Avaa Applications-välilehti. Mikäli valinta ei ole näkyvillä, se löytyy >> valinnan alta.
- Manifest välilehti näyttää perustiedot ja virheilmoituksia, mikäli jokin määritys ei ole kunnossa `manifest.json` tiedostossa.

<img src="https://nsdwww.azurewebsites.net/github-images/image001.png" width="60%" />

- Service Workers välilehti näyttää listan mahdollisista virheistä liittyen Service Worker:iin.

<img src="https://nsdwww.azurewebsites.net/github-images/image003.png" width="60%" />

- Update on reload rasti kannattaa laittaa päälle, jotta Service Worker päivittyy koodiin tehtyjen muutosten jälkeen.
- Offline tilaa voi testata vaihtamalla Offline ja Update on reload rastit.

<img src="https://nsdwww.azurewebsites.net/github-images/image005.png" width="60%" />
 
## Muutosten päivittyminen

Mikäli tuntuu siltä, että `index.html` ym. tiedostoihin tehdyt muutokset eivät päivity sivulle:
- Valitse Clear storage välilehti ja paina Clear site data painiketta.

<img src="https://nsdwww.azurewebsites.net/github-images/image007.png" width="60%" />

## PWA sovelluksen Lighthouse testaus

PWA testauksessa suosittelen käyttämään Chrome lisäosaa: Google Lighthouse, jonka voi asentaa [täältä](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=en).

Lighthouse:n käyttö tapahtuu seuraavasti:
- Käynnistä Lighthouse Chromen toolbar:ista.
- Valitse Generate report.

<img src="https://nsdwww.azurewebsites.net/github-images/image009.png" width="40%" />
 
- Avaa raportin Progressive Web App osio.

<img src="https://nsdwww.azurewebsites.net/github-images/image011.png" width="60%" />
 
- Jotta HTTP => HTTPS redirect virheilmoitusta ei tule, täytyy tuotantoympäristössä eli esim. Azure Web App määrityksistä määrittää HTTPS redirect pakolliseksi.
- Virheilmoitusten oikeassa yläkulmassa olevasta nuolesta löytyy suositus, millä virheen voi korjata sekä lisätietoja Learn more linkistä.
