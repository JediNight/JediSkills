# Key Statistics and Sources

## Why This Matters

- AI code produces **1.7x more issues** and **2.74x more security vulnerabilities** (CodeRabbit, 470 PRs)
- **20% of vibe-coded apps** contain discoverable security risks (Wiz, 5,600 apps)
- **21.7% of AI-suggested package names** are hallucinations (slopsquatting risk)
- **45% of developers** say debugging AI code takes longer than writing without AI (Stack Overflow)
- Developers are **19% slower** with AI tools but **believe** they're 24% faster (METR)
- **4x increase in code cloning** and **60% decline in refactoring** in AI-heavy repos (GitClear)
- Mutation testing reveals AI tests with **100% line coverage** but **near-zero mutation scores** (anecdotal, multiple Reddit practitioners — directional, not peer-reviewed)
- Practitioners report **significantly higher maintenance costs** for fully vibe-coded production systems (anecdotal — quantified estimates vary widely)
- **87% of AI PRs** contained at least one vulnerability (DryRun Security, 38 scans across Claude/Codex/Gemini)
- **45% of AI-generated code** contains security vulnerabilities (Georgetown CSET)
- AI-assisted devs scored **17% lower** on comprehension quizzes (Anthropic, 52 engineers)
- Code churn **doubled** and copy-pasted code rose **48%** in AI-assisted repos (CMU/GitClear, 807 repos)

## Sources

- [CodeRabbit AI vs Human Code Generation Report](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report)
- [Wiz - Security Risks in Vibe-Coded Apps](https://www.wiz.io/blog/common-security-risks-in-vibe-coded-apps)
- [Variant Systems - 10 Anti-Patterns](https://variantsystems.io/blog/vibe-code-anti-patterns)
- [METR - AI Developer Productivity Study](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)
- [Trend Micro - Slopsquatting](https://www.trendmicro.com/vinfo/us/security/news/cybercrime-and-digital-threats/slopsquatting-when-ai-agents-hallucinate-malicious-packages)
- [GitClear - AI Coding Quality Report](https://www.gitclear.com/coding_on_copilot_data_shows_ais_downward_pressure_on_code_quality) — 4x code cloning, 60% decline in refactoring
- [Stack Overflow Developer Survey 2025](https://survey.stackoverflow.co/2025/) — 45% say debugging AI code takes longer
- Reddit r/ExperiencedDevs, r/programming, r/ChatGPTCoding — practitioner-reported anti-patterns (2025-2026)
- [DryRun Security - AI Agent Vulnerability Study](https://www.globenewswire.com/news-release/2026/03/11/3253696/) — 87% of AI PRs contained vulnerabilities
- [Ox Security - 10 AI Code Anti-Patterns (300+ repos)](https://www.prnewswire.com/news-releases/ox-report-ai-generated-code-violates-engineering-best-practices-302592642.html)
- [Addy Osmani - Comprehension Debt](https://addyosmani.com/blog/comprehension-debt/) — the hidden cost of code nobody understands
- [Arcanum sec-context](https://github.com/Arcanum-Sec/sec-context) — 25+ security anti-patterns as LLM system prompt context
- [Engineering Pitfalls in AI Coding Tools (arXiv 2603.20847)](https://arxiv.org/html/2603.20847) — 3,800+ bugs across AI coding agents
