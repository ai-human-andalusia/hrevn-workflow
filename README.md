# hrevn-workflow

Experimental, local-first Python SDK for checkpointing AI workflows and resuming from the last valid step.

## Status and scope

`hrevn-workflow` is preserved as an open-source developer experiment. It is not a current HREVN product, managed service or committed roadmap line.

Its supported scope is local only:

- record step-level checkpoints;
- resume a workflow from the last valid step;
- hash inputs and outputs with SHA-256;
- detect changed artifacts;
- export a machine-readable workflow manifest;
- verify local checkpoint and manifest consistency.

It does **not** issue, sign, anchor, certify or ratify HREVN bundles. It is not part of AgentProof, EvalDossier or the current HREVN Core.

The repository retains historical remote-integration code for reproducibility of the experiment. That path is unsupported, is not a current HREVN issuance surface and should not be configured for new use.

## What the experiment demonstrates

Long AI workflows often fail after several expensive or review-heavy steps. A local checkpoint chain can help a developer determine:

- which steps completed;
- which inputs and outputs were recorded;
- whether a recorded artifact changed;
- where execution can resume without blindly rerunning earlier work.

This is continuity and local integrity evidence. It is not proof of authorship, external authority, legal validity or HREVN issuance.

## Installation

The package is distributed from this repository rather than as a supported PyPI product.

```bash
git clone https://github.com/miguel-herrero-systems/hrevn-workflow.git
cd hrevn-workflow
python -m venv .venv
source .venv/bin/activate
pip install -e '.[dev]'
```

## Minimal usage

```python
from hrevn_workflow import Workflow

wf = Workflow(workflow_id="agent_task_001", storage_path="./.hrevn")

with wf.step("01_analyze_input", inputs=["source.txt"]) as step:
    if step.should_run():
        result = run_agent_analysis("source.txt")
        write_json("result.json", result)
        step.complete(outputs=["result.json"], model_used="example-model")

wf.export_manifest("workflow_manifest.json")
```

The SDK records metadata and hashes. It does not need to own or upload the workflow files.

## Local files

The experiment stores state under the directory selected by `storage_path`:

```text
.hrevn/
  workflow_state.json
  checkpoints/
    01_analyze_input.json
  manifests/
    workflow_manifest.json
```

Generated outputs remain wherever the calling workflow writes them.

## CLI

```bash
hrevn-workflow --storage-path .hrevn status
hrevn-workflow --storage-path .hrevn history
hrevn-workflow --storage-path .hrevn doctor --manifest-path workflow_manifest.json
hrevn-workflow --storage-path .hrevn verify --manifest-path workflow_manifest.json
hrevn-workflow --storage-path .hrevn manifest --path workflow_manifest.json
hrevn-workflow --storage-path .hrevn inspect-step --step-id 01_analyze_input
```

The CLI can inspect state, show checkpoint history, export a manifest and compare current files with recorded hashes.

## Included examples

- `examples/basic_ai_pipeline.py`
- `examples/vendor_due_diligence_pipeline.py`

Run the tests and examples locally:

```bash
pytest -q
python examples/basic_ai_pipeline.py
python examples/vendor_due_diligence_pipeline.py
```

## Verification boundary

A successful local verification means that the files and checkpoint metadata supplied to this implementation are consistent with the hashes it recorded. It does not establish:

- that HREVN issued the manifest;
- that a trusted external signer approved it;
- that the workflow description is true or complete;
- that an AI output is correct;
- that any event occurred at an externally verified time;
- legal or regulatory compliance.

For current HREVN developer products and their precise boundaries, see [HREVN Developers](https://hrevn.com/en/developers/).

## License

MIT. See `LICENSE`.
