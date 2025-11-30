# ai_document_analysis_service

## Project Overview
This is a FastAPI-based application for automated document analysis, primarily focused on extracting structured data from vendor quotations (PDFs) using Azure Document Intelligence and Azure OpenAI. The system processes PDFs, analyzes their content, validates results through self-reflection, and integrates with external APIs for data submission.

## Architecture & Data Flow

![image](https://hackmd.io/_uploads/B1lavTteZl.png)

- **Entry Points**:
  - `main.py`: FastAPI app exposing `/upload/` for PDF upload and analysis with background task processing.
  - `run.py`: Launches the FastAPI server using Uvicorn with multiprocessing support and settings from `app/conf/.env`.
  - `process_excel.py`: Batch processing of PDFs and Excel integration via async workflows with enhanced record management.
  - `run_history_di.py`: Historical Document Intelligence processing for existing Markdown files.
  - `start_server.bat`: Windows batch script for simplified server startup with environment validation.
- **Core Components**:
  - `app/repository/di_repository.py`: Integrates Azure Document Intelligence for PDF-to-markdown extraction.
  - `app/repository/ai_repository.py`: Handles Azure OpenAI (AOAI) chat model prompts, multimodal requests, and retry mechanisms.
  - `app/usecase/prompt_usecase.py`: Advanced prompt engineering with multimodal support and direct PDF processing capabilities.
  - `app/usecase/di_usecase.py`: Document Intelligence use case orchestration with file management.
  - `app/utils/`: Utilities for logging, PDF splitting, parsing LLM responses, and record writing.
- **Config & Data Models**:
  - All credentials and runtime settings are loaded from `app/conf/.env`.
  - `app/enums/`: Pydantic models for request/response validation including purchase data structures.

## Key Patterns & Conventions
- **PDF Processing**:
  - Supports both chunk-based and direct PDF processing methods.
  - Chunk processing: PDFs split into manageable pieces for granular analysis via Document Intelligence.
  - Direct processing: Multimodal LLM approach for immediate PDF analysis without pre-processing.
  - Enhanced record management with automatic timestamping and organized file structure.
- **Prompt Engineering**:
  - Highly structured and domain-specific prompts (see `SysPromptClass`) with strict extraction rules.
  - Supports both text-based and multimodal (PDF + text) prompt strategies.
  - Self-reflection prompts validate and correct initial LLM outputs with comprehensive JSON schemas.
  - Advanced retry mechanisms with exponential backoff for API resilience.
- **External API Integration**:
  - Structured data models using Pydantic for request/response validation.
  - Asynchronous background task processing for long-running operations.
  - External API submission with purchase data payload (`FinalPurchasePayload`).
- **Excel Integration**:
  - Enhanced DataFrame operations with filename-based matching.
  - Comparison functionality for validation and quality assurance.
  - Support for batch processing and result aggregation.
- **Logging & Record Management**:
  - Comprehensive Loguru-based logging with rotation and retention policies.
  - Organized record storage: DI results, recognition responses, and reflection outputs.
  - Historical processing capabilities for existing data analysis.
- **Async Workflows**:
  - Background task processing for non-blocking API responses.
  - Concurrent processing support with proper error handling and resource management.

## Developer Workflows
- **Run API Server**:
  - `python run.py` (uses Uvicorn with multiprocessing, settings from `.env`)
  - `start_server.bat` (Windows batch script with environment validation)
- **Batch Process PDFs & Excel**:
  - `python process_excel.py` for current PDF processing workflows
  - `python run_history_di.py` for historical data processing from existing DI markdown files
- **Environment Setup**:
  - All secrets/configs in `app/conf/.env`. Do not hardcode credentials.
  - Virtual environment setup and dependency management via batch scripts.
- **Debugging & Monitoring**:
  - Check `logs/app.log` and rotated/compressed logs for errors and workflow traces.
  - Structured record storage in `records/` with separate folders for different processing stages.
  - Background task monitoring through API response tracking.

## Integration Points
- **Azure Document Intelligence**: Used for PDF layout and content extraction with configurable model selection.
- **Azure OpenAI**: Used for prompt-based and multimodal LLM analysis with retry mechanisms and timeout handling.
- **Excel**: Pandas-based integration for reading/writing results with enhanced DataFrame operations.
- **External APIs**: Structured payload submission to procurement systems with validation and error handling.

## Project-Specific Guidance
- Always use the provided prompt templates and output schemas in `prompt_usecase.py`.
- When updating analysis logic, ensure both recognition and self-reflection flows are updated.
- For new document types, extend the prompt templates and parsing logic accordingly.
- When adding new external integrations, update `.env` and avoid hardcoding secrets.
- Use multimodal capabilities (`set_recognition_prompt_from_pdf`, `set_self_reflection_prompt_with_multimodal`) for direct PDF processing.
- Implement proper error handling and logging for production deployments.
- Follow async patterns for all I/O operations and API calls.

## Data Models & Validation
- **Upload Processing**: `UploadRequestEnum` for API request validation
- **Item Management**: `CommonItemEnum`, `StockItemEnum`, `NonStockItemEnum` for different item types
- **API Integration**: `FinalPurchasePayload`, `PurchaseContent`, `FinalPurchaseItem` for external API submission
- **Parsing Logic**: `DataParserClass` handles JSON parsing and DataFrame integration with robust error handling

## Record Management Structure
```
records/
├── di_markdown/           # Document Intelligence results (timestamped)
├── recognition_response/  # AI recognition results (timestamped)
└── reflection_response/   # AI self-reflection results (timestamped)

logs/
└── app.log               # Application logs (compressed rotation)
```

## Example: Adding a New PDF Analysis Flow
1. Create/extend a method in `SysPromptClass` for the new prompt (consider multimodal support).
2. Update `process_excel.py` or create new entry point to use the new flow.
3. Ensure output parsing in `parser_utils.py` matches the new schema.
4. Add proper error handling and logging throughout the workflow.
5. Update data models in `app/enums/` if new validation structures are needed.

## Advanced Features
- **Multimodal Processing**: Direct PDF-to-LLM analysis without Document Intelligence preprocessing
- **Historical Processing**: Batch processing of existing DI markdown files via `run_history_di.py`
- **Background Tasks**: Non-blocking PDF processing with FastAPI background task integration
- **Retry Logic**: Exponential backoff for API calls with configurable timeout and retry parameters
- **Record Validation**: Comprehensive data validation using Pydantic models throughout the pipeline

---
