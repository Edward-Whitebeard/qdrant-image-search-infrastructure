# Qdrant infrastruktura — član 3

Ovo je završeni deo **Člana 3: Qdrant Docker/Cloud + instalacija** za projekat pretrage sličnih STL-10 slika.

Paket nastavlja na već pripremljene ulaze:

- Član 1: STL-10 subset, 1.000 slika, `metadata.csv`;
- Član 2: CLIP `openai/clip-vit-base-patch32`, 1.000 L2-normalizovanih embeddinga `(1000, 512)`;
- Qdrant kolekcija koju će napraviti član 4 mora imati `size=512` i `Cosine` distance.

## Brzo pokretanje (Windows PowerShell)

```powershell
Copy-Item .env.example .env
docker compose up -d
py scripts\wait_for_qdrant.py
py scripts\check_qdrant_connection.py
```

Zatim otvoriti Qdrant dashboard: `http://localhost:6333/dashboard`.

## Šta se ovim dobija

- Qdrant `v1.18.2` u Docker kontejneru;
- REST API na `localhost:6333`;
- gRPC na `localhost:6334`;
- trajni Docker volume `qdrant-image-search-storage`;
- lokalno ograničena mrežna dostupnost (`127.0.0.1`);
- spreman `.env` ugovor za lokalni ili Cloud Qdrant.

Detaljno uputstvo je u [docs/member3_setup.md](docs/member3_setup.md), a precizan ugovor prema sledećim članovima u [docs/integration_contract.md](docs/integration_contract.md).

## Bitna granica odgovornosti

Ovaj deo **ne kreira kolekciju i ne uvozi vektore**. To je za člana 4, da import bude čist, ponovljiv i lako dokaziv.
