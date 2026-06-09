# DSA Search Engine

A high-performance academic search engine built on the **CORD-19 dataset** (COVID-19 Open Research Dataset). The project combines a C++ core for indexing and retrieval with a Node.js/Express web frontend, enabling fast keyword-based and semantic search over research papers.

---

## Features

- **Semantic Search** — Uses GloVe word embeddings (300D) and cosine similarity to find conceptually relevant documents, not just keyword matches
- **Keyword Search** — AND logic with OR fallback, ranked by term frequency using an inverted index
- **Auto-Complete** — Trie-based prefix suggestions powered by the lexicon
- **Lemmatization** — Normalizes query terms for better recall
- **Text Processing** — Tokenization, stop-word removal, and normalization pipeline
- **Web Interface** — Node.js/Express server bridges the browser UI to the C++ backend via stdin/stdout JSON protocol
- **Fast Loading** — GloVe and document embeddings stored in binary format for quick startup

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│               Browser (Web UI)                  │
└────────────────────┬────────────────────────────┘
                     │ HTTP (GET /api/search, /api/autocomplete)
┌────────────────────▼────────────────────────────┐
│           Node.js / Express Server              │
│               (web/server.js)                   │
└────────────────────┬────────────────────────────┘
                     │ stdin/stdout (JSON protocol)
┌────────────────────▼────────────────────────────┐
│          C++ Search Server (main_server)         │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │  Lexicon │  │ Inverted │  │ Semantic      │  │
│  │          │  │  Index   │  │ Search(GloVe) │  │
│  └──────────┘  └──────────┘  └───────────────┘  │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ Forward  │  │  Auto-   │  │  Lemmatizer   │  │
│  │  Index   │  │ Complete │  │               │  │
│  └──────────┘  └──────────┘  └───────────────┘  │
└─────────────────────────────────────────────────┘
```

---

## Project Structure

```
DSA_search_engine/
├── include/                    # Header files
│   ├── auto_complete.hpp
│   ├── forward_index.hpp
│   ├── inverted_index.hpp
│   ├── lemmatizer.hpp
│   ├── lexicon.hpp
│   ├── MetaDataParser.hpp
│   ├── searching.hpp
│   ├── semantic_search.hpp
│   ├── text_processing.hpp
│   └── nlohmann_json.hpp       # JSON library
├── src/                        # Source files
│   ├── main.cpp                # Interactive terminal mode
│   ├── main_server.cpp         # Server mode (used by web frontend)
│   ├── searching.cpp
│   ├── sematic_search.cpp
│   ├── auto_complete.cpp
│   ├── forward_index.cpp
│   ├── inverted_index.cpp
│   ├── lemmatizer.cpp
│   ├── lexicon.cpp
│   ├── text-processing.cpp
│   └── MetaDataParser.cpp
├── web/                        # Web frontend
│   ├── server.js               # Express server + C++ bridge
│   ├── public/                 # Static frontend files
│   └── package.json
├── sample_data_subset/         # Sample CORD-19 data
│   ├── metadata.csv
│   └── sample_json_files/
└── CMakeLists.txt
```

---

## Prerequisites

### C++ Backend
- CMake 3.20+
- GCC/G++ with C++17 support

### Web Frontend
- Node.js (v14+)
- npm

### Data Files (not included in repo — large files)
You will need to download and place the following in your data directory:

| File | Description |
|------|-------------|
| `lemmatization-en.txt` | English lemmatization dictionary |
| `indices/lexicon.csv` | Built lexicon |
| `indices/forward_index.txt` | Forward index |
| `indices/inverted_index` | Inverted index |
| `data/2020-04-10/metadata.csv` | CORD-19 metadata |
| `embedding/glove_embeddings.bin` | GloVe binary embeddings |
| `embedding/doc_embeddings.bin` | Pre-built document embeddings |

> The CORD-19 dataset is available at [semanticscholar.org/cord19](https://www.semanticscholar.org/cord19).  
> GloVe embeddings (`glove.6B.300d.txt`) are available at [nlp.stanford.edu/projects/glove](https://nlp.stanford.edu/projects/glove/).

---

## Building

```bash
# Clone the repository
git clone https://github.com/murshadaziz/DSA_search_engine.git
cd DSA_search_engine

# Create build directory
mkdir build && cd build

# Configure and build
cmake ..
cmake --build . --config Release
```

This produces two executables:
- `main` — Interactive terminal mode
- `main_server` — Server mode for the web frontend

---

## Configuration

Before running, update the base path in `src/main_server.cpp` (and `src/main.cpp` for terminal mode):

```cpp
const std::string BASE_PATH = "path/to/your/searchEngine/";
```

Also update the executable path in `web/server.js`:

```js
const CPP_SERVER = 'path/to/your/build/main_server';
```

---

## Running

### Terminal Mode (Interactive)

```bash
./build/main
```

```
> coronavirus treatment          # Keyword/semantic search
> ?corona                        # Auto-complete (prefix with '?')
> exit                           # Quit
```

### Web Mode

```bash
# Install Node.js dependencies
cd web
npm install

# Start the web server
node server.js
```

Then open your browser at `http://localhost:3000`.

The Node.js server spawns the C++ `main_server` process and communicates via a simple stdin/stdout JSON protocol:

| Command | Example |
|---------|---------|
| `SEARCH <query>` | `SEARCH coronavirus vaccine` |
| `AUTOCOMPLETE <prefix>` | `AUTOCOMPLETE cor` |
| `EXIT` | Gracefully shuts down |

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/search?q=<query>` | Returns top-10 semantic search results with scores and URLs |
| `GET` | `/api/autocomplete?q=<prefix>` | Returns up to 10 word suggestions |
| `GET` | `/api/health` | Returns server readiness status |

### Example Response (`/api/search?q=covid vaccine`)

```json
{
  "results": [
    {
      "title": "Efficacy of mRNA vaccines against SARS-CoV-2",
      "score": 0.9231,
      "url": "https://doi.org/..."
    }
  ],
  "search_time_ms": "42.317"
}
```

---

## Dataset

This project uses the **CORD-19 (COVID-19 Open Research Dataset)**, a dataset of over 400,000 scholarly articles about COVID-19 and related coronaviruses. A sample subset is included in `sample_data_subset/` for testing.

---

## Technologies Used

| Layer | Technology |
|-------|-----------|
| Core Engine | C++17 |
| Build System | CMake |
| Semantic Search | GloVe (300D word embeddings) |
| Web Server | Node.js + Express |
| JSON Parsing | nlohmann/json |
| Dataset | CORD-19 |

---

## License

This project is open source. Feel free to fork and build upon it.
