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
- Tip: use `magika --label path/to/file` when you only need the content type label and want cleaner output without the full JSON.
- Tip: use `magika --mime path/to/file` to get the MIME type directly (e.g. `text/x-python`) — useful when you need to set Content-Type headers or integrate with tools that expect MIME types.
- Tip: for a quick sanity check on a single file, `magika --json file | jq '{type: .result.output.ct_label, score: .result.output.score}'` gives a compact summary.
- Tip: when processing a large directory, `magika --json --recursive dir/ 2>/dev/null` suppresses permission errors that would otherwise clutter the output.
- Tip: to count detections by type across a directory, `magika --json --recursive dir/ 2>/dev/null | jq -r '.result.output.ct_label' | sort | uniq -c | sort -rn` gives a quick frequency breakdown.
- Tip: save TSV output to a file for later review: `magika --json --recursive dir/ 2>/dev/null | jq -r '[.path, .result.output.ct_label, (.result.output.score | tostring)] | join("\t")' > audit.tsv`
- Tip: on macOS, `xargs` has a lower default argument limit — use `xargs -S 131072 magika --json` if you hit "argument list too long" errors with large file sets.
