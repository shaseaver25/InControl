# InControl — Lesson-Generation Handoff

This package is the **lesson-generation half** of InControl: the concept/learning graph, the variation model (how lessons adapt to ability, strength, and motivation), and the recommended system architecture for delivering it as a service your app connects to.

You (the app/UX engineer) **consume an API**; you never touch the graph directly. This package is the spec and the data behind that API.

## Read in this order

1. **`incontrol_architecture.html`** — open in a browser. The system architecture (hybrid: your app delivers the education/lesson flow, Trainerize hosts the workout, the lesson service feeds both). Click any box for its role and owner. **Start here.**
2. **`InControl Learning Graph — Playbook.md`** — the full reference. How the graph works, how lessons vary per learner (§4), the API/handoff boundary, and the build/maturity roadmap. The single source of truth.
3. **`incontrol_learning_graph.html`** — the concept graph (133 concepts, 177 prerequisite links across 9 domains). Prerequisites flow left → right. Click a node to see what it depends on and unlocks.
4. **`incontrol_variant_demo.html`** — interactive proof of the variation model. Set a learner profile + today's energy and watch the engine pick the right exercise variant, flag cautions, and rule out contraindicated options.

## Data files (the content behind the engine)

- **`incontrol_learning_graph_nodes.csv`** — the 133 concepts (id, label, domain, role, parent, month, modifiable).
- **`incontrol_learning_graph_edges.csv`** — the 177 prerequisite links (source → target).
- **`incontrol_variants.csv`** — worked variant data for two exercises (Squat, Forward Fold); the authoring template for the rest.

## What this is / isn't

- It **is** the design + content + architecture for the lesson engine, all self-contained (no external libraries — the HTML files render offline).
- It **is not** the running service yet, and not the API contract — the OpenAPI spec is the next artifact.

## Decisions that need you and the team (flagged in the playbook)

- **Identity/auth ownership** — recommendation: the main app owns it; the service keys off a learnerId + token.
- **Trainerize integration** — the engine pushes the generated workout into Trainerize via a swappable adapter; confirm API access and the variant→exercise mapping.
- **Learner-data posture** — profiles/mastery are health-adjacent; consent + privacy (FERPA/GDPR) must be settled before storing any.
- **Contraindications + capability dimensions** are a non-clinician draft and need coach/clinician sign-off before they drive anything real.

Questions on any of this go to Shannon.
