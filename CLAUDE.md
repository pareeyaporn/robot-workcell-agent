# CLAUDE.md: robot-workcell-agent

Read TASKS.md first: it is the build plan, and its checkboxes are the source of truth for progress.

## Ground rules
- **Honest results only.** Every number in the README (success rate, steps, recovery) comes from running `tests/` or the evaluation script. Never estimate or round up.
- **The agent acts only through skills** (`look`, `pick`, `place`, `home`, `finish`). Skills report real success or failure from the simulation.
- **Perception uses the camera image.** True simulator positions are allowed in tests only, never in the agent's inputs.
- **No API keys in code or commits.** Keys live in `.env` (gitignored). Everything must also run with the offline planner and no key.
- **Reproducible:** scenes take a `seed`; demos and evaluations record their seeds.
- Small commits with clear messages (`sim: add bins`, `skills: pick with suction check`). Run `pytest` before pushing.

## Layout
- `workcell/` the package: scene, kinematics, skills, perception, agent, demo
- `tests/` pytest
- `docs/` images, GIFs, notes
- `app.py` Streamlit dashboard (Week 3)

## Rendering
Use `MUJOCO_GL=egl` for offscreen rendering on headless machines.
