# Član 3 — Qdrant Docker/Cloud i instalacija

Ovaj deo postavlja **samo infrastrukturu**. Ne pravi kolekciju i ne uvozi vektore, jer to pripada članu 4.

## Šta je postavljeno

- Qdrant Docker Compose servis, sa trajnim named volume-om `qdrant-image-search-storage`;
- REST port `6333` i gRPC port `6334`;
- lokalno vezivanje na `127.0.0.1`, pa baza nije dostupna drugim uređajima u mreži;
- `.env.example` kao jedinstveno mesto za URL, ime kolekcije i tehnički ugovor sa sledećim članovima;
- dve dijagnostičke skripte koje **ne menjaju bazu**.

## Preduslov na Windows-u

1. Instalirati Docker Desktop.
2. Pokrenuti Docker Desktop i sačekati da status bude **Engine running**.
3. U PowerShell-u, iz korena ovog foldera, proveriti:

```powershell
docker --version
docker compose version
```

Ako druga komanda javlja da `docker` nije prepoznat, Docker Desktop nije pokrenut ili nije uspešno instaliran.

## Lokalno pokretanje — obavezna varijanta

```powershell
Copy-Item .env.example .env
docker compose up -d
docker compose ps
py scripts\wait_for_qdrant.py
py scripts\check_qdrant_connection.py
```

Očekivan rezultat poslednje komande je:

```text
Qdrant connection OK
URL: http://localhost:6333
Collections: 0
```

`Collections: 0` je normalno pre rada člana 4.

### Gde se proverava da Qdrant radi

- Qdrant dashboard: `http://localhost:6333/dashboard`
- REST API lista kolekcija: `http://localhost:6333/collections`
- status kontejnera: `docker compose ps`
- poslednji logovi: `docker compose logs --tail=100 qdrant`

## Gašenje i trajnost podataka

```powershell
# Zaustavlja servis, ali čuva Qdrant podatke.
docker compose down

# Ponovo startuje servis sa istim podacima.
docker compose up -d
```

Nemoj koristiti `docker compose down -v` osim ako tim namerno želi potpuno praznu bazu: ta komanda briše named volume, a sa njim i sve Qdrant kolekcije/podatke.

## Cloud varijanta — samo ako se tim kasnije odluči za Qdrant Cloud

Docker lokalna varijanta je dovoljna za osnovni projekat. Cloud ne pokretati paralelno bez jasnog dogovora, jer se tada lako pomešaju lokalni i udaljeni podaci.

1. Napraviti cluster i Database API key u Qdrant Cloud Console.
2. U `.env` **zameniti** lokalne vrednosti:

```dotenv
QDRANT_URL=https://<cluster-url>
QDRANT_API_KEY=<database-api-key>
```

3. Ne pokretati `docker compose up` za cloud test.
4. Pokrenuti:

```powershell
py scripts\check_qdrant_connection.py
```

`.env` je u `.gitignore`; API key se nikad ne commit-uje i ne šalje kroz GitHub chat ili screenshot.

## Interfejs prema članu 4 — uvoz u kolekciju

Član 4 preuzima sledeći dogovor:

| Stavka | Dogovorena vrednost |
|---|---|
| Qdrant URL | `QDRANT_URL`, lokalno `http://localhost:6333` |
| Ime kolekcije | `stl10_clip_images` |
| Veličina vektora | `512` |
| Metoda distance | `Cosine` |
| ID poena | `id` iz `embeddings_metadata.csv` |
| Payload | `id`, `image_path`, `label` |
| Vektor | `embeddings[int(row["embedding_index"])]` samo za `status == "ok"` |

Član 2 je pripremio 1.000 L2-normalizovanih CLIP embeddinga dimenzije 512, tako da kolekcija mora biti napravljena kao `VectorParams(size=512, distance=Distance.COSINE)`.

## Predaja člana 3 / dokaz na demonstraciji

Na demonstraciji pokazati redom:

```powershell
docker compose ps
py scripts\check_qdrant_connection.py
```

Zatim otvoriti `http://localhost:6333/dashboard`. Time je dokazano da je Qdrant servis podignut, REST port radi i baza je spremna da član 4 ubaci kolekciju.

## Česte greške

- **`Cannot reach Qdrant`**: Docker Desktop nije pokrenut, kontejner je stao ili je port zauzet. Pogledati `docker compose ps` i `docker compose logs qdrant`.
- **Port 6333 is already allocated**: promeniti `QDRANT_HTTP_PORT` u `.env`, na primer na `6335`, i ažurirati `QDRANT_URL=http://localhost:6335`.
- **Qdrant rejected the request**: kod Cloud varijante je pogrešan/nedostaje `QDRANT_API_KEY`.
- **Nestali podaci posle restarta**: verovatno je pokrenuto `docker compose down -v`; time se namerno briše volume.
