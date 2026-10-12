---
name: langfuse-llm-judge
description: Create or update native Langfuse LLM-as-a-Judge evaluators on Curefit's Langfuse instance. Select the project, inspect representative observations, gather missing evaluation requirements, configure prompts and field mappings, and verify the saved evaluator. Use for any project; application instrumentation changes and external evaluator workers require separate scope.
---

# Langfuse LLM Judge

Configure native evaluators at **https://langfuse.curefit.co** using the selected project's actual evidence and capabilities.

## Start with the project

Before configuring anything, ask: **“Which Langfuse project should I configure? Give its name or project URL.”** If the request already explicitly identifies the project, use that selection without repeating the question. Do not assume that the currently open project, an example in this skill, or a project used in a previous task is the intended target.

Resolve a project name through the authenticated organization/project picker. If several projects match, present the discovered names and ask the user to choose. Extract a project ID from a supplied URL only after verifying that the accessible page belongs to the requested project. Never guess IDs or substitute another project when access fails.

Open the verified project's evaluator page, normally `https://langfuse.curefit.co/project/<verified-project-id>/evals`, using an existing relevant tab or a browser tool. Prefer a purpose-built authenticated connector when available. Keep credentials out of skill files and reports.

## Inspect, then gather missing requirements

Inspect representative traces and the exact observation to be evaluated, existing evaluator names/configurations, configured judge connections, and the deployed editor. Select samples that expose the relevant input/output structure and context coverage. Read current official documentation when capabilities are uncertain; the self-hosted editor may differ from current documentation.

Ask focused questions in small groups about unresolved requirements. Do not ask the user to rediscover information available in the project, repeat an answered question, or infer material choices silently. Use a question tool when available; otherwise ask concise questions in conversation. Continue independent inspection while awaiting answers.

| Topic | Establish before configuring |
| --- | --- |
| Purpose | The behavior to assess and how the scores will be used |
| Target | A response, tool operation, conversation, or another recorded unit; eligible observation types/names/environments |
| Rubric | Good, bad, and inconclusive behavior; representative examples or reference answers |
| Context | Required evidence, its actual location, and whether it describes the evaluated event or current state |
| Scores | Score name/type, allowed values and meanings, missing-evidence treatment, and rationale requirements |
| Model | Configured judge connection/model, data-handling requirements, and cost constraints |
| Execution | Save only, sample testing, historical batch, or live evaluation; applicable dates, limits, and sampling |
| Acceptance | Who reviews judgments and what agreement/coverage is needed before broader execution |
| Existing evaluator | Create or update; the exact evaluator and configuration/score meanings to preserve |

Offer concrete recommendations based on inspected data. For example, recommend an observation containing the final response for per-response scores. Conversation-level evaluation needs the relevant conversation on the selected target, not merely a shared session ID. If examples are unavailable, propose synthetic cases for the user to review; label them synthetic rather than treating them as production evidence.

Before configuration, briefly state the agreed objective, target filters, variable mappings, rubric, score meanings, model, execution scope, and evidence limitations. Resolve material ambiguities first. This summary is not an extra approval gate when the user has already authorized the configuration.

Defaults when unspecified: use the configured project judge model, save inactive with live execution disabled, and perform read-only mapping checks. Clarify scope before paid testing, historical scoring, or live rules. If acceptance requirements are unnecessary for save-only work, record calibration as pending rather than blocking creation on an unspecified reviewer.

## Map the evidence actually available

| Variable | Meaning | Mapping |
| --- | --- | --- |
| `input` | User request or application input | Selected observation's Input, optionally narrowed by a verified JSONPath |
| `output` | Response being evaluated | Selected observation's Output, optionally narrowed by a verified JSONPath |
| `context` | Evidence required for the agreed rubric | Relevant selected-observation field, optionally narrowed by a verified JSONPath |

Use these variable names when they fit; add only variables the rubric needs. Preserve structured JSON. Context is optional for criteria such as tone; do not fabricate it or force an unrelated metadata field into the prompt. For retrieval, workout, reference-answer, or tool evaluations, discover the appropriate context rather than hardcoding a domain.

For example, if **the selected observation's Metadata** contains a `workout_context` property, `context` can map to Metadata with JSONPath `$.workout_context`. This is an example, not a default for all projects. Verify supported JSONPath syntax in the actual editor.

