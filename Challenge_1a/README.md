# Challenge 1A: PDF Structure Extraction Solution

## Overview
This solution processes PDF documents to extract their title and hierarchical outline structure, outputting the results in a structured JSON format. The system is designed to work efficiently on CPU-only environments with strict performance constraints.

## Approach

### 1. Title Extraction Strategy
The solution employs a multi-layered approach for title extraction:

1. **Primary Method**: Extract title from PDF metadata (`doc.metadata["title"]`)
2. **Fallback Method**: Analyze the first page content:
   - Extract text blocks with font size and positioning information
   - Identify the largest/most prominent text elements
   - Select the most likely title candidate based on position and size

### 2. Outline Generation Strategy
The outline extraction uses a hierarchical approach:

1. **Primary Method**: Extract from PDF bookmarks/Table of Contents if available
2. **Fallback Method**: Font-based analysis:
   - Analyze font sizes across all pages
   - Map font sizes to heading levels (H1, H2, H3, etc.)
   - Apply heuristics to identify likely headings:
     - Font size significance
     - Bold formatting
     - Text length constraints
     - Pattern matching for numbered sections

### 3. Smart Heading Detection
The system uses multiple criteria to identify headings:
- **Font Size Analysis**: Larger fonts typically indicate higher-level headings
- **Formatting Detection**: Bold text is often used for headings
- **Pattern Recognition**: Numbered sections (1., 2.1, Chapter, Section, etc.)
- **Length Constraints**: Headings are typically shorter than body text
- **Position Analysis**: Headings often appear at specific page positions

## Libraries and Dependencies

### Core Libraries
- **PyMuPDF (fitz) v1.23.14**: 
  - Primary PDF processing library
  - Lightweight and efficient for text extraction
  - Provides font, formatting, and positioning information
  - Size: ~20MB (well under 200MB constraint)
  - No GPU dependencies, pure CPU implementation

### Standard Libraries
- **json**: JSON output formatting
- **os**: File system operations
- **re**: Regular expression pattern matching
- **collections.defaultdict**: Data structure optimization

## Performance Optimizations

1. **Efficient Text Processing**: Uses PyMuPDF's optimized text extraction
2. **Smart Fallbacks**: Prioritizes faster methods (bookmarks) before font analysis
3. **Memory Management**: Proper document cleanup and resource management

## Docker Configuration

The solution is containerized using a Python 3.10 base image with linux/amd64 platform specification:

- **Base Image**: `python:3.10` (AMD64 compatible)
- **Platform**: Explicitly set to `linux/amd64`
- **Dependencies**: Only PyMuPDF installed via pip
- **No Network Access**: Works completely offline
- **Resource Requirements**: CPU-only, no GPU dependencies

## File Structure

```
Challenge_1a/
├── Dockerfile                 # Container configuration
├── process_pdfs_improved.py  # Main processing script
├── process_pdfs.py           # Alternative implementation
├── test_process_pdfs.py      # Testing version
├── README.md                 # This documentation
└── sample_dataset/           # Test data and expected outputs
```

## Expected Input/Output

### Input
- PDF files in `/app/input` directory
- Any number of PDF documents

### Output
- JSON files in `/app/output` directory
- One JSON file per input PDF (filename.pdf → filename.json)

### JSON Schema
```json
{
  "title": "Document Title",
  "outline": [
    {
      "level": "H1|H2|H3|H4|H5|H6",
      "text": "Heading text",
      "page": 1
    }
  ]
}
```

## How to Build and Run

### Building the Docker Image
```bash
docker build --platform linux/amd64 -t challenge1a:latest .
```

### Running the Solution
```bash
docker run --rm \
  -v $(pwd)/input:/app/input \
  -v $(pwd)/output:/app/output \
  --network none \
  challenge1a:latest
```

### Local Testing (Development)
```bash
# Install dependencies
pip install PyMuPDF==1.23.14

# Run with local paths
python test_process_pdfs.py
```

## Performance Characteristics

- **Processing Speed**: Designed to process 50-page PDFs in ≤10 seconds
- **Memory Usage**: Optimized for 16GB RAM systems
- **CPU Usage**: Efficiently utilizes 8-core CPU systems
- **Model Size**: Uses no AI models, only traditional text processing
- **Network**: Zero network dependencies, works completely offline

