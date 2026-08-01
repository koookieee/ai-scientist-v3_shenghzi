FROM python:3.12-slim

# System deps: lancedb needs libgomp, httpx needs SSL
RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates curl libgomp1 && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# torch/transformers/adapters are required because the embedder module imports them
# at load time even when using the Gemini backend. CPU wheel keeps the image ~200MB
# instead of ~3GB.
RUN pip install --no-cache-dir --extra-index-url https://download.pytorch.org/whl/cpu \
        "torch==2.12.0" && \
    pip install --no-cache-dir \
        "lancedb>=0.6" "numpy>=1.24" "httpx>=0.24" "networkx>=3.0" \
        "scipy>=1.11" "pandas>=2.0" "pyarrow>=14.0" \
        "aiohttp==3.13.5" \
        "huggingface_hub>=0.20" \
        "google-genai==2.7.0" \
        "transformers==4.57.6" \
        "adapters==1.3.0"

COPY arxiv_search_kit/ /app/arxiv_search_kit/
COPY search_api.py /app/search_api.py

# The index lives in a volume so the ~10 GB download persists across restarts.
ENV ARXIV_SEARCH_KIT_CACHE=/data/arxiv_search_kit \
    HF_HOME=/data/hf

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

EXPOSE 8081

# /health is reachable as soon as the server binds; long start_period covers first-run index download.
HEALTHCHECK --interval=30s --timeout=10s --retries=3 --start-period=900s \
    CMD curl -fsS http://localhost:8081/health || exit 1

ENTRYPOINT ["/entrypoint.sh"]
