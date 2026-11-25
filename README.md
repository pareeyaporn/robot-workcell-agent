# 🤖 Talk to a Robot: an AI agent for a robot workcell

**Tell a robot arm what to do in plain English, and watch it plan, see, and act.**

> "Sort the red parts into bin A and anything damaged into the reject bin."

An AI agent turns the request into a step-by-step plan. The robot's camera finds the parts, the arm picks and places them in a physics simulation, and the agent checks its own work and retries when something goes wrong. A live dashboard shows the plan, what the camera sees, and every step as it happens.

![Demo](docs/demo.gif) <!-- added in Week 3 -->

## Why this project
Language models are starting to drive robots: you give an instruction, and the robot works out how to do it. This project builds that loop end to end, on a small factory workcell (a robot arm, a table of parts, and sorting bins). The same pattern shows up in warehouse picking, kitting, and quality inspection.

## How it works

```
"Sort the red parts into bin A"
        │
        ▼
 ┌──────────────┐   plan: [find red parts] → [pick part 3] → [place in bin A] → ...
 │  AI planner  │──────────────────────────────────────────────────────────────┐
 │  (LLM agent) │◀── results / errors / camera observations ──┐               │
 └──────────────┘                                             │               ▼
                                                     ┌─────────────────────────────┐
                                                     │  Robot skills (the "tools")  │
                                                     │  look · pick · place · home  │
                                                     └─────────────────────────────┘
                                                        │                    ▲
                                                        ▼                    │
                                           ┌──────────────────┐   ┌──────────────────┐
                                           │ Physics sim       │   │ Perception        │
                                           │ (MuJoCo arm,      │──▶│ (overhead camera, │
                                           │  parts, bins)     │   │  part detection)  │
                                           └──────────────────┘   └──────────────────┘
```

1. **Planner (AI agent):** an LLM with tool use. It never moves the arm directly; it can only call a small set of safe, tested skills.
2. **Skills:** `look()`, `pick(part_id)`, `place(bin)`, `home()`. Each one is real robot code (inverse kinematics and motion), and each returns success or a clear error.
3. **Perception:** an overhead camera renders the table, and a vision step finds each part's position, color, and whether it's damaged. The agent works from what the camera sees, not from hidden simulator data.
4. **Verify and recover:** after each step, the agent looks again. If a pick failed or a part slipped, it re-plans.
5. **Guardrails:** workspace limits, a maximum number of steps, and refusal of unsafe or impossible requests (for example "throw the part across the room").

## Results (filled in as the project runs)

| Metric | Value |
|---|---|
| Task success rate (50-task evaluation set) | _TBD_ |
| Average steps per task | _TBD_ |
| Recovery rate after a failed pick | _TBD_ |
| Average planning time per task | _TBD_ |

## Tech stack
Python · MuJoCo (physics simulation) · Franka-style 7-joint arm with a suction gripper · NumPy / OpenCV (perception) · LLM agent with tool calling · Streamlit dashboard · pytest · built with Claude Code

## Run it
```bash
pip install -r requirements.txt
python -m workcell.demo "sort the red parts into bin A"   # runs with the offline planner, no API key needed
streamlit run app.py                                      # the dashboard
```
To use the AI planner, set an API key in `.env` (see `.env.example`). Without one, a rule-based offline planner runs the same skills, so the project always works.

## Roadmap
See [TASKS.md](TASKS.md). Possible next steps: a real low-cost arm (SO-101 with LeRobot), a vision-language model for perception, and learning-based grasping.
