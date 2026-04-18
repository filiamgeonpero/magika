# Magika

A fork of [google/magika](https://github.com/google/magika) — AI-powered file type detection.

Magika uses a deep learning model to detect file content types with high accuracy, even when file extensions are missing or misleading.

## Features

- Fast, accurate file type detection using a custom Keras model
- Supports 100+ content types
- Python library and CLI tool
- Rust bindings available
- Docker support

## Installation

### From PyPI

```bash
pip install magika
```

### From Source

```bash
git clone https://github.com/your-org/magika.git
cd magika
pip install -e python/
```

## Quick Start

### Python API

```python
from magika import Magika

m = Magika()
result = m.identify_path("path/to/file")
print(result.output.ct_label)  # e.g. "python"
print(result.output.score)     # confidence score (0.0 to 1.0)
```

> **Note:** A score above ~0.9 generally indicates high confidence. Scores below 0.5 suggest the file type is ambiguous or unknown.

### CLI

```bash
magika path/to/file
magika --recursive path/to/directory/
magika --json path/to/file
```

## Docker

```bash
docker build -t magika .
docker run --rm -v $(pwd):/data magika /data/yourfile
```

## Development

### Prerequisites

- Python 3.10+
- Rust (for Rust bindings)

### Setup

```bash
pip install -e "python/[dev]"
```

### Running Tests

```bash
pytest python/tests/
```

## Personal Notes

- I primarily use this for quickly auditing uploaded files in small scripts.
- The `--json` flag is especially handy when piping output into `jq`.
- Useful one-liner for batch auditing: `find . -type f | xargs magika --json | jq 'select(.result.output.score < 0.5)'` to surface low-confidence detections.
- I keep a local alias: `alias ftype='magika --json'` in my shell config.
- Another handy alias for recursive audits: `alias faudit='magika --json --recursive'`.
- To list only high-confidence results: `find . -type f | xargs magika --json | jq 'select(.result.output.score >= 0.9)'`
- Tip: pipe results through `jq -r '[.path, .result.output.ct_label, (.result.output.score | tostring)] | join("\t")'` for a clean TSV summary.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Please report misdetections or request new content types via [GitHub Issues](https://github.com/your-org/magika/issues).

## License

Apache 2.0 — see [LICENSE](LICENSE).
