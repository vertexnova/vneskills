# CodeRabbit settings for VertexNova

[vertexnova.coderabbit.yaml](vertexnova.coderabbit.yaml) is the single source of
truth for the org-wide CodeRabbit setup (schema v2). The review rules themselves
are the `vne-review*` skills in [../skills](../skills); this file only wires them
to paths.

## Apply in the web UI

1. Install the CodeRabbit app on `vertexnova/vneskills` (read access). Without
   it, the `vertexnova/vneskills:...` guideline entries cannot be read.
2. Open Organization Settings in CodeRabbit. Each field maps to a YAML key path;
   copy the values section by section:

| YAML key | What it sets |
|----------|--------------|
| `tone_instructions` | Severity-first comment style (max 250 chars) |
| `reviews.profile`, `poem`, `sequence_diagrams` | Assertive review, no extras |
| `reviews.auto_review.base_branches` | Also review PRs into `VNE_*` branches |
| `reviews.path_filters` | Skip vendored, generated, build, and test data |
| `reviews.path_instructions` | Short per-path checklists (fallback) |
| `reviews.pre_merge_checks.title` | Conventional Commit title (warning) |
| `reviews.tools` | Turns off noisy linters; the rest stay on |
| `knowledge_base.learnings.scope` | `global`: learnings shared by all repos |
| `knowledge_base.code_guidelines.filePatterns` | Each vne-review skill, applied only to its paths |

A repo that needs extra rules can commit its own `.coderabbit.yaml` with
`inheritance: true` and only the additions.

## Check it works

On the next reviewed PR, open the review details and confirm the guideline files
from `vertexnova/vneskills` are listed as used context. If they are not, the
`path_instructions` still carry the key checks.

## Change a rule

Edit the matching `skills/vne-review-*/SKILL.md`. Change this YAML only when a
path, tool, or setting changes, then validate:

```bash
check-jsonschema --schemafile https://coderabbit.ai/integrations/schema.v2.json \
  coderabbit/vertexnova.coderabbit.yaml
```

Keep skills compact (under about 90 lines): every guideline file is sent with
each review that matches its paths.
