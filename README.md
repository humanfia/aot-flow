# aot _(flow-aot)_

AOT for hmz: the flow that writes a flow - a description in, a loaded, smoke-run and reviewed flow out.

Describe a flow, and get one you can run. A writer drafts it; [hmz](https://github.com/humanfia/humanize)
loads the draft and runs it on fake agents; a critic reads it fresh. Only a draft that passes
all three lands among this project's own flows, the `local` flowverse.

## Table of Contents

- [Install](#install)
- [Usage](#usage)
  - [How a draft is checked](#how-a-draft-is-checked)
  - [Roles and params](#roles-and-params)
  - [What ends it](#what-ends-it)
- [Contributing](#contributing)
- [License](#license)

## Install

You need [hmz](https://github.com/humanfia/humanize). In hmz, open `/flow`, go to
**Flowverses → official → aot** and **Install** it.

To run a release without installing it, name it by its git ref:

```sh
hmz exec -f git+https://github.com/humanfia/flow-aot@v0.1.1#aot ...
```

## Usage

```text
❯ $aot two agents take turns until a reviewer says it is done
```

```sh
hmz exec -f aot -a writer=claude/claude-opus-5:high -a critic=codex/gpt-5.6-sol:high \
    -p budget.cost=10 "two agents take turns until a reviewer says it is done"
```

When it lands, it prints what the new flow drives, takes and ends on, and the line that runs
it:

```text
compiled: turn_taking_review -- two agents take turns until a reviewer approves
landed:   .hmz/flows/turn_taking_review
drives:   worker -- an agent
drives:   reviewer -- an agent
ends:     by verdict -- reviewer says done, within 6 rounds

hmz exec -f local/turn_taking_review -a worker=CLI/MODEL:EFFORT -a reviewer=CLI/MODEL:EFFORT -p budget.cost=USD "the task"
```

At the prompt, the new flow is `$local/turn_taking_review`.

### How a draft is checked

1. **A spec first.** The writer draws up the roles the flow drives, what each must be able to
   do, its params and what ends it. A capability no backend serves is sent back to be restated,
   then put to you. A name that is already taken is put to you too.
2. **It loads.** hmz loads the draft and reads back what it declares.
3. **It ends on its own.** The draft runs on fake agents in three worlds: an agent that never
   says it is done, one that says so at once, and one that answers nothing. It must end in
   every one within `seconds`.
4. **A critic approves it.** A fresh session that may only read, and never saw it written.

A refusal at any step goes back to the writer word for word, for up to `repairs` rounds.

### Roles and params

| Role | |
| --- | --- |
| `writer` | Draws the spec, then drafts and repairs the flow, in one session. |
| `critic` | Reads each draft that passed the gates, in a fresh session. It may only read. |
| `human` | You, filled in by hmz. Asked about names, capabilities, and a draft whose repairs ran out. |

Both agents carry the flow's own [skill](https://docs.humanfia.ai/humanize/user/skills),
`writing-flows`: how a flow is written against hmz. Any backend can fill either role.

| Param | Default | |
| --- | --- | --- |
| `name` | blank | What to call the new flow. Blank takes the name the spec gives it. |
| `into` | `local` | `local` lands it among this project's flows; `user` among yours, in your home directory. |
| `repairs` | `3` | Rounds of repair after the first draft, 0 to 6. |
| `strict` | `false` | Send a draft back for every warning, not only for what blocks it. |
| `seconds` | `60` | How long each run on fakes may take. |

### What ends it

- **The flow lands.** It is moved into place whole, and never over a flow that is already
  there.
- **Nothing lands.** No spec could be drawn, you declined a name or a capability, or the
  repairs ran out and you did not take the draft as it stands. Under `hmz exec` nobody is there
  to say yes, so every such question is a no.
- **The [budget](https://docs.humanfia.ai/humanize/features/allowances).**

`aot` keeps nothing for `--resume`: each run is one description, and running it again writes
another flow. To write a flow by hand instead, see
[Writing a flow](https://docs.humanfia.ai/humanize/weaver/writing-a-flow).

## Contributing

Issues and pull requests are welcome. The flow is the `aot/` directory; its tests are in
`tests/`. With [uv](https://docs.astral.sh/uv/):

```sh
uv sync
uv run ruff check
uv run ruff format --check
uv run pytest
```

`tests/test_aot_golden.py` compiles real flows with real agents, so it is skipped unless
`AOT_WRITER=cli/model:effort` (and optionally `AOT_CRITIC`, `AOT_BUDGET`, `AOT_SMOKE=1`) says
which to use.

CI runs the same on every pull request. A release is a `vX.Y.Z` tag on `main`; it reaches hmz
users once a `flows/aot/X.Y.Z/flow.yaml` pointing at it is merged into
[humanfia/flowverse](https://github.com/humanfia/flowverse).

## License

[Apache-2.0](LICENSE) © Humanfia
