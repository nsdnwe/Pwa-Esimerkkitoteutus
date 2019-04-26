# Pwa - Progressive Web Apps - Perusteet
Tässä esimerkissä on käytetty editorina VS2015, mutta toteutus on sama IDE:stä riippumatta.
Alla kuvattu esimerkki PWA applikaatio löytyy kokonaisuudessaan täältä:

Toteutus vaiheittain:
-	Tee uusi web site
  -	Valite VS:n menusta: File, New Web Site
- Lisää projektiin index.html
- Lisää projektiin styles.css
  - Tiedoston ei tarvitse sisältää mitään tietoa, se on vain esimerkkinä myöhempää käyttöä varten
-	Lisää projektiin manifest.json ja sen sisällöksi esim.
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
-	manifest.json sekä eri kokoiset ikonit voi generoida esim. tällä generaattorilla https://app-manifest.firebaseapp.com/ 
  -	Vain 192x192px ja 512x512px kokoiset ikonit ovat pakollisia
-	Lisätietoja manifest.json parametreistä ym. löytyy täältä https://developers.google.com/web/fundamentals/web-app-manifest/
-	Lisää viittaus manifest.json tiedostoon index.html:n
```link rel="manifest" href="/manifest.json">```
-	Lisää projektiin uusi javascript-tiedosto sw.js
-	Lisää sw.js sisällöksi
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
-	Esimerkki sw.js cachettaa halutut tiedostot offline käyttöä varten. Tiedostot valitaan listassa sw.js tiedoston alussa.
-	Lisätietoja Service Worker kautta toteutettavista toiminnoista ja parametreista löytyy täältä https://serviceworke.rs/
-	Lisää index.html Service Worker:in kutsu, jonka jälkeen index.html näyttää tältä
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

