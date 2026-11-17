# 0001: Modular monolith with ports and adapters

- **Date and status:** 14/11/2026, approved by Victor.
- **Context:** Docenda is a web platform built by one person, with a short first deadline. The requirements demand that nothing is tied to a vendor (RNF-30), that auth, AI usage and cost, misuse checks, audit and export are reusable modules (RNF-31), and that the whole system is built with TDD (RNF-09). The market guidance is to start with a monolith and split later only for a measured reason ([Fowler, MonolithFirst](https://martinfowler.com/bliki/MonolithFirst.html)). Ports and adapters keeps business rules isolated from technology ([Cockburn](https://alistair.cockburn.us/hexagonal-architecture/); [AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/hexagonal-architectures/best-practices.html)).
- **Decision:**
  1. **Modular monolith:** one deployable application, split into modules by business subject, each with a public entry point (`index`) and private internals.
  2. **Ports and adapters:** ports only at the external edges (AI, e-mail, database, file storage, social login). Each port has a real adapter and a fake one for tests.
  3. **Dependency injection:** only `entrypoints/` (the composition root of the web server and the queue worker) chooses adapters, based on configuration (RNF-32).
  4. **Reusable modules as folders (option B):** `reusable/` lives inside the API. Its modules never know Docenda and can be promoted to packages later.
  5. **Boundary rules checked in CI** with dependency-cruiser (18.7.2, MIT): a module is used only through its `index`; `reusable/` never imports `modules/`; `domain/` never imports frameworks or vendor SDKs; only `entrypoints/` imports `providers/`.
  6. **Language:** code, commits, README, API docs and ADRs in English. UI text in Portuguese. Study and planning material in this repository in Portuguese.
  7. **Versions:** stable releases only, LTS when there is one.
- **Initial layout (API):** `src/modules/` (plans, class context, BNCC, sources, review), `src/reusable/` (auth, ai-usage, misuse, audit, export), `src/providers/` (ai, email, storage), `src/entrypoints/` (web, worker).
- **Alternatives considered:**
  - Plain layers (controllers, services, repositories): fastest start, but vendor SDKs leak into the rules and break RNF-30.
  - Each reusable module as a workspace package (option C): reuse across projects from day one, but each package needs its own config; not worth it for v1.
  - Microservices: network cost and premature boundaries; ruled out.
- **Consequences:**
  - Pros: rules testable with fakes (TDD), switching vendor means switching an adapter, and modules can be extracted later.
  - Cons: more files and discipline at the start; the boundary depends on a tool, not on the language.
  - Cost: no money, a few extra hours of setup.
- **Update (16/11/2026, planning review):**
  - Item 2: the database keeps its interfaces but gets no fake adapter; repositories are tested against the real PostgreSQL. The inside of a module and the new `src/shared/` folder are defined in [0012](0012-transactions-repositories-and-module-layout.md).
  - Item 6: the README and the product documents (purpose, requirements, domain, backlog) are in Portuguese, the language of the people the product is for. Code, commits, API docs and ADRs stay in English. *Replaced by the note of 17/11 below.*
- **Update (17/11/2026, owner's decision).** Item 6 had no clear line ("README in English" but "planning material in Portuguese"), and the note of 16/11 made it worse by sending "product documents" to Portuguese without saying where product ends. All twelve commits up to 16/11 were in Portuguese. The rule is now **by reader**: everything in this repository that developers or the coding agent read is in English, including every document in `docs/` and the instructions of the prompts; everything teachers see (screen text, AI output, the exported file, the BNCC catalog, evaluation case content) is in Brazilian Portuguese. Reasons: one question decides every case; English is the usual language of software repositories; and documents the agent reads on every task cost fewer tokens in English (to be measured with the token counting endpoint in F03). The table lives in `CLAUDE.md`, and slice F00 adds a CI check that rejects commit titles with Portuguese accents.
