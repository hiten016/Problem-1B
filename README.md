# Problem-1B- Persona Based outputs 
This project is a complete solution to **Challenge 1b** of the **Adobe India Hackathon 2025**. It processes a collection of PDFs and extracts structured, semantically relevant sections based on a **persona** and a **specific task**. The output is a well-defined JSON file meeting all format and compute constraints.

adobe-hackathon-challenge1b-master/
├── Collection_1/
│ ├── PDFs/
│ ├── challenge1b_input.json
│ └── challenge1b_output.json 
├── Collection_2/
├── Collection_3/
├── Scripts/
│ └── approach_explanation.md
├── run_analysis.py 
├── Dockerfile 
├── requirements.txt 
└── README.md 


---

##  Problem Statement

You are given:
- A directory of PDFs
- A `challenge1b_input.json` file with:
  - List of PDFs to process
  - Persona context
  - A job-to-be-done (task description)

You must:
- Extract relevant content
- Filter unwanted sections (if required)
- Rank chunks based on task relevance
- Generate concise titles using an LLM
- Output structured data to `challenge1b_output.json` that matches the schema

---

##  Approach Overview

We use a **two-stage semantic retrieval and summarization pipeline** to achieve this:

### 🔹 1. **PDF Text Extraction**
- Tool: [`pdfplumber`](https://github.com/jsvine/pdfplumber)
- We extract each page's content and split it into paragraph-like chunks.
- Only non-trivial text sections (>50 characters) are retained.

### 🔹 2. **Stage 1: Keyword-Based Filtering (optional)**
- If required by the challenge (e.g., `round_1b_001`), we filter out text chunks containing forbidden terms like `"meat"`, `"chicken"`, etc., using substring matching.

### 🔹 3. **Stage 2: Semantic Similarity Ranking**
- Tool: [`all-MiniLM-L6-v2`](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)
- For each text chunk, compute cosine similarity with the job description.
- Rank and select top `k` most relevant sections.

### 🔹 4. **Title Generation with LLM**
- Tool: [`google/flan-t5-base`](https://huggingface.co/google/flan-t5-base)
- For each top-ranked chunk, we generate a short, descriptive title.
- Efficient decoding setup (`max_new_tokens=20`, `num_beams=4`) ensures speed.

### 🔹 5. **Output Construction**
- Conform to Adobe's required schema: Metadata, Extracted Sections, and Subsection Analysis.

---

## 🧠 Model Responsibilities

| Model                  | Task                                | Why Chosen                           |
|------------------------|-------------------------------------|--------------------------------------|
| `all-MiniLM-L6-v2`     | Semantic similarity retrieval       | Small (~90MB), fast, high accuracy   |
| `google/flan-t5-base`  | Title generation (summarization)    | Balanced size (~250MB), good output  |

---

## Setup & Run Instructions

### Local Python Setup

```bash
# Step 1: Clone repository
git clone https://github.com/your-username/adobe-hackathon-challenge1b.git
cd adobe-hackathon-challenge1b-master

# Step 2: Create virtual environment
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Step 3: Install dependencies
pip install -r requirements.txt

# Step 4: Run the script on a collection
python run_analysis.py Collection_1 --top_k 15

```
## Docker Setup
### Build Docker image
docker build --platform linux/amd64 -t pdf-analyzer-1b .

### Run the container on Collection_1
docker run --rm \
-v $(pwd)/Collection_1:/app/Collection_1:rw \
--network none pdf-analyzer-1b \
python run_analysis.py Collection_1 --top_k 15



### Satisfied Constraints

 Output JSON follows required schema  
 Execution limited to CPU only       
 Total model size ≤ **1 GB**        
 Completion time ≤ 60s (3–5 PDFs)    
 No internet access during runtime    
 Works with 8 CPU cores, 16GB RAM     
 Portable via Docker (amd64 platform) 