## Compliance with Requirements

### Docker Requirements ✅
- **Platform**: `--platform=linux/amd64` specified in Dockerfile
- **Architecture**: Compatible with AMD64 (x86_64) CPU
- **No GPU**: Pure CPU implementation
- **Model Size**: No models used (0MB < 200MB limit)
- **Offline**: No network/internet calls

### Expected Execution ✅
- **Build Command**: `docker build --platform linux/amd64 -t mysolutionname:identifier .`
- **Run Command**: `docker run --rm -v $(pwd)/input:/app/input -v $(pwd)/output:/app/output --network none mysolutionname:identifier`
- **Auto Processing**: Processes all PDFs from `/app/input`
- **Output Format**: Generates `filename.json` for each `filename.pdf`

### Constraints ✅
- **Execution Time**: ≤10 seconds for 50-page PDF
- **Model Size**: No models used
- **Network**: No internet access required
- **Runtime**: CPU-only on 8 CPUs, 16GB RAM

## Error Handling

The solution includes robust error handling:
- Graceful fallback when PDF metadata is unavailable
- Valid JSON output even when processing fails
- Detailed logging for debugging
- Handles malformed or corrupted PDFs

## Testing

The solution has been tested with:
- Various PDF document types
- Documents with and without bookmarks
- Different font size distributions
- Form documents and structured reports
- Academic papers and technical documents

All test cases produce valid JSON output conforming to the required schema.

### Current Sample Solution
The provided `process_pdfs.py` is a **basic sample** that demonstrates:
- PDF file scanning from input directory
- Dummy JSON data generation
- Output file creation in the specified format

**Note**: This is a placeholder implementation using dummy data. A real solution would need to:
- Implement actual PDF text extraction
- Parse document structure and hierarchy
- Generate meaningful JSON output based on content analysis

### Sample Processing Script (`process_pdfs.py`)
```python
# Current sample implementation
def process_pdfs():
    input_dir = Path("/app/input")
    output_dir = Path("/app/output")
    
    # Process all PDF files
    for pdf_file in input_dir.glob("*.pdf"):
        # Generate structured JSON output
        # (Current implementation uses dummy data)
        output_file = output_dir / f"{pdf_file.stem}.json"
        # Save JSON output
```

### Sample Docker Configuration
```dockerfile
FROM --platform=linux/amd64 python:3.10
WORKDIR /app
COPY process_pdfs.py .
CMD ["python", "process_pdfs.py"]
```

## Expected Output Format

### Required JSON Structure
Each PDF should generate a corresponding JSON file that **must conform to the schema** defined in `sample_dataset/schema/output_schema.json`.


## Implementation Guidelines

### Performance Considerations
- **Memory Management**: Efficient handling of large PDFs
- **Processing Speed**: Optimize for sub-10-second execution
- **Resource Usage**: Stay within 16GB RAM constraint
- **CPU Utilization**: Efficient use of 8 CPU cores

### Testing Strategy
- **Simple PDFs**: Test with basic PDF documents
- **Complex PDFs**: Test with multi-column layouts, images, tables
- **Large PDFs**: Verify 50-page processing within time limit


## Testing Your Solution

### Local Testing
```bash
# Build the Docker image
docker build --platform linux/amd64 -t pdf-processor .

# Test with sample data
docker run --rm -v $(pwd)/sample_dataset/pdfs:/app/input:ro -v $(pwd)/sample_dataset/outputs:/app/output --network none pdf-processor
```

### Validation Checklist
- [ ] All PDFs in input directory are processed
- [ ] JSON output files are generated for each PDF
- [ ] Output format matches required structure
- [ ] **Output conforms to schema** in `sample_dataset/schema/output_schema.json`
- [ ] Processing completes within 10 seconds for 50-page PDFs
- [ ] Solution works without internet access
- [ ] Memory usage stays within 16GB limit
- [ ] Compatible with AMD64 architecture

---

**Important**: This is a sample implementation. Participants should develop their own solutions that meet all the official challenge requirements and constraints. 