Distinguish trace metadata from observation metadata. A property on the parent trace is not proof that the generation has that property. Observation evaluators receive fields from the selected observation; they do not automatically fetch sibling/child observations or other traces. A tool-calls field can be used only if it is actually present and supported by this deployment.

Tool evidence must be explicitly associated with the evaluated operation or turn. Session IDs or timestamps alone do not safely associate separate calls. Missing evidence is not proof of no calls. A success claim is not proof of business success, and HTTP success alone may not establish business success. A correctly explained failure can be a correct response. Use the rubric's agreed abstention outcome when a required result/link is unavailable; distinguish this from a recorded contradiction.

Require a populated rendered preview for each required variable on a representative eligible observation. Compare the rendered values with the source fields. A saved JSONPath or success toast proves configuration was saved, not that context reaches the judge. If the preview is blank, inspect the selected sample and field level, choose an eligible sample, and correct the mapping. If none contains the required evidence, report the gap and do not declare that mapping verified. For optional context, explicitly confirm that empty context is intentional.

Prefer a relevant metadata subtree over all metadata. Avoid passing unrelated identifiers, secrets, audio references, or large plans when an adequate summary is already available. Native mapping is not arbitrary preprocessing: do not claim a prompt removes data before provider transmission. Explain scope and cost implications when the available context is large or contains unnecessary fields.

## Define and save the evaluator

Read [references/judge-prompt-template.md](references/judge-prompt-template.md) when drafting the prompt. Adapt its objective, evidence rules, rubric, verdicts, and examples to the project. Do not carry Voice Coach names, numeric codes, workout facts, or a model from a previous task into a new project.

Use the score types actually supported by the deployed editor. If only numeric scores are supported, agree a numeric encoding for the desired labels, including an inconclusive value where needed. Preserve existing score names and meanings when updating; status codes are categorical encodings and should not be averaged as quality ratings. Resolve incompatible schema changes explicitly.

Configure the evaluation prompt, score reasoning/comment prompt, and score-range/schema controls where the editor separates them. Comments should be concise evidence summaries without quoting private transcript details or identifiers. Exclude evaluator/internal traffic through filters appropriate to the project; do not copy environment names without checking them. Check prompt-only/non-conversational records against the intended target.

When updating, identify the exact running configuration and referenced template. Some versions separate template versions from execution configurations: verify the running configuration references the intended saved version and retains its mappings, filters, model, and execution state. Avoid accidental duplicate evaluators and unrelated edits.

Save and inspect the resulting configuration. Keep save-only work inactive. Enable rules or historical execution only within the agreed scope. If the deployed version lacks the requested batch feature, report that limitation; do not silently activate live evaluation or implement an external worker/application change as a workaround.

## Validate and report

For authorized sample testing, use a small varied set matching the objective: supported response, contradiction, missing context, ambiguous input, and applicable tool success/failure/retry/clarification cases. Inspect rendered evidence first, then review scores and comments against expected judgments. Do not represent save-only inspection as model calibration. Broader scoring waits for the agreed acceptance criteria.

Verify score attachment to the intended original observation (or expressly chosen target), score name/type/value, visible rationale, and evidence provenance where supported. Record which tests ran and which remain pending. Freeze historical date bounds before running a batch and report counts, verdict distribution, exclusions, and evidence gaps without reproducing transcripts/user identifiers.

Check configured pricing before interpreting cost. Verify the exact judge model, currency, per-token versus per-million-token units, and input/output/cache/audio pricing as applicable. Compare token usage and billing period with provider usage when available. Saving a configuration or seeing a changed dashboard counter does not establish that evaluations ran or that spend is correct; inspect execution logs and pricing before attributing the change.

Finish with the project/evaluator link, saved version, model, input/output/context mappings and verification status, score meanings, execution state, tests performed, and remaining limitations. Keep application changes and external integrations separate unless requested.

## References

- [Langfuse LLM-as-a-Judge and observation data availability](https://langfuse.com/docs/evaluation/evaluation-methods/llm-as-a-judge)
- [Langfuse token and cost tracking](https://langfuse.com/docs/observability/features/token-and-cost-tracking)
- [Langfuse score data model](https://langfuse.com/docs/evaluation/scores/data-model)
