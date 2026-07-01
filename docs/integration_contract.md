# Usklađivanje celog projekta

## Stanje pre člana 3

1. **Član 1**: STL-10 podskup od 1.000 slika; `data/metadata.csv` daje `id`, `image_path`, `label`.
2. **Član 2**: `openai/clip-vit-base-patch32` pravi 1.000 L2-normalizovanih embeddinga; `embeddings.npy` je oblika `(1000, 512)`; `embeddings_metadata.csv` daje vezu između slike i `embedding_index`.

## Stanje posle člana 3

Qdrant je lokalno dostupan i spreman za sledeći korak. Član 3 namerno ne pravi kolekciju ili testne podatke, kako bi član 4 imao čist, ponovljiv import.

## Redosled spajanja repozitorijuma

Najčistiji završni repozitorijum treba da izgleda ovako:

```text
qdrant-image-project/
├── data/
│   ├── images/stl10/                 # lokalno generisano, nije na Git-u
│   ├── metadata.csv
│   └── embeddings/
│       ├── embeddings.npy
│       ├── embeddings_metadata.csv
│       └── embedding_config.json
├── src/
│   ├── generate_embeddings.py
│   ├── check_embeddings.py
│   └── ...
├── infra/
│   ├── docker-compose.yml
│   ├── .env.example
│   └── scripts/
├── scripts/
│   └── prepare_dataset.py
├── docs/
└── requirements.txt
```

Za početak je dovoljno da član 4 klonira repo člana 2, pa doda `infra/` iz ovog paketa. Ako se odlučite za jedan potpuno nov zajednički repo, prvo prebaciti sadržaj člana 1, zatim člana 2, pa `infra/` iz člana 3. Ne duplirati `data/metadata.csv` na dva mesta.

## Šta nije završeno ovim paketom

- kreiranje kolekcije;
- batch import 1.000 tačaka;
- payload index; 
- similarity search i UI/demonstracija rezultata.

To ostaje članovima 4–6 prema raspodeli posla.
