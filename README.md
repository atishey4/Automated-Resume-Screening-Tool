# 📋 Automated Resume Screening Tool

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.103-green.svg)](https://fastapi.tiangolo.com/)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.28-red.svg)](https://streamlit.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

An AI-powered resume screening and matching system that automatically ranks candidates against job descriptions using fuzzy skill matching, sentence embeddings, and composite scoring formulas.

## 📸 Screenshots

![Screenshot 1](screenshots/Screenshot%202026-05-06%20175714.png)

![Screenshot 2](screenshots/Screenshot%202026-05-06%20175730.png)

![Screenshot 3](screenshots/Screenshot%202026-05-06%20175743.png)

![Screenshot 4](screenshots/Screenshot%202026-05-06%20175754.png)

## 🎯 Problem Statement

Recruiting teams face significant challenges when screening large volumes of resumes:
- **Manual Review**: Time-consuming to read hundreds of resumes
- **Inconsistency**: Subjective evaluation criteria across reviewers
- **Hidden Gems**: Qualified candidates can be overlooked
- **Skill Mismatches**: Difficulty identifying candidate skills vs. job requirements

This tool **automates resume screening** by intelligently matching candidate skills and experience against job requirements, providing recruiters with ranked, scored candidates.

## ✨ Key Features

- **Multi-Format Support**: Parse PDF, DOCX, and TXT resumes
- **Smart Skill Detection**: Fuzzy matching against 20+ technical skills
- **Experience Extraction**: Regex-based years of experience parsing
- **Semantic Similarity**: Sentence-transformer embeddings for relevance scoring
- **Composite Scoring**: 4-component formula for fair matching
- **Web Dashboard**: Streamlit UI for non-technical recruiters
- **REST API**: FastAPI endpoints for programmatic access
- **Database Persistence**: SQLite for tracking candidates and rankings

## 🏗️ Tech Stack

### Backend
- **Python 3.10+**: Core language
- **FastAPI**: REST API framework
- **Uvicorn**: ASGI server
- **SQLite**: Lightweight database

### Data Processing
- **pdfminer.six**: PDF text extraction
- **python-docx**: DOCX parsing
- **pandas**: Data manipulation
- **rapidfuzz**: Fuzzy string matching
- **sentence-transformers**: Semantic embeddings (all-MiniLM-L6-v2)
- **scikit-learn**: ML utilities
- **spacy**: NLP capabilities

### Frontend
- **Streamlit**: Interactive dashboard
- **Pydantic**: Data validation

## 📁 Project Structure

```
Automated-Resume-Screening-Tool/
├── api/
│   ├── __init__.py
│   └── app.py                    # FastAPI endpoints
│
├── db/
│   ├── init_db.py                # Database initialization script
│   ├── schema.sql                # SQLite schema definition
│   └── resumes.db                # SQLite database (auto-created)
│
├── src/
│   ├── ingest.py                 # Resume file parsing & storage
│   ├── extract.py                # Skill & experience extraction
│   ├── features.py               # JD-resume similarity scoring
│   ├── rank.py                   # Composite ranking formula
│   └── dashboard.py              # Streamlit UI
│
├── data/
│   ├── resume_alice.txt          # Sample resumes
│   ├── resume_bob.txt
│   ├── resume_carol.txt
│   ├── resume_david.txt
│   ├── resume_emma.txt
│   └── job_data_analyst.json     # Sample job description
│
├── screenshots/                  # App screenshots
├── resumes/                      # Upload directory for resumes
├── tool/                         # Tool utilities
├── docs/                         # Documentation
├── notebooks/                    # Jupyter notebooks
│
├── main.py                       # Entry point
├── requirements.txt              # Python dependencies
├── .gitignore                    # Git ignore rules
├── API_QUICKSTART.md             # API usage guide
├── DASHBOARD_GUIDE.md            # Dashboard usage guide
└── README.md                     # This file
```

## 🚀 Quick Start

### Prerequisites
- Python 3.10 or higher
- pip or conda

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/Automated-Resume-Screening-Tool.git
   cd Automated-Resume-Screening-Tool
   ```

2. **Create virtual environment** (recommended)
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Initialize database**
   ```bash
   python db/init_db.py
   ```

### Running the Application

#### Option 1: Streamlit Dashboard (Recommended for Recruiters)
```bash
streamlit run src/dashboard.py
```
Access at: http://localhost:8501

#### Option 2: FastAPI Server (Programmatic Access)
```bash
uvicorn api.app:app --reload --host 0.0.0.0 --port 8000
```
Access at: http://localhost:8000
- Interactive API docs: http://localhost:8000/docs
- Alternative docs: http://localhost:8000/redoc

#### Option 3: Command Line (For Testing)
```bash
python main.py
```

## 📖 Usage Guide

### Dashboard Workflow

1. **Upload Resumes**
   - Click sidebar → Select resume files (PDF/DOCX/TXT)
   - Click "Upload & Parse Resumes"

2. **Enter Job Details**
   - Paste job description in text area
   - Enter must-have skills (comma-separated)
   - Set minimum years of experience

3. **Screen Resumes**
   - Click "🔍 Screen Resumes"
   - View ranked candidate table

4. **Export Results**
   - Click "📥 Download Results as CSV"

### API Usage

**Create Job:**
```bash
curl -X POST "http://localhost:8000/job/create" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Data Analyst",
    "description": "Need SQL, Excel, Power BI expert",
    "experience_years": 3
  }'
