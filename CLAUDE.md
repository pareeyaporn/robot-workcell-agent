# CLAUDE.md: robot-workcell-agent

Read TASKS.md first: it is the build plan, and its checkboxes are the source of truth for progress.

## Ground rules
- **Honest results only.** Every number in the README (success rate, steps, recovery) comes from running `tests/` or the evaluation script. Never estimate or round up.
- **The agent acts only through skills** (`look`, `inspect`, `pick`, `place`, `home`, `report`). Skills report real success or failure from the simulation.
- **Every action passes the safety shield** (`workcell/safety.py`) before the arm moves. Never add a code path that bypasses it.
- **Perception uses the camera image.** True simulator positions are allowed in tests only, never in the agent's inputs.
- **No API keys in code or commits.** Keys live in `.env` (gitignored). Everything must also run with the offline planner and no key.
- **Reproducible:** scenes take a `seed`; demos and evaluations record their seeds.
- Small commits with clear messages (`sim: add bins`, `skills: pick with suction check`). Run `pytest` before pushing.

## Layout
- `workcell/` the package: scene, kinematics, skills, perception, agent, safety, chaos, bench, voice, log, demo
- `tests/` pytest
- `docs/` images, GIFs, notes
- `app.py` mission-control dashboard (Week 4)
- `docs/results/` benchmark outputs and charts (Week 4)

## Rendering
Use `MUJOCO_GL=egl` for offscreen rendering on headless machines.
