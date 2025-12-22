# CLAUDE.md vs cc-sdd: Time Savings Comparison
# CLAUDE.md と cc-sdd の時間節約比較

---

## 📊 Quick Summary / クイックサマリー

| Aspect | CLAUDE.md Only | cc-sdd (SDD) | Savings |
|--------|----------------|--------------|---------|
| **Setup Time** | 10-30 min | 30 sec + 10-30 min customization | Similar |
| **Feature Planning** | Days | Hours | **70-80%** |
| **Context Repeating** | Every session | Once (auto-loaded) | **90%+** |
| **Requirements Definition** | Manual | AI-generated templates | **60-70%** |
| **Design Documentation** | Manual | Auto-generated with diagrams | **70-80%** |
| **Task Breakdown** | Manual | AI decomposition with dependencies | **50-60%** |
| **Implementation Consistency** | Variable | Spec-first guaranteed | **Higher Quality** |

---

## 🎯 What Each Tool Does / 各ツールの役割

### CLAUDE.md
**Purpose**: Project memory and context persistence
- Stores project structure, commands, coding standards
- Auto-loaded at start of every Claude Code session
- Eliminates repetitive explanations
- Simple setup, minimal overhead

### cc-sdd (Spec-Driven Development)
**Purpose**: Full development lifecycle management
- Structured workflow: Requirements → Design → Tasks → Implementation
- Kiro-style commands (`/kiro:spec-init`, `/kiro:spec-requirements`, etc.)
- Auto-generates documentation with Mermaid diagrams
- Team-aligned templates
- Includes CLAUDE.md functionality + much more

---

## ⏱️ Time Savings Analysis / 時間節約分析

### Scenario 1: New Feature Development (Greenfield)

**Without any tool (Traditional):**
```
Requirements Meeting:     4-8 hours
Design Documentation:     8-16 hours  
Task Planning:            4-8 hours
Context Switching:        2-4 hours/day
Total Planning Phase:     18-36 hours (3-5 days)
```

**With CLAUDE.md only:**
```
Requirements:             Manual (4-8 hours)
Design:                   Manual (6-12 hours)
Task Planning:            Manual (3-6 hours)
Context Switching:        Eliminated (-90%)
Total:                    13-26 hours (2-3 days)
Savings:                  ~30%
```

**With cc-sdd:**
```
/kiro:spec-init:          5 minutes
/kiro:spec-requirements:  30-60 minutes (AI-assisted)
/kiro:spec-design:        30-60 minutes (AI-generated)
/kiro:spec-tasks:         15-30 minutes (AI decomposition)
Total:                    2-4 hours
Savings:                  ~70-80%
```

### Scenario 2: Brownfield Enhancement (Existing Code)

| Phase | CLAUDE.md | cc-sdd | Time Saved |
|-------|-----------|--------|------------|
| Understand existing code | 1-2 hours | `steering` + `validate-gap` in 30 min | **60%** |
| Plan changes | 2-4 hours | `spec-design` in 30 min | **75%** |
| Break into tasks | 1-2 hours | `spec-tasks` in 15 min | **80%** |

---

## 🔄 Workflow Comparison / ワークフロー比較

### CLAUDE.md Workflow
```
1. Create CLAUDE.md (once)
2. Start Claude Code
3. Context auto-loaded ✓
4. Work on tasks (manual planning)
5. Repeat steps 2-4
```

### cc-sdd Workflow
```
1. npx cc-sdd@latest --claude --lang ja  (30 sec install)
2. /kiro:spec-init "Feature description"
3. /kiro:spec-requirements <project-name>  → requirements.md generated
4. /kiro:spec-design <project-name>        → design.md + diagrams
5. /kiro:spec-tasks <project-name>         → tasks.md with dependencies
6. /kiro:spec-impl <task-id>               → Implement per spec
7. Context and specs persist across sessions
```

---

## 📈 Real-World Time Estimates

### Example: Photo Album Feature

**Traditional (No Tools):**
- Meetings: 6 hours
- Documentation: 12 hours
- Planning: 6 hours
- **Total: 24 hours (3+ days)**

**CLAUDE.md Only:**
- Setup once: 30 min
- Planning (manual): 8-12 hours
- **Total: 8-12 hours (1-2 days)**

**cc-sdd:**
- Install: 30 seconds
- `/kiro:spec-init`: 5 min
- `/kiro:spec-requirements`: 30 min (AI generates 15 EARS-format requirements)
- `/kiro:spec-design`: 30 min (Architecture + Mermaid diagrams)
- `/kiro:spec-tasks`: 15 min (12 tasks with dependencies)
- **Total: ~90 minutes**

**🎯 Time Savings: 24 hours → 1.5 hours = 93.75%**

---

## 🤔 When to Use What / 使い分け

### Use CLAUDE.md Only When:
- ✅ Small projects or personal projects
- ✅ Simple, repetitive tasks
- ✅ You mainly need context persistence
- ✅ Minimal documentation requirements
- ✅ Solo development

### Use cc-sdd When:
- ✅ Team projects requiring alignment
- ✅ Production-quality documentation needed
- ✅ Complex features requiring planning
- ✅ Need approval gates (requirements → design → tasks)
- ✅ Want AI-generated architecture diagrams
- ✅ Multiple developers working in parallel
- ✅ Enterprise/client projects

### Use Both Together (Recommended for Production):
```
cc-sdd includes and extends CLAUDE.md functionality
Install cc-sdd → Get both benefits
```

---

## 💰 Cost-Benefit Analysis / コスト・ベネフィット分析

### Developer Time (Hourly Rate: $50-100/hour)

| Feature | Traditional | CLAUDE.md | cc-sdd | Savings (cc-sdd) |
|---------|-------------|-----------|--------|------------------|
| Small Feature (8h) | $400-800 | $300-600 | $100-200 | **$300-600** |
| Medium Feature (24h) | $1,200-2,400 | $600-1,200 | $150-300 | **$900-2,100** |
| Large Feature (80h) | $4,000-8,000 | $2,000-4,000 | $500-1,000 | **$3,000-7,000** |

### Key Statistics (from cc-sdd claims):
- **70% reduction** in development time spent on meetings & documentation
- **Hours instead of weeks** for feature planning
- **50% reduction** in code review time (due to consistent specs)

---

## 🚀 Recommendation / おすすめ

### For Your Use Case (Construction Document + AI Development):

1. **Start with CLAUDE.md** for immediate context benefits
2. **Add cc-sdd** when you need:
   - Structured feature development
   - Documentation for clients/teams
   - Quality assurance through spec-first approach

### Quick Install:
```bash
# Japanese version for Claude Code
npx cc-sdd@latest --claude --lang ja

# Then use commands:
/kiro:spec-init "機能の説明"
/kiro:spec-requirements <project-name>
/kiro:spec-design <project-name> -y
/kiro:spec-tasks <project-name> -y
/kiro:spec-impl <task-id>
```

---

## 📚 Resources

- [cc-sdd GitHub](https://github.com/gotalab/cc-sdd)
- [CLAUDE.md Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Spec-Driven Guide (日本語)](https://zenn.dev/gotalab/articles/3db0621ce3d6d2)
