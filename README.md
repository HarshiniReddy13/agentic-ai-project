# Portfolio Automation

## Overview
Portfolio Sync Automation is a unified system that collects student profile data from a frontend form, enriches it using public coding platforms, extracts certificates from Google Drive, and stores the structured output in Google Sheets while displaying a real-time consolidated portfolio on the frontend.

It eliminates the need for:
- Manually collecting portfolio information
- Checking coding platforms individually
- Searching certificate folders
- Maintaining inconsistent spreadsheets

The system ensures that every student portfolio is complete, standardized, and verified. Instead of filling long forms or manually compiling data, students submit one form, and the system builds a complete portfolio automatically.

## Key Data Sources
- **Form Input (Frontend)**: Personal details, socials, clubs
- **LeetCode API / scraped stats**: Competitive programming performance
- **Codeforces API**: Rating + contest experience
- **GitHub API**: Meaningful project history
- **Google Drive Folder Link**: Certificates + award extraction
- **Google Sheets**: Persistent storage + lookup cache

## Output Structure
Each submission generates a clean, normalized portfolio object:
```
{
  "profile": { ... },
  "leetcode": { ... },
  "codeforces": { ... },
  "projects": [ ... ],
  "certificates": [ ... ]
}
```

This object is:
- Stored in Google Sheets
- Sent to the Web Frontend
- Displayed as a structured portfolio preview
- Editable and re-submittable

## High-Level Flow
1. User fills the Portfolio Sync Form → submits.
2. Frontend sends structured JSON to n8n Webhook.
3. n8n processes five branches in parallel:
   - Profile
   - LeetCode
   - Codeforces
   - GitHub Projects
   - Certificates (Google Drive)
4. Each branch produces a normalized output block:
```
{ section: "<name>", <sectionData>: {...} }
```
5. Outputs are combined and stored + displayed.

## Architecture
```
Frontend (React) 
    ↓ POST JSON
n8n Webhook Trigger
    ↓
Parallel Branches --------------------------------------------------------┐
  Profile Formatter        LeetCode Stats      Codeforces API Call       GitHub Projects Fetch
  Certificates Parser (Drive Folder Scan)                                  │
    ↓                                                                      │
   Normalized Data Blocks (5)                                              │
    ↓                                                                      │
Google Sheets Append/Update (Persistent Record + Past Runs Cache)          │
    ↓                                                                      │
Webhook → Frontend Response → mergeWebhookPayload → PortfolioDisplay UI ←-┘
```

## Nodes and Responsibilities

### Webhook Intake & Field Normalization
| Node | Purpose |
|------|---------|
| Webhook (POST /portfolio) | Receives portfolio submission |
| Edit Fields | Cleans and standardizes input fields |

### Branch 1 — Certificates Extraction
| Node | Function |
|------|----------|
| List Certificates | Fetches certificate files from Google Drive |
| Download File | Downloads certificate images/PDFs |
| HTTP Request (OCR/Metadata) | Extracts title + issuer |
| Code (JS) | Normalizes into `{ id, awardedBy, for }` |
| Certificate Sheet | Stores structured certificates |
| Code (JS) 2 | Outputs `"certificates"` block |

### Branch 2 — LeetCode Stats
| Node | Function |
|------|----------|
| LeetCode Fetch | Retrieves user stats |
| LeetCode Sheet | Stores stats |
| Code (JS 3) | Outputs `"leetcode"` section |

### Branch 3 — LinkedIn Profile
| Node | Function |
|------|----------|
| LinkedIn Scrape | Extracts skills, education, experience |
| LinkedIn Sheet | Stores profile data |
| Code (JS 4) | Outputs `"profile"` section |

### Branch 4 — GitHub Projects
| Node | Function |
|------|----------|
| GitHub Fetch | Gets repositories list |
| Git Sheet | Stores project info |
| Code (JS 5) | Outputs `"projects"` list |

### Branch 5 — Codeforces Stats
| Node | Function |
|------|----------|
| Codeforces API | Fetches rating and history |
| CF Sheet | Logs rating |
| Code (JS 6) | Outputs `"codeforces"` block |

### Final Aggregation
| Node | Function |
|------|----------|
| Merge (Append Inputs) | Combines all sections |
| Respond to Webhook | Returns structured portfolio |

## Key Code Patterns
```
payloadArray.forEach((item) => {
  if (item.section && item[item.section]) {
    result[item.section] = item[item.section];
  }
});
```

### Drive File Viewer Link Format
```
https://drive.google.com/file/d/${fileId}/view
```

## Running the System
1. User submits form.
2. n8n runs parallel processing.
3. Sheets updated, UI refreshes.
4. User can revise and re-submit anytime.

## Data Model Summary (Google Sheets)

### LeetCode Sheet
| Handle | Total Solved | Total Questions | Easy | Medium | Hard | AcceptanceRate | Ranking |

### GitHub Sheet
| Handle | Name | url | Description | Created at | Updated at | Technology |

### Codeforces Sheet
| handle | rating | maxRating | lastOnlineTimeSeconds | registrationTimeSeconds |

### Certificates Sheet
| id | awardedBy | for |

### LinkedIn Sheet
| full name | skills | experience | education |

## Error Handling
| Failure | Action |
|--------|--------|
| API fetch fails | Section omitted, others still render |
| Drive access fails | Certificate section empty |
| Sheets write error | Execution logged for retry |

## Performance
- Sheets caching prevents re-fetching
- Parallel execution reduces processing time
- UI supports partial display when data missing

## Extensibility
Add optional integrations:
- Hackerrank
- Kaggle
- Devpost
- Medium Blog Feed
