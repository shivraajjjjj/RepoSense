# RepoSense - Repository to Role Analyzer
RepoSense analyzes a GitHub repository and intelligently predicts the most suitable job/internship roles based on real technical signals .

No resumes.
No forms.
Just code → roles.
## Problem Statement

Developers struggle to articulate their skills without manually analyzing their repositories. Teams need a scalable way to assess repository patterns against specific role requirements. RepoSense automates this by inspecting a public GitHub repository's structure, manifests, and signals to infer aligned developer roles.

## How It Works

1. **Input**: User provides a GitHub repository URL
2. **Analysis**: Backend fetches repo metadata, languages, folder structure, and manifest files (package.json, pom.xml, requirements.txt, etc.)
3. **Signal Extraction**: Parses manifests to extract runtime, frameworks, databases, and build tools
4. **Scoring**: Matches detected signals against predefined role configurations
5. **Output**: Returns ranked roles with raw scores, final scores, and confidence metrics

## Key Features

- **Role Detection**: Automatically identifies suitable roles (Frontend React Dev, Backend Java Dev, Full Stack JS Dev, ML Engineer, etc.)
- **Signal-Based Scoring**: Matches languages, frameworks, databases, and project structure against role requirements
- **Confidence Metrics**: Factors in coverage, strength, and penalties (toy projects, weak structure) to compute confidence
- **Repository Metadata**: Displays repo structure, detected languages, frameworks, databases, and build tools
- **Monorepo Detection**: Identifies and reports multi-manifest projects
- **Toy Project Detection**: Flags repositories that appear to be learning projects
- **Caching**: Reduces redundant GitHub API calls for recently analyzed repos

## Tech Stack

### Backend
- **Runtime**: Node.js
- **Framework**: Express.js
- **API**: GitHub REST API
- **Database**: PostgreSQL (optional, for caching)
- **Packages**: axios, express-validator, dotenv

### Frontend
- **Framework**: React 19
- **Build Tool**: Vite
- **Styling**: Tailwind CSS
- **HTTP Client**: Axios
- **UI**: Custom components with dark theme

## Project Structure

```
JobEngine/
├── Backend/
│   ├── src/
│   │   ├── app.js                    # Express app setup with CORS
│   │   ├── server.js                 # HTTP server entry
│   │   ├── config/
│   │   │   └── env.js               # Environment config (GitHub token, API base URL)
│   │   ├── controllers/
│   │   │   └── analyze.controller.js # Request handler & error management
│   │   ├── routes/
│   │   │   └── analyze.routes.js    # POST /analyze/your-roles endpoint
│   │   ├── services/
│   │   │   ├── analyze.service.js   # Core analysis logic
│   │   │   ├── github.service.js    # GitHub API client (direct fetch, no cache)
│   │   │   ├── signal.service.js    # Signal extraction from manifests
│   │   │   ├── scanner.service.js   # Manifest file scanning
│   │   │   └── toyDetector.js       # Toy project detection
│   │   ├── engines/
│   │   │   └── scoring.engine.js    # Role scoring & confidence calculation
│   │   ├── roles/
│   │   │   ├── index.js             # All role configurations
│   │   │   ├── backendJavaDev.js
│   │   │   ├── backendJsDev.js
│   │   │   ├── frontendReactDev.js
│   │   │   ├── fullstackJavaDev.js
│   │   │   ├── fullstackJsDev.js
│   │   │   └── mlEngineer.js
│   │   ├── utils/
│   │   │   └── parser.js            # GitHub URL parser
│   ├── package.json
│   ├── .env.example                 # Environment template
│   └── README.md
├── Frontend/
│   ├── src/
│   │   ├── App.jsx                  # Landing page with input & analyze button
│   │   ├── Result.jsx               # Results page with role cards & stack info
│   │   ├── main.jsx                 # React entry point
│   │   └── App.css
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── eslint.config.js
│   └── README.md
├── .gitignore                        # Git ignore rules (env, node_modules, db files, logs)
├── README.md                         # Project documentation
└── (project root)
```

**Note**: Database files (`db.js`, `cache.service.js`, `cacheKey.js`) have been removed. The app fetches directly from GitHub API without caching. Database-related env variables are not required.

## Setup & Run

### Backend
```bash
cd Backend
npm install
# Create .env with GITHUB_TOKEN and other configs
node src/server.js
```

### Frontend
```bash
cd Frontend
npm install
npm run dev  # Vite dev server on http://localhost:5173
```

## API Example

**Endpoint**: `POST /analyze/your-roles`

**Request**:
```json
{
  "repo": "https://github.com/facebook/react"
}
```

**Response**:
```json
{
  "supported": true,
  "projectSignals": {
    "repo": {
      "owner": "facebook",
      "name": "react",
      "url": "https://github.com/facebook/react"
    },
    "languages": ["JavaScript", "TypeScript"],
    "runtime": ["node"],
    "frameworks": ["React"],
    "databases": [],
    "buildFiles": ["package.json", "build.gradle"],
    "structure": ["src", "tests", "docs"],
    "flags": [],
    "roles": [
      {
        "roleId": "frontend-react-dev",
        "title": "Frontend React Developer",
        "rawScore": "78/100",
        "finalScore": 78,
        "confidence": "21%",
        "matchedSignals": [
          { "category": "frameworks", "signal": "React", "points": 25 },
          { "category": "languages", "signal": "JavaScript", "points": 10 }
        ]
      }
    ],
    "metadata": {
      "isToy": false,
      "hasReadme": true,
      "isMonorepo": false,
      "supported": true
    }
  }
}
```

**Error Response**:
```json
{
  "supported": false,
  "message": "Cannot reach GitHub API. Check your internet connection."
}
```

## Important Notes

- **Public Repositories Only**: Analyzer requires public GitHub repos; private repos are not accessible without authentication
- **Network Dependency**: Backend must have internet access to reach GitHub API
- **GitHub Rate Limiting**: Each analysis consumes one GitHub API request; consider rate limits (60 req/hr unauthenticated, 5000 req/hr authenticated)
- **Token Security**: Store `GITHUB_TOKEN` in `.env` securely; never commit to version control
- **No Caching**: Every request fetches fresh data from GitHub API; no local cache or database storage
- **No Database Required**: Database logic removed; app is stateless and requires only GitHub token
- **Monorepo Limitation**: V1 does not support projects with multiple manifest files in different directories
- **Role Accuracy**: Scores depend on manifest quality and project structure; toy projects are flagged with lower confidence

## Configuration

### Environment Variables (.env)
```
GITHUB_TOKEN=github_pat_xxxx...      # GitHub personal access token
GITHUB_API_BASE_URL=https://api.github.com
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=password
DB_NAME=algohire
PORT=3000
```

## Future Improvements

- **V2 Infrastructure Scoring**: Detect Docker, Kubernetes, cloud configs (AWS, Azure) for DevOps roles
- **Multi-File Analysis**: Support monorepos with multiple manifests
- **Real-time Caching**: Switch to Redis for distributed caching
- **Private Repository Support**: OAuth flow for private repo access
- **More Roles**: Add ML Ops, DevOps, Cloud Engineer, Data Engineer roles with refined weights
- **Historical Tracking**: Store analysis history per user/team
- **Webhook Integration**: Trigger analysis on GitHub push events
- **Custom Role Definitions**: Allow teams to create custom role profiles
- **Bulk Analysis**: Analyze multiple repos in one request
- **Export Reports**: Generate PDF/CSV role summaries

## License

MIT
