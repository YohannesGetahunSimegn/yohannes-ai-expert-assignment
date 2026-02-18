
#### Running tests locally

```bash
python -m venv .venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
pytest -v
```

#### Building and running tests with Docker

```bash
docker build -t ai-experts-assignment . && docker run ai-experts-assignment
```



