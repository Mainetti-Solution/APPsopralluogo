// Nome della cache
const CACHE_NAME = 'sopralluogo-app-v1';

// Lista degli asset da mettere in cache
const filesToCache = [
  './',
  './index.html',
  './manifest.json',
  './Logo Mainetti Solution con palazzo.PNG',
  './icons/icon-192x192.png',
  './icons/icon-512x512.png',
  'https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js'
];

// Installazione del Service Worker
self.addEventListener('install', event => {
  console.log('Service Worker: Installazione in corso');
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('Service Worker: Cache aperta');
        return cache.addAll(filesToCache);
      })
      .then(() => {
        console.log('Service Worker: Tutti i file sono stati messi in cache');
        return self.skipWaiting();
      })
  );
});

// Attivazione del Service Worker
self.addEventListener('activate', event => {
  console.log('Service Worker: Attivazione in corso');
  
  // Rimuovi le vecchie cache
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== CACHE_NAME) {
            console.log('Service Worker: Cancellazione della vecchia cache', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    }).then(() => {
      console.log('Service Worker: Ora è attivo');
      return self.clients.claim();
    })
  );
});

// Intercetta le richieste di rete
self.addEventListener('fetch', event => {
  console.log('Service Worker: Intercettazione richiesta per', event.request.url);
  
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // Se la risorsa è nella cache, restituiscila
        if (response) {
          console.log('Service Worker: Risorsa trovata in cache per', event.request.url);
          return response;
        }
        
        // Altrimenti, fai la richiesta alla rete
        console.log('Service Worker: Risorsa non trovata in cache, richiesta alla rete per', event.request.url);
        return fetch(event.request)
          .then(networkResponse => {
            // Se la risposta non è valida, restituiscila semplicemente
            if (!networkResponse || networkResponse.status !== 200 || networkResponse.type !== 'basic') {
              return networkResponse;
            }
            
            // Fai una copia della risposta
            const responseToCache = networkResponse.clone();
            
            // Aggiungi la risposta alla cache
            caches.open(CACHE_NAME)
              .then(cache => {
                cache.put(event.request, responseToCache);
              });
            
            return networkResponse;
          });
      })
      .catch(error => {
        console.log('Service Worker: Errore nel recupero della risorsa', error);
        // Qui potresti fornire una pagina di fallback
      })
  );
});