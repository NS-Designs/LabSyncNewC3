# LabSync

LabSync helps a life-sciences lab book shared equipment, rooms and supplies, and helps the department spot savings across labs.

A researcher types something like "three hours on a confocal Tuesday morning, with a prep room first". LabSync works out the steps, finds times that fit, and books them. If the best time clashes with another lab's session, it asks that lab's manager before anything moves. On the department side, an admin can see which instruments are overbooked or sitting idle, and where two labs could combine supply orders to save money.

Nothing happens on its own. Every booking, move and order request is approved by a person, and every change is logged with who made it and when.

We built it for the C3 AI University Accelerator hackathon. All data is made up: a fictional department with four labs, ten instruments, four rooms and a small supply catalog. No real university systems or records are connected.

## Run it

You need Python 3.9 or newer. There's nothing to install.

```sh
python3 -m labsync_app.web --fresh --empty-calendar
```

Open http://127.0.0.1:8000. The terminal prints a reset code, which you'll need if you want to start over from the **Reset data** button.

- `--empty-calendar` starts with a blank week, so every booking goes straight through. Leave it off to load a busy week where bookings clash.
- Use **Signed in as** in the sidebar to switch between people:
  - Morgan is a researcher.
  - Chen is a lab manager.
  - Department admin sees the department view.

## Try it

**A simple booking.** As Morgan, click **Book the confocal**, press **Generate plan**, then press **Book this time**. That's it.

**A booking that needs someone else.** Start without `--empty-calendar`. As Morgan, use **Plan an imaging experiment**. The best time overlaps Chen's session, so hold it and ask Chen to move. Switch to Chen and approve it in **Approvals**. Switch back to Morgan and commit.

**The department view.** Switch to Department admin, open **Insights** and press **Run department review**. You'll see a shared-order saving, an overloaded microscope and an underused cytometer, each with the math behind it.

The full click-by-click demo is in [docs/DEMO-SCRIPT.md](docs/DEMO-SCRIPT.md).

## How it works

Three AI agents handle the language and judgment:

- The **Lab Agent** turns a request into a plan.
- The **Conflict Negotiator** advises a lab manager on whether to accept a move.
- The **Department Optimizer** writes up the department findings.

Plain code does everything that has to be right: permissions, availability, cleaning buffers, qualifications, prices and every state change. The agents can only suggest. A person presses the button.

By default the agents run offline as a rule-based stand-in. It understands the example requests and simple ones like "plate reader for 45 minutes Monday at 10". To use Claude instead, set `ANTHROPIC_API_KEY` and add `--live`.

Data is saved to `var/labsync_state.json` between runs. `--fresh` resets it when the app starts.

## Project layout

| Folder | What's in it |
|---|---|
| `labsync_app/` | The web app, the server and the code that connects the agents |
| `labsync_tools/` | The tools the agents call: search, booking, approvals and audit |
| `labsync_core/` | Scheduling, pricing and forecasting logic |
| `agents/` | Agent prompts and output formats |
| `data_gen/` | The made-up demo data |
| `cli/` | Command-line version of the demo |
| `tests/`, `evals/` | Tests, including a browser test of the full demo |
| `docs/` | The product spec and the demo script |

## Tests

```sh
python3 -m pytest tests -q
```

The browser test runs if Node and Playwright are installed, and skips itself otherwise.
