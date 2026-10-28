# small-model-agents-md

A global `AGENTS.md` for coding agents running **small local models** (around 4B parameters), tuned and measured on a 50-task benchmark.

Small models follow short, concrete rules much better than long workflows. This file is the version that scored best and most consistently out of the 12 variants we benchmarked: it passed 46 of 50 tasks in both of two runs, and a full run takes about 25 minutes.

## What's in the file

[`.config/opencode/AGENTS.md`](.config/opencode/AGENTS.md) (534 words) has four short sections:

- **Safety and Paths**: never `rm`, `mv` or `>` existing files; test "missing file" cases on copies in `/tmp`; work only in the reported working directory; check with `ls` before saying a file is missing.
- **Exact Requirements**: copy names and formats exactly; use the input interface the user asked for; a condition applies only to the cases it names; in follow-up requests, read the current code first and keep earlier behavior.
- **Verification**: always run the code after a change; test both sides of each condition and the error cases; compare the full output, and show differences with `repr()`.
- **Reporting**: report only what was actually run and seen; never end with only a plan.

It is language-agnostic, and none of its examples come from the benchmark tasks.

## Install (opencode)

```bash
mkdir -p ~/.config/opencode
cp .config/opencode/AGENTS.md ~/.config/opencode/AGENTS.md
```

opencode loads `~/.config/opencode/AGENTS.md` in every session, in any folder, together with a project's own `AGENTS.md` if it has one. Other agents that read an `AGENTS.md` can use the same file.

## Test setup

- **Model:** Qwen3.5 4B (Q4_K_M) on Ollama, text-only (vision projector removed so the whole model fits on the GPU), 32k context, thinking on, `temperature 0.6`, `presence_penalty 0`
- **Hardware:** GTX 1650 Ti, 4 GB VRAM; Ollama with flash attention and q8_0 KV cache
- **Agent:** opencode 2.1.4, `execute` tool disabled (it runs JavaScript, and the model kept sending it shell commands), 40-step cap per message in tests
- **Benchmark:** 50 Python tasks, each in a fresh folder and session, graded by an independent checker the model never sees:
  - 26 main tasks (exact conditions, scoped edits, bug fixes, output formats, Persian prompts, follow-up messages, honesty)
  - 7 held-out tasks never used while writing rules
  - 12 new tiny tasks and 5 larger multi-file tasks (packages, refactors, a bug fix against existing tests, CSV export, a two-message Persian change)

## What we learned

1. **Concrete beats general.** "To test a missing file, copy it to `/tmp`" stopped the model from deleting user files. "Never delete files" alone did not.
2. **Long workflows get ignored or cost too much.** A multi-step "write tests first" workflow was followed in at most 1 of 50 tasks when it was one section among others. With a fixed path and a code template the model followed it in 29 of 50 tasks, but those runs took about 2.5 times longer and passed fewer tasks. The same ideas written as a few short verification lines were applied and scored best.
3. **There is a sweet spot in length.** Halving this file, or adding an example to every rule, both made results worse.
4. **More rules isn't always better.** Adding four sensible extra rules made results worse; one of them even made a task fail in a new way.
5. **A rules file clearly helps.** With no `AGENTS.md`, the model already had 5 failures within the first 26 tasks; with this file it averages 4 failures across all 50.
6. **Infrastructure matters as much as rules.** Fitting the whole model on the GPU, using real 32k context, disabling a confusing tool, and setting `presence_penalty` to 0 gave large gains before any rule was written.
7. **Avoid benchmaxing.** Early drafts used examples taken from the test tasks. They were replaced with neutral ones, and every draft was checked against all 50 tasks.
8. **Results are noisy.** Single runs move by about ±2 tasks, so differences of 1–2 tasks need repeated runs before you trust them.

## Limitations

- Tested with one model (Qwen3.5 4B) and one agent (opencode) on small Python tasks. Larger models and real projects may behave differently.
- Most variants were run only once or twice.
- Remaining weak spots: multi-command programs changed in a follow-up message, and occasional output-format slips such as an extra blank line.

## License

[MIT](LICENSE)
