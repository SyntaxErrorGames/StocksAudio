# StocksAudio

Sito di ascolto pubblico: chi apre il link entra e ascolta. Nessun account, nessun login.
Le canzoni sono link SoundCloud (anche privati) elencati in un file JSON.

Stack: **Expo Router + React Native Web** per il sito · **Supabase Edge Function** per il catalogo ·
**Widget API ufficiale di SoundCloud** per la riproduzione.

---

## Aggiungere una canzone

1. Apri [data/tracks.json](data/tracks.json) e aggiungi una voce:

```json
{
  "id": "nome-univoco-senza-spazi",
  "title": "Titolo del brano",
  "artist": "Nome artista",
  "tag": "remix",
  "originalArtist": "Artista originale",
  "genre": "House",
  "year": 2026,
  "artwork": null,
  "soundcloud": "https://soundcloud.com/tuo-profilo/traccia/s-XXXXXXXXXXX"
}
```

2. Pubblica il catalogo:

```bash
npm run catalog
```

`tag` può essere `original_mix`, `remix` (richiede `originalArtist`) o `bootleg`.
`artwork` è un URL immagine, oppure `null` per usare quella di SoundCloud.
Per una traccia **privata** incolla il link segreto completo, quello che finisce con `/s-xxxxxxxx`.

Lo script controlla il file prima di pubblicarlo: id duplicati, campi mancanti, link non validi e
remix senza artista originale fanno fallire il comando con un messaggio preciso.

---

## Quanto sono protetti i link — leggi questo

Il catalogo con i link **non sta nel sito**: vive dentro la Edge Function. Il browser riceve
titolo, artista e copertina; l'indirizzo della traccia lo chiede solo quando premi play, uno alla
volta. In pagina non compare nessun URL, non c'è nessun pulsante verso SoundCloud, e la risposta
del server non viene mai messa in cache.

**Però**: per suonare, quell'indirizzo deve arrivare al browser. Chi apre gli strumenti di
sviluppo (F12 → Rete) può leggerlo. Nessun accorgimento lato sito può evitarlo — nasconderlo
davvero richiederebbe di far passare lo stream da un proxy nostro, cosa che i termini di servizio
di SoundCloud vietano e che questo progetto non fa.

In pratica: un utente normale non troverà mai il link, uno tecnico sì. Se una traccia non deve
assolutamente circolare, non metterla su SoundCloud.

---

## Sviluppo

```bash
npm install
cp .env.example .env     # metti l'URL della tua Edge Function
npm run web              # http://localhost:8081
```

| Comando | Cosa fa |
|---|---|
| `npm run catalog` | valida `data/tracks.json`, lo compila e lo pubblica |
| `npm run catalog:build` | solo validazione + compilazione, senza pubblicare |
| `npm run typecheck` | controllo dei tipi |
| `npm run build:web` | build statico in `dist/` |
| `npm run deploy:web` | build e deploy su Vercel |

## Pubblicare il sito

```bash
npx vercel --prod
```

[vercel.json](vercel.json) è già configurato (build, cartella `dist`, rewrite di tutte le rotte su
`index.html`, header di sicurezza). Nelle variabili d'ambiente del progetto Vercel serve solo:

```
EXPO_PUBLIC_CATALOG_URL=https://<project-ref>.supabase.co/functions/v1/catalog
```

E nei secret della funzione, per limitare chi può interrogare il catalogo:

```bash
npx supabase secrets set SITE_ORIGIN=https://il-tuo-dominio.com
```

## Com'è fatto

| Pezzo | Dove |
|---|---|
| Catalogo che modifichi tu | `data/tracks.json` |
| Validazione e compilazione | `scripts/build-catalog.mjs` |
| Server che serve il catalogo e i link | `supabase/functions/catalog/` |
| Motore audio (iframe SoundCloud nascosto) | `src/features/player/widget.ts` |
| Coda, casuale, ripeti | `src/features/player/PlayerProvider.tsx` |
| Barra di riproduzione persistente | `src/components/PlayerBar.tsx` |
| Pagina unica con ricerca e filtri | `app/index.tsx` |

La riproduzione funziona **sul web**. Su iOS e Android l'app mostra un avviso: il widget di
SoundCloud è un iframe, e per farlo girare in un'app nativa servirebbe `react-native-webview`.

> La precedente piattaforma privata (account su invito, pannello admin, upload dei file, social e
> playlist) è conservata nel primo commit git: `git show f4df342 --stat`.
