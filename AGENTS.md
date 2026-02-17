# AGENTS.md — Development Best Practices

This document gives AI agents and developers shared guidance for this project. Follow these practices when writing, reviewing, or modifying code. Each section is grounded in well-established industry sources; see [References](#references) for links.

---

## Security

Minimize risk from vulnerabilities and misuse. Security is a baseline, not an afterthought.

- Never commit secrets, API keys, or credentials to the repository; use environment variables or a secrets manager. [12-Factor: Config]
- Validate and sanitize all user input; assume input is untrusted. Enforce validation at a trusted service layer; do not rely on client-side validation alone. [OWASP; ASVS V2.2]
- Apply the principle of least privilege for permissions, accounts, and network access. [OWASP]
- Keep dependencies updated and address known vulnerabilities promptly. [OWASP]
- Use parameterized queries or prepared statements; never concatenate user input into SQL or shell commands. Protect against OS command injection via parameterized calls or safe encoding. [OWASP; ASVS V1.2]
- Encode output for the correct context (HTML, URL, JavaScript, headers) so structure cannot be changed by untrusted data; decode input to a canonical form only once, before validation. [ASVS V1.1, V1.2]
- Sanitize untrusted HTML (e.g. from WYSIWYG editors) with a secure library; avoid eval() or dynamic code execution with user input. [ASVS V1.3]
- Protect against SSRF: validate URLs and targets against an allowlist of protocols, domains, and ports before calling external services. [ASVS V1.3]
- Configure XML parsers restrictively; disable external entity resolution to prevent XXE. Use safe deserialization (allowlists or restricted types) for untrusted data. [ASVS V1.5]
- Use secure cookie attributes: Secure, HttpOnly for session tokens, and SameSite as appropriate; prefer __Host- or __Secure- prefix where applicable. [ASVS V3.3]
- Enforce HTTPS (e.g. HSTS), set Content-Type correctly on responses, and use anti-CSRF tokens or safe methods (POST/PUT/PATCH/DELETE) for state-changing requests. [ASVS V3.4, V3.5; V4.1]
- Validate file uploads: limit size, check extension and content (e.g. magic bytes), store outside web root with safe names, and serve with safe Content-Disposition/headers. [ASVS V5]
- Do not expose sensitive data or stack traces to clients; log security-relevant events (auth failures, access denials) for auditing. [ASVS V16]
- Treat backing services (databases, caches, queues) as attached resources with clear contracts. [12-Factor: Backing services]

---

## Quality

Ensure correctness and confidence to change code through testing and review.

- Prefer the test pyramid: many fast unit tests, some integration tests, few end-to-end tests. [Fowler: Practical Test Pyramid]
- Test observable behavior, not implementation details; avoid tests that break on refactors. [Fowler: Practical Test Pyramid]
- Structure tests as Arrange, Act, Assert (or Given, When, Then). [Fowler: Practical Test Pyramid]
- Do not test trivial code (e.g. simple getters/setters); focus on non-trivial paths and edge cases. [Fowler: Practical Test Pyramid]
- Require meaningful code review for all changes; review for design, correctness, complexity, tests, and style. [Google: Code Review]
- Keep changes small and self-contained so they are easier to review and reason about. [Google: Code Review]

---

## Observability

Make the system understandable and debuggable in production.

- Emit structured logs (e.g. JSON) with consistent fields; avoid free-form prose only. [12-Factor: Logs]
- Treat logs as event streams; do not write to or manage log files from application code. [12-Factor: Logs]
- Expose health and readiness endpoints for orchestration and load balancers.
- Use correlation IDs or trace IDs across services where applicable for request tracing.
- Prefer metrics and alerts for critical flows rather than relying only on log inspection.

---

## Maintainability

Keep the codebase readable, understandable, and easy to change over time.

- Use clear, consistent naming; prefer names that reveal intent. [Clean Code]
- Favor single responsibility: one clear reason for a module or function to change.
- Document the "why" (decisions, constraints) more than the "what"; avoid redundant comments.
- Declare and isolate dependencies explicitly; avoid implicit or global dependency injection. [12-Factor: Dependencies]
- Prefer domain logic in the domain model over anemic data structures with logic only in services. [Fowler: Test Pyramid sample]

---

## Documentation and diagrams

Create and maintain documentation and diagrams so the system’s structure, decisions, and behavior are clear to humans and agents. Keep docs close to the codebase and easy to update.

- **Place documentation in a `docs` directory.** Put all project documentation (architecture, ADRs, runbooks, API notes) under a root-level `docs` folder so it has a single, predictable location.
- **Use Markdown for documentation.** Write prose and structured docs in Markdown (`.md`) for readability in editors, version control, and common documentation platforms.
- **Use Mermaid for diagrams.** Create diagrams in Mermaid syntax (e.g. in `.md` files or `.mmd` files) so they are text-based, versionable, and render in many tools (GitHub, GitLab, IDEs, static site generators).
- **Prefer C4 model for architecture diagrams.** Use the C4 model to describe the system at different levels of detail:
  - **Level 1 (System Context):** system and its users and external systems.
  - **Level 2 (Container):** high-level building blocks (applications, data stores).
  - **Level 3 (Component):** components inside a container where useful.
  - **Level 4 (Code):** only when it adds real value (e.g. for critical modules).
- Add diagrams when they clarify structure or flow (e.g. architecture, deployment, or key workflows); avoid diagrams that duplicate what the code or a short description already makes obvious.
- Keep diagrams and docs in sync with the code; update them when making significant structural or behavioral changes.

---

## Reliability

Design for graceful failure and recovery so the system degrades predictably.

- Support graceful shutdown: finish in-flight work, close connections, then exit. [12-Factor: Disposability]
- Aim for fast startup so processes can be spun up quickly and scaled horizontally. [12-Factor: Disposability]
- Prefer stateless processes so any instance can serve any request. [12-Factor: Processes]
- Design critical operations to be idempotent where possible to allow safe retries.
- Define explicit error boundaries and failure modes; avoid silent failures or unhandled exceptions.

---

## Operability

Make deployment and operations predictable and repeatable.

- Store configuration in the environment; do not embed environment-specific values in code. [12-Factor: Config]
- Strictly separate build, release, and run stages; do not mutate code at runtime. [12-Factor: Build, release, run]
- One codebase tracked in version control, many deploys (e.g. staging, production). [12-Factor: Codebase]
- Keep development, staging, and production as similar as possible to reduce deployment surprises. [12-Factor: Dev/prod parity]
- Export services via port binding; the app is self-contained and binds to a port to serve traffic. [12-Factor: Port binding]

---

## Collaboration

Make code review and version history effective for the whole team.

- Prefer small, self-contained changes (pull requests / changelists) with a clear purpose. [Google: Code Review]
- Write clear commit messages and PR descriptions: what changed and why. [Google: CL Author Guide]
- Review for design, correctness, complexity, tests, naming, and style; give constructive feedback. [Google: Code Reviewer Guide]
- Respond to review feedback professionally; iterate until the change meets the team bar. [Google: Handling Pushback]

---

## Performance

Avoid obvious bottlenecks and unnecessary work; measure before optimizing.

- Avoid N+1 query patterns; use batching, joins, or bulk APIs where appropriate.
- Use asynchronous or non-blocking I/O for I/O-bound work to avoid blocking threads.
- Introduce caching only where there is a clear benefit; define invalidation and consistency rules.
- Profile or measure before major optimizations; avoid premature optimization.

---

## Accessibility

Design and build so people with disabilities and situational limitations can perceive, operate, and understand the product. Accessibility benefits everyone and is required by law in many contexts. [W3C WAI: Introduction]

- **Perceivable:** Provide text alternatives for non-text content (e.g. meaningful `alt` for images); use null/empty alt for purely decorative images. [WCAG 1.1.1]
- **Perceivable:** Do not convey information by color alone; use icons, text, or pattern in addition. [WCAG 1.4.1]
- **Operable:** Make all functionality available from the keyboard; ensure visible focus styles and a logical focus order. [W3C WAI: Keyboard; WCAG 2.1.1, 2.4.7]
- **Operable:** Do not disable viewport zoom; support text resizing (e.g. to 200%). [WCAG 1.4.4; A11Y Project]
- **Understandable:** Use plain language where possible; ensure link and button text is unique and descriptive (avoid "click here"). [A11Y Project: Content]
- **Understandable:** Associate form inputs with labels; surface errors clearly and associate them with the relevant field. [WebAIM: Forms; A11Y Project: Forms]
- **Robust:** Use valid, semantic HTML; use landmark elements and a logical heading hierarchy (one `h1` per page, no skipped levels). [WebAIM: Semantic Structure; A11Y Project: Headings]
- **Media:** Provide captions for video and transcripts for audio; avoid autoplay and flashing content that can trigger seizures. [W3C WAI: Transcripts; WCAG 1.2.2, 2.3.1]
- Evaluate accessibility early and throughout development; combine automated checks with knowledgeable human evaluation. [W3C WAI: Evaluating Accessibility]

---

## Instructions for agents (task workflow)

When carrying out coding or change tasks, follow these workflow steps so work is traceable, reviewable, and safe to iterate on.

- **Use a pull request (PR) for changes.** Do not merge directly to the main/default branch. Open a branch, make changes, and propose them via a PR so they can be reviewed and integrated in a controlled way.
- **Commit early and often.** Make small, logical commits with clear messages (what changed and why). This keeps history readable and makes it easier to revert or bisect if needed.
- **Push changes regularly.** Push your branch to the remote after meaningful commits so work is backed up, visible to others, and ready for CI and review. Do not leave large amounts of unpushed work on a single branch.
- **Run tests and linter before pushing.** Run the project’s test suite and any lint/format checks locally so the branch is in a passing state before CI runs and before requesting review.
- **Keep the branch up to date with the target.** Before opening or updating a PR, rebase or merge from the default branch (e.g. `main`) so the branch is current and merge conflicts are minimized.
- **Use a descriptive branch name.** Name branches so their purpose is clear (e.g. `feature/add-login`, `fix/typo-in-readme`, `docs/update-api`) to make history and PR lists easier to scan.
- **Write a clear PR description.** In the PR, state what changed, why, and how to verify (steps or checklist). Link to issues or context when relevant so reviewers can evaluate without guessing.
- **Keep PRs small and focused.** Prefer one logical change per PR when possible; split large work into several PRs so each is easier to review, discuss, and merge.
- **Avoid force-pushing to shared branches.** Do not force-push to the default branch or others’ branches. Force-push only to your own feature branch when necessary (e.g. after a rebase), and only if no one else is building on that branch.

---

## References

- **12-Factor App** — [https://12factor.net/](https://12factor.net/) (config, dependencies, backing services, build/release/run, processes, port binding, disposability, logs, dev-prod parity)
- **OWASP** — OWASP Secure Coding Practices / Developer Guide — [https://owasp.org/www-project-developer-guide/](https://owasp.org/www-project-developer-guide/)
- **OWASP ASVS** — Application Security Verification Standard (v5.0) — [https://owasp.org/www-project-application-security-verification-standard/](https://owasp.org/www-project-application-security-verification-standard/) — requirements for secure development and verification; [GitHub (v5.0.0)](https://github.com/OWASP/ASVS/tree/v5.0.0)
- **Martin Fowler — Practical Test Pyramid** — [https://martinfowler.com/articles/practical-test-pyramid.html](https://martinfowler.com/articles/practical-test-pyramid.html)
- **Google Engineering Practices** — Code review (author and reviewer guides) — [https://google.github.io/eng-practices/](https://google.github.io/eng-practices/)
- **W3C WAI — Introduction to Web Accessibility** — [https://www.w3.org/WAI/fundamentals/accessibility-intro/](https://www.w3.org/WAI/fundamentals/accessibility-intro/)
- **W3C WAI — WCAG 2.x Quick Reference** — [https://www.w3.org/WAI/WCAG21/quickref/](https://www.w3.org/WAI/WCAG21/quickref/)
- **The A11Y Project — Checklist** — [https://www.a11yproject.com/checklist/](https://www.a11yproject.com/checklist/)
- **WebAIM** — Articles and techniques (semantic structure, alt text, keyboard, forms) — [https://webaim.org/articles/](https://webaim.org/articles/)
- **C4 model** — Model for visualising software architecture (Context, Container, Component, Code) — [https://c4model.com/](https://c4model.com/)
