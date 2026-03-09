# Claude Code — Project Instructions

## Working Directory Layout
```
c:/playground/try-proliferate/
├── CLAUDE.md                  ← you are here (auto-loaded every session)
├── .claude/                   ← persistent knowledge base (read this first)
│   ├── knowledgebase/         ← architecture, tech stack, decisions
│   ├── planning/              ← roadmap, MVP features, SaaS requirements
│   ├── standards/             ← coding standards, git workflow
│   ├── debugger/              ← known issues, troubleshooting
│   └── context/               ← setup log, next steps
├── repo/                      ← cloned source code (proliferate-ai/proliferate)
└── prompts/                   ← session prompts + summaries
```

## Session Workflow (IMPORTANT — follow every session)

### How Sessions Work
1. User feeds a prompt file: `prompts/NN - CLAUDE_CODE_PROMPT.md`
2. Claude reads it, does the work
3. Claude creates: `prompts/NN - CLAUDE_CODE_PROMPT_summary.md`
4. Claude presents **3 options** for next session in chat
5. User picks A/B/C
6. Claude creates: `prompts/NN+1 - CLAUDE_CODE_PROMPT.md`
7. User reviews/edits the next prompt, then feeds it to a **new chat window**

### File Naming Convention
- Prompts: `prompts/01 - CLAUDE_CODE_PROMPT.md`, `prompts/02 - ...`, etc. (zero-padded)
- Summaries: `prompts/01 - CLAUDE_CODE_PROMPT_summary.md`

### At the Start of Every Session
1. Read the prompt file for this session
2. Skim `.claude/context/NEXT_STEPS.md` — current project state
3. Skim `.claude/debugger/KNOWN_ISSUES.md` — active blockers
4. Then execute the prompt objectives

### At the End of Every Session
1. Update `.claude/context/SETUP_LOG.md` — append what was done
2. Update `.claude/context/NEXT_STEPS.md` — reflect current state
3. Log any new issues in `.claude/debugger/KNOWN_ISSUES.md`
4. Update relevant `.claude/knowledgebase/` files if decisions were made
5. Create summary file: `prompts/NN - CLAUDE_CODE_PROMPT_summary.md`
6. Present 3 next session options in chat
7. After user picks, create `prompts/NN+1 - CLAUDE_CODE_PROMPT.md`

---

## Communication Style
- **Chat**: short (1-3 sentences), bullets over paragraphs
- **Detail**: goes in `.claude/` files, not chat
- **Decisions needed**: present as MCQ (A/B/C) — wait for answer before proceeding
- **Blockers**: state problem + proposed fix in 1-2 sentences

---

## Project: Proliferate SaaS
- **Source repo**: `repo/` — MIT licensed, https://github.com/proliferate-ai/proliferate
- **Goal**: Commercialize into production SaaS
- **Package manager**: pnpm only (never npm or yarn)
- **Sandbox provider**: E2B (decided Session 01)
- **Full context**: `.claude/knowledgebase/ARCHITECTURE.md`, `.claude/knowledgebase/TECH_STACK.md`
- **Gaps to close**: `.claude/planning/SAAS_REQUIREMENTS.md`
- **Roadmap**: `.claude/planning/ROADMAP.md`
