# gdocs-client

I extracted this small synchronous client for integrations that create Google Docs documents and append or read plain text. It uses Google's official Python client and has no Home Assistant dependency.

## Install

```sh
pip install gdocs-client
```

## Use

I keep OAuth authorization and token refresh in the calling application. Each operation takes a valid access token and closes its HTTP connection when it finishes.

```python
from gdocs_client import append_text, create_document, get_document

access_token = "YOUR_ACCESS_TOKEN"
document = create_document(access_token, "Daily log")
document_id = document["documentId"]

append_text(access_token, document_id, "The washing machine finished.\n")
result = get_document(access_token, document_id)
print(result["title"])
print(result["text"])

# Check access without retrieving document content.
metadata = get_document(access_token, document_id, include_content=False)
```

I use the first document tab. Appends preserve the supplied text and do not add a timestamp, separator, or newline. Writes are not automatically retried, since an uncertain result could otherwise duplicate text.

The `text` field includes text runs in paragraphs, nested tables, and tables of contents in document order. It removes Google's mandatory final newline and preserves all other newlines. Formatting, images, headers, footers, and footnotes are not included. Metadata-only reads do not add a `text` field.

Google authentication and HTTP errors propagate to the caller. `API_ERRORS` groups the expected authentication, HTTP, transport, and operating system errors for applications that need a shared handler.

## Development

```sh
pip install -e '.[test]'
ruff check .
ruff format --check .
mypy src
pytest --cov --cov-report=term-missing
python -m build
twine check dist/*
```

I publish releases through the repository's GitHub Actions workflow using PyPI Trusted Publishing. Each release tag matches the package version and publishes a wheel and source distribution.