```

**Upload Resume:**
```bash
curl -X POST "http://localhost:8000/resume/upload" \
  -F "file=@resume.pdf" \
  -F "candidate_name=John Doe" \
  -F "candidate_email=john@example.com"
```

**Rank Resumes:**
```bash
curl -X POST "http://localhost:8000/rank/1"
```

## 📊 Scoring Formula

The composite match score combines four weighted components:

```
score = 0.55×similarity + 0.35×coverage + 0.10×experience − penalty

Where:
- similarity (55%):     Cosine similarity between JD and resume skills
- coverage (35%):       Must-have skills matched / total must-have skills
- experience (10%):     min(1.0, candidate_years / required_years)
- penalty:              Gap deduction for missing skills and experience
```

**Example Calculation:**
```
Job: Need 3 years, SQL, Excel, Power BI
Candidate: Has 4 years, SQL, Excel (missing Power BI)

similarity = 0.82 (semantic match)
coverage = 2/3 = 0.67 (2 of 3 skills)
experience = min(1.0, 4/3) = 1.0
penalty = 0.33 (1 missing skill + small experience gap)

score = 0.55(0.82) + 0.35(0.67) + 0.10(1.0) - 0.33
score = 0.451 + 0.235 + 0.10 - 0.33 = 0.456
```

## 📋 Sample Output

### Screening Results Table

| Candidate | Email | Match Score | Matched Skills | Missing Skills | Coverage | Years |
|-----------|-------|------------|----------------|----------------|----------|-------|
| Alice Johnson | alice@email.com | 0.8234 | SQL, Excel, Power BI | - | 3/3 | 6.0 |
| Bob Chen | bob@email.com | 0.6521 | SQL, Excel | Power BI | 2/3 | 4.0 |
| Carol Martinez | carol@email.com | 0.4892 | Excel, Power BI | SQL | 2/3 | 8.0 |
| David Kumar | david@email.com | 0.3156 | SQL | Excel, Power BI | 1/3 | 7.0 |
| Emma Wilson | emma@email.com | 0.2847 | Excel | SQL, Power BI | 1/3 | 3.0 |

## 🔧 Module Documentation

### `src/ingest.py`
Handles resume file parsing and database storage
- `TextExtractor.read_text()`: Parse PDF/DOCX/TXT
- `ResumeStore.save_resume()`: Store to database

### `src/extract.py`
Extracts candidate features from resume text
- `fuzzy_skills()`: Match skills using rapidfuzz
- `total_years()`: Extract years of experience
- `parse_resume()`: Combined feature extraction

### `src/features.py`
Computes JD-resume matching features
- `jd_resume_features()`: Similarity scoring and skill coverage
- Lazy-loads sentence-transformers model

### `src/rank.py`
Calculates final match scores and persists results
- `rank_job()`: Apply scoring formula
- `bulk_rank_resumes()`: Batch ranking

### `api/app.py`
REST API endpoints
- `POST /job/create`: Create job
- `POST /resume/upload`: Upload and parse resume
- `POST /rank/{job_id}`: Rank resumes

### `src/dashboard.py`
Streamlit interactive dashboard for recruiters
- Resume upload interface
- Job description input
- Real-time ranking visualization
- CSV export functionality

## 📚 Database Schema

### Tables

**jobs**
- id, title, description, requirements, skills, experience_years, created_at, updated_at

**candidates**
- id, name, email, phone, location, created_at, updated_at

**resumes**
- id, candidate_id, file_path, file_type, file_size, extracted_text, parsed_json, uploaded_at, updated_at

**features**
- id, resume_id, feature_name, feature_value, score, extracted_at

**rankings**
- id, job_id, resume_id, candidate_id, match_score, skills_match, experience_match, ranking_date

## 🎓 Learning Outcomes

This project demonstrates:

### Machine Learning
- ✅ Fuzzy string matching (rapidfuzz)
- ✅ Sentence embeddings (sentence-transformers)
- ✅ Cosine similarity scoring
- ✅ Feature extraction and normalization
- ✅ Composite scoring and weighted formulas

### NLP
- ✅ Text preprocessing and normalization
- ✅ Regex pattern matching for data extraction
- ✅ Semantic similarity computation
- ✅ Skill entity recognition

### Software Engineering
- ✅ REST API design (FastAPI)
- ✅ Database design and SQL (SQLite)
- ✅ Multi-format file parsing
- ✅ Interactive UI development (Streamlit)
- ✅ Session management and state handling

### Data Processing
- ✅ PDF text extraction
- ✅ DOCX document parsing
- ✅ JSON serialization
- ✅ Data validation with Pydantic
- ✅ CSV export

### DevOps & Deployment
- ✅ Python packaging and dependencies
- ✅ Virtual environments
- ✅ Docker-ready structure
- ✅ Configuration management

## 🔮 Future Enhancements

- [ ] Add more language support
- [ ] Implement duplicate resume detection
- [ ] Add candidate feedback mechanism
- [ ] Build advanced analytics dashboard
- [ ] Integrate with ATS systems
- [ ] Support batch job processing
- [ ] Add role-based access control
- [ ] Implement search and filtering
- [ ] Add candidate notes and comments
- [ ] Support dynamic skill weighting
- [ ] Build email notification system
- [ ] Add A/B testing for scoring formula

## 🧪 Testing

Run unit tests:
```bash
pytest tests/
```

Test with sample data:
```bash
# Upload sample resumes
for file in data/resume_*.txt; do
  curl -X POST "http://localhost:8000/resume/upload" \
    -F "file=@$file" \
    -F "candidate_name=$(basename $file)"
done

# Screen against sample job
curl -X POST "http://localhost:8000/rank/1"
```

## 📋 Dependencies

See [requirements.txt](requirements.txt) for complete list. Key packages:

```
pdfminer.six==20221105         # PDF parsing
python-docx==0.8.11           # DOCX parsing
fastapi==0.103.0              # REST API
streamlit==1.28.1             # Dashboard
sentence-transformers==2.2.2  # Embeddings
rapidfuzz==3.1.1              # Fuzzy matching
pandas==2.0.3                 # Data processing
```

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📧 Contact & Support

For questions, issues, or suggestions, please open an issue on GitHub.

---

**Built with ❤️ for recruiters and data professionals**

Last Updated: May 2026 | Version: 1.0.0
