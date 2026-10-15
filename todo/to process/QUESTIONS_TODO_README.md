# QUESTIONS_TODO.json - Master Interview Questions Tracking

**Master file for tracking all interview questions to be added to the repository.**

---

## 📋 Overview

- **Total Questions**: 215
- **Format**: JSON (machine-readable)
- **Purpose**: Central tracking for review, deduplication, and batch addition
- **Accessible**: ✅ Version controlled in repo - all team members can access/update

---

## 🎯 Quick Start

### Clone/Access on Any Machine

```bash
git clone <repo-url>
cd anandu-interview-playbook
cat QUESTIONS_TODO.json
```

### Open & Edit

- **VS Code**: `File → Open File → QUESTIONS_TODO.json`
- **Notepad**: Right-click → Open with → Notepad
- **Online**: View on GitHub (raw content)

---

## 📊 File Structure

```json
{
  "metadata": {
    "version": "1.0",
    "created_date": "2026-10-14",
    "total_questions": 215,
    "status": "EXTRACTION_COMPLETE"
  },
  "categories": {
    "spring_boot_advanced": {
      "count": 75,
      "priority": "HIGH",
      "questions": [
        {
          "id": "SB_001",
          "question": "What happens when run() is called?",
          "source": "Spring Boot - Interview Questions.txt",
          "status": "NEW",
          "added_to_repo": false,
          "notes": "Application lifecycle"
        },
        ...
      ]
    },
    ...
  },
  "summary": {
    "total_new": 180,
    "total_review": 20,
    "total_duplicate": 15
  }
}
```

---

## 🏷️ Status Meanings

| Status | Meaning | Action |
|--------|---------|--------|
| `NEW` | Question not in repo yet | ✅ Add to repo |
| `REVIEW` | Manually check vs repo | 🔍 Verify if duplicate |
| `SIMILAR` | Possibly paraphrased | ❓ Check if same question |
| `DUPLICATE` | Already in repo | ❌ Skip |
| `REVIEW_COMPLETE` | Reviewed & decided | ✅ Ready for action |
| `REVIEW_ADDED` | Already added to repo | ✅ Done |

---

## 👥 Team Workflow

### **Step 1: Review Phase** (Anyone can do)

#### Local Machine:
```bash
# 1. Pull latest
git pull origin main

# 2. Open file
cat QUESTIONS_TODO.json | less

# 3. For each question, decide:
#    - Is it NEW? (Add to repo)
#    - Is it DUPLICATE? (Already in repo)
#    - Is it SIMILAR? (Need to verify)

# 4. Update status in file
#    Edit QUESTIONS_TODO.json:
#    "status": "REVIEW_COMPLETE"
```

#### On GitHub (web):
1. Navigate to `QUESTIONS_TODO.json`
2. Click edit icon
3. Review questions
4. Update status field
5. Commit changes with message: "Review: Marked SB_001-SB_010 as [status]"

---

### **Step 2: Extraction** (After review)

Once reviewed, extract only "NEW" questions:

```bash
# Filter to NEW questions (pseudo-code)
jq '.categories[].questions[] | select(.status=="NEW")' QUESTIONS_TODO.json
```

Then add to respective files:
- `docs/interview-questions/verbal/verbal-spring-boot-questions.md`
- `docs/interview-questions/verbal/verbal-microservices-questions.md`
- etc.

---

### **Step 3: Batch Processing**

#### For Answers (Delegate to AI):
```
Batch 1: Spring Boot Basic (20 Q) → AI fills in answers
Batch 2: Spring Boot Advanced (20 Q) → AI fills in answers
Batch 3: Microservices (15 Q) → AI fills in answers
...
```

Mark as `"added_to_repo": true` after adding.

#### For Learning Content:
Create placeholders in:
- `docs/resilience-patterns/`
- `docs/distributed-tracing/`
- `docs/kafka/`
- etc.

---

## 📈 Progress Tracking

### Current Status

```
✅ EXTRACTED: 215 questions
├─ NEW: 180 questions (ready to add)
├─ REVIEW: 20 questions (manual verification needed)
├─ SIMILAR: 10 questions (check for paraphrasing)
└─ ALREADY ADDED: 7 questions (design patterns)
```

### How to Update Progress

After reviewing a category, update the summary:
```json
"summary": {
  "total_new": 180,
  "total_review": 20,
  "total_duplicate": 15,
  "status_breakdown": {
    "NEW": 180,
    "REVIEW": 20,
    "SIMILAR": 10,
    "DUPLICATE": 5,
    "REVIEW_ADDED": 7,
    "REVIEW_COMPLETE": X  ← Update this
  }
}
```

---

## 🔄 Git Workflow for Team

### Pulling Latest Changes
```bash
git pull origin main
```

### Making Changes Locally
```bash
# 1. Update QUESTIONS_TODO.json with status changes
nano QUESTIONS_TODO.json

# 2. Stage changes
git add QUESTIONS_TODO.json

# 3. Commit with clear message
git commit -m "Review: Marked Spring Boot questions SB_001-SB_050 as REVIEW_COMPLETE"

# 4. Push to remote
git push origin main
```

### Resolving Conflicts
If multiple people edit simultaneously:
```bash
# Pull to see conflict
git pull origin main

# Resolve manually - keep all status updates (don't overwrite others' work)
# Each person's status marks should be preserved

# Commit merged version
git add QUESTIONS_TODO.json
git commit -m "Merge: Resolved conflicts - both reviews integrated"
git push origin main
```

---

## 💡 Best Practices

### ✅ DO

- **Commit frequently** with clear messages about what you reviewed
- **Include question IDs** in commit messages: "Reviewed SB_001-SB_050"
- **Pull before editing** to get latest changes
- **Be specific** in notes field if marking as SIMILAR/REVIEW
- **Update only your reviewed items** - don't overwrite others' reviews

### ❌ DON'T

- Edit the entire file at once (creates merge conflicts)
- Leave `status` as blank - always set to something
- Forget to commit progress (changes stay local)
- Change question text (only update status/notes)

---

## 📋 Sample Review Process

```json
// BEFORE Review
{
  "id": "SB_001",
  "question": "What happens when run() is called?",
  "status": "NEW",
  "added_to_repo": false,
  "notes": "Application lifecycle"
}

// AFTER Review
{
  "id": "SB_001",
  "question": "What happens when run() is called?",
  "status": "REVIEW_COMPLETE",
  "added_to_repo": false,
  "notes": "Application lifecycle - Already similar to Q2, but different angle. ADD."
}

// OR if duplicate
{
  "id": "SB_001",
  "question": "What happens when run() is called?",
  "status": "DUPLICATE",
  "added_to_repo": true,
  "notes": "Same as verbal-spring-boot-questions.md Q1 - Skip"
}
```

---

## 🚀 Next Steps

1. **Review** - Go through categories and mark status
2. **Extract** - Filter to "NEW" questions only
3. **Add to Repo** - Create/update question files in batches
4. **Add Answers** - Delegate to AI in parallel
5. **Add Learning** - Create learning content in parallel
6. **Link** - Cross-reference all topics

---

## ❓ Questions?

- **How to format questions?** → See `docs/interview-questions/verbal/` examples
- **How to add answers?** → See `CONTRIBUTING.md` in root
- **How to create learning content?** → See `docs/` structure

---

**Last Updated**: 2026-10-14  
**Maintainer**: Team  
**Format**: JSON (UTF-8)  
**Version Control**: ✅ Git repo
