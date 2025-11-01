# Portfolio Automation System

## Overview
Portfolio Sync Automation is a unified system that collects student profile data from a frontend form, enriches it using public coding platforms, extracts certificates from Google Drive, and stores the structured output in Google Sheets while displaying a real-time consolidated portfolio on the frontend.

It eliminates the need for:

- Manually collecting portfolio information  
- Checking coding platforms individually  
- Searching certificate folders  
- Maintaining inconsistent spreadsheets  

The system ensures that every student portfolio is complete, standardized, and verified. Instead of filling long forms or manually compiling data, students submit one form, and the system builds a complete portfolio automatically.

---

## Key Data Sources
| Source | Meaning |
|--------|---------|
| Form Input | Basic info, socials, clubs |
| LeetCode API | Stats + problem progress |
| Codeforces API | Rating + contest history |
| GitHub API | Project history |
| Google Drive | Certificates / achievements |
| Google Sheets | Storage + lookup cache |

---

## Output Structure
Each submission generates a clean, normalized portfolio object:

```json
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
- Editable & re-submittable  

---

## High-Level Flow

1. User fills the **Portfolio Sync Form**
2. Frontend sends structured JSON to **n8n Webhook**
3. n8n triggers **parallel processing pipelines**
4. Each branch produces a normalized data block
5. Data blocks are merged and:
   - Stored in Google Sheets
   - Returned to frontend for live portfolio display

---

## Architecture Diagram

```
Frontend (React) 
      ↓ POST JSON
Webhook Trigger (n8n)
      ↓
Parallel Branches -----------------------------------------------------------┐
  Profile Extractor      LeetCode Stats         Codeforces API               |
  GitHub Project Fetch   Certificate Parser (Drive Folder Scan)              |
      ↓                                                                      |
  Normalized Output Blocks (5)                                               |
      ↓                                                                      |
Google Sheets (Append / Update)                                              |
      ↓                                                                      |
Webhook → Frontend Display ← mergePortfolioPayload --------------------------┘
```

---

## NEW MODULE — Weekly Progress Mail Automation

This workflow sends each student a **weekly progress feedback email** based on Codeforces & LeetCode performance.

### Code Node (Final Email Body Generator)

```javascript
const cf = items[0].json.output;
const name = items[1].json.Name;
const email = items[1].json["Email Id"];
const lc = items[2].json.output;

const emailBody = `
<h2>Hi ${name}, here's your weekly coding progress 🚀</h2>

<p><b>🔹 Codeforces Feedback:</b><br>${cf}</p>

<p><b>🔹 LeetCode Feedback:</b><br>${lc}</p>

<p>You’re improving — keep pushing! Consistency compounds 🌱⚡</p>
`;

return [
  { json: { email, emailBody } }
];
```

---

## Data Storage Format (Google Sheets)

### LeetCode Sheet
| Handle | Total Solved | Easy | Medium | Hard | AcceptanceRate |

### GitHub Sheet
| Handle | Name | URL | Description | Tech Stack |

### Codeforces Sheet
| Handle | Rating | Max Rating | Contests Attended |

### Certificates Sheet
| FileId | AwardedBy | For |

---

## Error Handling
| Failure | Recovery |
|--------|----------|
| API unavailable | Section skipped, workflow continues |
| Drive access denied | Certificate section empty |
| Sheets write failure | Logged for retry |

---

## Extensibility
You can add:
- Hackerrank  
- Kaggle  
- Devpost  
- Blog Aggregation (Medium / Hashnode)

---

## Status
✅ Fully Functional  
✅ Tested with multiple student submissions  
🚀 Live syncing supported  
