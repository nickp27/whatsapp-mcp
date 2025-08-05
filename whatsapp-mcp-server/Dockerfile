# Use Python 3.11 slim image
FROM python:3.11-slim

WORKDIR /app

# Install uv for dependency management
RUN pip install uv

# Copy dependency files
COPY pyproject.toml uv.lock ./

# Install dependencies using uv
RUN uv sync --frozen

# Copy application files
COPY *.py ./

# Run the MCP server
CMD ["uv", "run", "python", "main.py"]