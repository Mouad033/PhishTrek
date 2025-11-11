# ![PhishTrek Logo](https://raw.githubusercontent.com/Mouad033/PhishTrek/main/assets/phishtrek_logo.png)

PhishTrek
=========

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Status: Development](https://img.shields.io/badge/Status-Development-orange.svg)
![Build Status](https://github.com/Mouad033/PhishTrek/workflows/CI/badge.svg)

**PhishTrek: AI-Powered Phishing Testing & Defense Improvement Platform**

[PhishTrek](https://github.com/Mouad033/PhishTrek) is an open-source platform designed for security teams and penetration testers. It provides the ability to quickly generate realistic AI-powered phishing campaigns, automatically test defenses, and instantly generate security rules for SIEM systems.

### Features

- **AI-Powered Generation**: Create 30+ realistic phishing email variants in 2 minutes using GPT-4, Claude, or Ollama
- **Automatic Detection Testing**: Scan generated emails against your security tools
- **Instant Rule Generation**: Auto-generate Sigma rules ready for Splunk, Elastic, or other SIEM systems
- **Closed-Loop Feedback**: Continuous improvement cycle from attack generation to defense enhancement
- **Interactive Dashboard**: Real-time metrics, detection rates, and actionable insights
- **Multi-LLM Support**: Use OpenAI API, Claude API, or run Ollama locally (no internet required)

### Install

Installation of PhishTrek is straightforward - just download and extract the repository:

```bash
git clone https://github.com/Mouad033/PhishTrek.git
cd PhishTrek
```

### Quick Start with Docker

The fastest way to get started is with Docker:

```bash
# Clone the repository
git clone https://github.com/Mouad033/PhishTrek.git
cd PhishTrek

# Start the application
docker-compose up -d

# Open browser
Visit: http://localhost:3000
```

Default credentials and API configuration will be shown in the logs.

### Building From Source

PhishTrek requires Python 3.10+ and Node.js 18+

**Backend Setup:**
```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

**Frontend Setup:**
```bash
cd frontend
npm install
npm run dev
```

### Setup

After starting PhishTrek with Docker, open your browser to `http://localhost:3000` and login with the default credentials shown in the logs.

**Your first phishing test:**
1. Click "Generate Phishing Campaign"
2. Choose a target persona (e.g., "Finance Manager")
3. Click "Generate" - wait 2 minutes for AI to create variants
4. View automatic detection results
5. Export ready-to-deploy security rules

### Documentation

Complete documentation is available on our [GitHub Wiki](https://github.com/Mouad033/PhishTrek/wiki). Find something missing? Let us know by filing an issue!

Key documentation:
- **[SETUP.md](docs/SETUP.md)** - Installation & local development
- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - System design & technical details
- **[API_SPEC.md](docs/API_SPEC.md)** - Complete API reference
- **[CONTRIBUTING.md](docs/CONTRIBUTING.md)** - How to contribute

### API Integration

For teams wanting to integrate PhishTrek with other tools:

```bash
# Generate phishing emails
POST /api/v1/generator/phishing

# Analyze email for phishing
POST /api/v1/detector/analyze

# Generate Sigma rules
POST /api/v1/rules/generate

# Get dashboard metrics
GET /api/v1/dashboard/metrics
```

Full API documentation: See [API_SPEC.md](docs/API_SPEC.md)

### Use Cases

| Role | Benefit |
|------|---------|
| **Red Team / Penetration Testers** | Test client defenses 10x faster with AI-generated variants |
| **SOC Analysts** | Validate detection rules before SIEM deployment |
| **SIEM Engineers** | Get production-ready Sigma rules automatically |
| **Security Awareness Trainers** | Create realistic phishing scenarios for employee training |
| **Compliance Teams** | Meet regulatory testing requirements (NIS2, GDPR) |

### Issues

Find a bug? Want more features? Have a suggestion? Let us know! Please don't hesitate to [file an issue](https://github.com/Mouad033/PhishTrek/issues/new) and we'll get right on it.

### Contributing

We welcome contributions! Please see [CONTRIBUTING.md](docs/CONTRIBUTING.md) for guidelines.

**Quick start for developers:**
```bash
git checkout -b feature/your-feature
git commit -m "feat: your description"
git push origin feature/your-feature
# Create Pull Request on GitHub
```

### Support & Questions

- **Documentation**: [docs/](docs/)
- **Issues & Bugs**: [GitHub Issues](https://github.com/Mouad033/PhishTrek/issues)
- **Feature Requests**: [GitHub Discussions](https://github.com/Mouad033/PhishTrek/discussions)

### Security Notice

PhishTrek is designed **ONLY** for authorized security testing. Before using:
- Ensure written authorization from system owners
- Use only in controlled, sandboxed environments
- Never target external systems without explicit consent
- Comply with local laws and regulations

Unauthorized phishing is illegal. Use responsibly.

### License
```sql
PhishTrek - AI-Powered Phishing Testing Platform

The MIT License (MIT)

Copyright (c) 2025 Mouad BENTEFRIT & Sofiane BENNEDJIMA

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```

### Team

Built with love by ESME MSI students:
- **Mouad BENTEFRIT** (Developer)
- **Sofiane BENNEDJIMA** (Developer)

Institution: ESME 
Location: Paris, France
Year: 2024-2025

---

Made with ❤️ for better cybersecurity

*Making security testing faster, smarter, and more effective*

---

Last Updated: November 11, 2025
