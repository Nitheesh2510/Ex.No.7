# Exno.7-Develop a prompt-based application tailored to their personal needs, fostering creativity and practical problem-solving skills while leveraging the capabilities of large language models.

### Date: 19.05.2026
### Name : Nitheesh Kumar B
### Register No: 212224230189
---

## Aim

To develop a prompt-based application using ChatGPT and demonstrate how to create a personal productivity assistant to organize daily tasks, schedule reminders, suggest wellness tips, and answer general queries using natural language interaction.

---

## AI Tools Required

- ChatGPT
- Python
- Tkinter
- VS Code

---

## Explanation

## Prompt

Design a personal productivity assistant that can help manage daily tasks, schedule reminders, suggest wellness tips, and answer general queries. The assistant should interact using natural language and be adaptable to the user’s changing preferences over time.

---

## Procedure

1. Defined the core requirements of a personal productivity assistant.
2. Constructed prompts for managing tasks, reminders, wellness tips, and user queries using ChatGPT.
3. Developed a simple GUI-based application using Python and Tkinter.
4. Simulated natural user interaction through buttons and input fields.
5. Added task management, reminders, wellness suggestions, and summary generation.
6. Displayed outputs and responses based on user interactions.

---

## Python Code

```python
import os
import json
import datetime
import anthropic
 
# ── Anthropic client ───────────────────────────────────────────────────────────
client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY", "YOUR_API_KEY_HERE"))
 
def ask_claude(system_prompt: str, user_message: str) -> str:
    """Send a message to Claude and return the text response."""
    message = client.messages.create(
        model="claude-opus-4-5",
        max_tokens=1000,
        system=system_prompt,
        messages=[{"role": "user", "content": user_message}]
    )
    return message.content[0].text
 
 
# ── Preference store (simulates memory) ───────────────────────────────────────
preferences: dict = {}
 
def save_pref(key: str, value: str):
    preferences[key] = value
 
def show_prefs():
    if preferences:
        print("\n  📌 Remembered Preferences:")
        for k, v in preferences.items():
            print(f"     • {k}: {v}")
 
 
# ── Colour / formatting helpers ────────────────────────────────────────────────
RESET  = "\033[0m"
BOLD   = "\033[1m"
GREEN  = "\033[92m"
CYAN   = "\033[96m"
YELLOW = "\033[93m"
RED    = "\033[91m"
MAGENTA= "\033[95m"
DIM    = "\033[2m"
 
def banner(text: str, color=CYAN):
    width = 62
    print(f"\n{color}{BOLD}{'─' * width}")
    print(f"  {text}")
    print(f"{'─' * width}{RESET}")
 
def section(text: str):
    print(f"\n{BOLD}{CYAN}  {text}{RESET}")
 
def ok(text: str):
    print(f"{GREEN}  ✔  {text}{RESET}")
 
def warn(text: str):
    print(f"{YELLOW}  ⚠  {text}{RESET}")
 
def info(text: str):
    print(f"{DIM}     {text}{RESET}")
 
def ai_response(text: str):
    print(f"\n{MAGENTA}  ✦  {text}{RESET}\n")
 
def prompt(text: str) -> str:
    return input(f"{BOLD}{CYAN}  › {text}{RESET} ").strip()
 
 
# ══════════════════════════════════════════════════════════════════════════════
# MODULE 1 — DAILY TASK MANAGER
# ══════════════════════════════════════════════════════════════════════════════
 
tasks: list[dict] = [
    {"id": 1, "text": "Review project proposal", "priority": "high",   "due": "Today",   "done": False},
    {"id": 2, "text": "Buy groceries",            "priority": "medium", "due": "Today",   "done": False},
    {"id": 3, "text": "Read 20 pages",            "priority": "low",    "due": "Tonight", "done": False},
]
_task_id_counter = 4
 
PRIORITY_SYMBOL = {"high": f"{RED}●{RESET}", "medium": f"{YELLOW}●{RESET}", "low": f"{CYAN}●{RESET}"}
 
 
def _next_task_id() -> int:
    global _task_id_counter
    val = _task_id_counter
    _task_id_counter += 1
    return val
 
 
def add_task_nl(description: str):
    """Parse a natural-language task description with Claude and add it."""
    print(f"{DIM}  Parsing with Claude...{RESET}", end="", flush=True)
    raw = ask_claude(
        'You are a task parser. Given a natural language task description, '
        'extract and return ONLY a JSON object with keys: '
        '"text" (string), "priority" ("high"|"medium"|"low"), '
        '"due" (string like "Today", "Tomorrow", "6 PM"). '
        'Return raw JSON only, no markdown, no explanation.',
        description
    )
    print("\r" + " " * 40 + "\r", end="")
    try:
        parsed = json.loads(raw.strip().strip("```json").strip("```"))
    except json.JSONDecodeError:
        parsed = {"text": description, "priority": "medium", "due": "Today"}
 
    task = {"id": _next_task_id(), "done": False, **parsed}
    tasks.append(task)
    ok(f'Added: "{task["text"]}"  [{task["priority"].upper()}]  Due: {task["due"]}')
 
 
def list_tasks():
    pending = [t for t in tasks if not t["done"]]
    done    = [t for t in tasks if t["done"]]
 
    section("Pending Tasks")
    if pending:
        for t in pending:
            sym = PRIORITY_SYMBOL.get(t["priority"], "●")
            print(f"  [{t['id']:>2}] {sym} {t['text']:<40} {DIM}{t['due']}{RESET}")
    else:
        info("No pending tasks — great job!")
 
    if done:
        section("Completed")
        for t in done:
            print(f"  {DIM}[{t['id']:>2}] ✓ {t['text']}{RESET}")
 
 
def complete_task():
    list_tasks()
    raw = prompt("Enter task ID to mark complete (or blank to cancel):")
    if not raw:
        return
    try:
        tid = int(raw)
        for t in tasks:
            if t["id"] == tid:
                t["done"] = True
                ok(f'"{t["text"]}" marked complete.')
                return
        warn("Task ID not found.")
    except ValueError:
        warn("Invalid ID.")
 
 
def daily_summary():
    pending = [t for t in tasks if not t["done"]]
    if not pending:
        ai_response("All tasks are complete — you crushed it today! 🎉")
        return
    print(f"{DIM}  Generating summary...{RESET}", end="", flush=True)
    summary = ask_claude(
        "You are a friendly productivity coach. Given a list of pending tasks, "
        "write a short motivating daily summary (3-4 sentences). Mention priorities. Be encouraging.",
        f"Pending tasks: {json.dumps(pending)}"
    )
    print("\r" + " " * 40 + "\r", end="")
    ai_response(summary)
 
 
def tasks_menu():
    while True:
        banner("✅  Daily Task Manager")
        print("  1) View tasks")
        print("  2) Add task (natural language)")
        print("  3) Mark task complete")
        print("  4) Get daily summary")
        print("  0) Back to main menu")
        choice = prompt("Choose:")
 
        if choice == "1":
            list_tasks()
        elif choice == "2":
            desc = prompt("Describe your task:")
            if desc:
                add_task_nl(desc)
        elif choice == "3":
            complete_task()
        elif choice == "4":
            daily_summary()
        elif choice == "0":
            break
        else:
            warn("Invalid choice.")
 
 
# ══════════════════════════════════════════════════════════════════════════════
# MODULE 2 — SMART SCHEDULER
# ══════════════════════════════════════════════════════════════════════════════
 
events: list[dict] = [
    {"id": 1, "title": "Team standup",   "time": "09:00", "duration": 30,  "type": "work"},
    {"id": 2, "title": "Lunch break",    "time": "13:00", "duration": 60,  "type": "personal"},
    {"id": 3, "title": "Design review",  "time": "15:00", "duration": 45,  "type": "work"},
]
_event_id_counter = 4
TYPE_COLOR = {"work": CYAN, "personal": MAGENTA, "health": GREEN}
 
 
def _next_event_id() -> int:
    global _event_id_counter
    val = _event_id_counter
    _event_id_counter += 1
    return val
 
 
def _minutes(time_str: str) -> int:
    h, m = map(int, time_str.split(":"))
    return h * 60 + m
 
 
def check_overlaps() -> list[str]:
    sorted_ev = sorted(events, key=lambda e: e["time"])
    overlaps = []
    for i in range(len(sorted_ev) - 1):
        a, b = sorted_ev[i], sorted_ev[i + 1]
        a_end = _minutes(a["time"]) + a["duration"]
        b_start = _minutes(b["time"])
        if a_end > b_start:
            overlaps.append(f'"{a["title"]}" ({a["time"]}) overlaps with "{b["title"]}" ({b["time"]})')
    return overlaps
 
 
def add_event_nl(description: str):
    print(f"{DIM}  Parsing with Claude...{RESET}", end="", flush=True)
    raw = ask_claude(
        'You are a scheduling assistant. Parse the natural language event description and return '
        'ONLY a JSON object with: "title" (string), "time" (HH:MM 24h format), '
        '"duration" (minutes as integer), "type" ("work"|"personal"|"health"). '
        'Return raw JSON only, no markdown.',
        description
    )
    print("\r" + " " * 40 + "\r", end="")
    try:
        parsed = json.loads(raw.strip().strip("```json").strip("```"))
    except json.JSONDecodeError:
        parsed = {"title": description, "time": "12:00", "duration": 30, "type": "personal"}
 
    event = {"id": _next_event_id(), **parsed}
    events.append(event)
    events.sort(key=lambda e: e["time"])
    ok(f'Scheduled: "{event["title"]}" at {event["time"]} ({event["duration"]} min) [{event["type"]}]')
 
    overlaps = check_overlaps()
    for o in overlaps:
        warn(f"Overlap: {o}")
 
 
def list_events():
    section("Today's Schedule")
    if not events:
        info("No events scheduled.")
        return
    for ev in sorted(events, key=lambda e: e["time"]):
        col = TYPE_COLOR.get(ev["type"], DIM)
        end_min = _minutes(ev["time"]) + ev["duration"]
        end_time = f"{end_min // 60:02d}:{end_min % 60:02d}"
        print(f"  {col}{ev['time']} – {end_time}{RESET}  {ev['title']:<35} {DIM}[{ev['type']}]{RESET}")
 
 
def find_free_slots():
    print(f"{DIM}  Analysing schedule...{RESET}", end="", flush=True)
    advice = ask_claude(
        "You are a scheduling assistant. Given a list of today's events, identify free time slots "
        "and suggest 2-3 productive activities for those gaps. Keep it under 5 sentences.",
        f"Today's events: {json.dumps(events)}"
    )
    print("\r" + " " * 40 + "\r", end="")
    ai_response(advice)
 
 
def schedule_menu():
    while True:
        banner("📅  Smart Scheduler")
        print("  1) View today's schedule")
        print("  2) Add event (natural language)")
        print("  3) Check for overlaps")
        print("  4) Find free slots & suggestions")
        print("  0) Back to main menu")
        choice = prompt("Choose:")
 
        if choice == "1":
            list_events()
        elif choice == "2":
            desc = prompt("Describe your event:")
            if desc:
                add_event_nl(desc)
        elif choice == "3":
            overlaps = check_overlaps()
            if overlaps:
                for o in overlaps:
                    warn(o)
            else:
                ok("No overlaps detected — your schedule is clean!")
        elif choice == "4":
            find_free_slots()
        elif choice == "0":
            break
        else:
            warn("Invalid choice.")
 
 
# ══════════════════════════════════════════════════════════════════════════════
# MODULE 3 — WELLNESS TIPS GENERATOR
# ══════════════════════════════════════════════════════════════════════════════
 
WELLNESS_CATEGORIES = [
    "hydration", "exercise", "sleep",
    "screen break", "nutrition", "mindfulness"
]
wellness_history: list[dict] = []
 
 
def get_wellness_tip(category: str):
    pref_note = (
        f"The user previously liked tips about: {preferences.get('wellnessFocus', '')}."
        if "wellnessFocus" in preferences else "Give a general wellness tip."
    )
    print(f"{DIM}  Fetching tip...{RESET}", end="", flush=True)
    tip = ask_claude(
        f"You are a warm wellness coach for busy people. Give a short, practical, specific "
        f"wellness tip (2-3 sentences max). {pref_note} Make it actionable and friendly.",
        f"Give me a {category} wellness tip for today."
    )
    print("\r" + " " * 40 + "\r", end="")
    ai_response(tip)
 
    wellness_history.append({
        "category": category,
        "tip": tip,
        "time": datetime.datetime.now().strftime("%H:%M")
    })
 
    fb = prompt("Was this helpful? (y/n/skip):").lower()
    if fb == "y":
        save_pref("wellnessFocus", category)
        ok("Preference saved — future tips will be tailored!")
    elif fb == "n":
        info("Got it — we'll try a different angle next time.")
 
 
def wellness_menu():
    while True:
        banner("🌿  Wellness Tips Generator")
        for i, cat in enumerate(WELLNESS_CATEGORIES, 1):
            print(f"  {i}) {cat.capitalize()}")
        print(f"  H) View tip history")
        print(f"  0) Back to main menu")
        choice = prompt("Choose category or option:")
 
        if choice == "0":
            break
        elif choice.upper() == "H":
            if wellness_history:
                section("Recent Wellness Tips")
                for entry in wellness_history[-5:]:
                    print(f"  {DIM}[{entry['time']}]{RESET} {CYAN}{entry['category'].capitalize()}{RESET}")
                    print(f"     {entry['tip'][:100]}{'...' if len(entry['tip']) > 100 else ''}")
            else:
                info("No tips yet — pick a category!")
        elif choice.isdigit() and 1 <= int(choice) <= len(WELLNESS_CATEGORIES):
            get_wellness_tip(WELLNESS_CATEGORIES[int(choice) - 1])
        else:
            warn("Invalid choice.")
 
 
# ══════════════════════════════════════════════════════════════════════════════
# MODULE 4 — GENERAL ASSISTANT (Conversational with memory)
# ══════════════════════════════════════════════════════════════════════════════
 
chat_history: list[dict] = []
 
 
def chat_session():
    banner("💬  General Assistant")
    print(f"  {MAGENTA}Hi! I'm Aria, your personal productivity assistant.")
    print(f"  Ask me anything — tasks, schedule, wellness, or general queries.")
    print(f"  Type 'back' to return to the main menu.{RESET}\n")
 
    while True:
        user_input = prompt("You:")
        if not user_input:
            continue
        if user_input.lower() in ("back", "exit", "quit", "0"):
            break
 
        # Build preference context
        pref_context = ", ".join(f"{k}: {v}" for k, v in preferences.items()) if preferences else ""
 
        # Build conversation history string for context
        history_str = ""
        for msg in chat_history[-6:]:   # last 3 exchanges
            role = "User" if msg["role"] == "user" else "Aria"
            history_str += f"{role}: {msg['content']}\n"
 
        system = (
            "You are Aria, a warm and intelligent personal productivity assistant. "
            "You help with tasks, scheduling, wellness, and general queries. "
            f"{'User preferences: ' + pref_context + '.' if pref_context else ''} "
            "Keep responses concise (under 5 sentences unless asked for detail). "
            "Be friendly, practical, and encouraging."
        )
 
        full_prompt = (f"Conversation so far:\n{history_str}\nUser: {user_input}" if history_str
                       else user_input)
 
        print(f"{DIM}  Thinking...{RESET}", end="", flush=True)
        reply = ask_claude(system, full_prompt)
        print("\r" + " " * 40 + "\r", end="")
        ai_response(reply)
 
        # Store in history
        chat_history.append({"role": "user",      "content": user_input})
        chat_history.append({"role": "assistant",  "content": reply})
 
        # Detect preference mentions
        if any(kw in user_input.lower() for kw in ("i prefer", "i like", "i enjoy", "i love")):
            save_pref("chatMention", user_input[:80])
            info("(Preference noted ✓)")
 
 
# ══════════════════════════════════════════════════════════════════════════════
# MAIN MENU
# ══════════════════════════════════════════════════════════════════════════════
 
def main():
    print(f"\n{BOLD}{CYAN}")
    print("  ╔══════════════════════════════════════════════╗")
    print("  ║       ARIA · Personal Productivity AI        ║")
    print("  ║       Powered by Anthropic Claude            ║")
    print("  ╚══════════════════════════════════════════════╝")
    print(RESET)
    now = datetime.datetime.now()
    hour = now.hour
    greeting = "Good morning" if hour < 12 else ("Good afternoon" if hour < 17 else "Good evening")
    print(f"  {greeting}! Today is {now.strftime('%A, %B %d')}.\n")
 
    while True:
        show_prefs()
        banner("MAIN MENU")
        print("  1) ✅  Daily Task Manager")
        print("  2) 📅  Smart Scheduler")
        print("  3) 🌿  Wellness Tips Generator")
        print("  4) 💬  General Assistant (Chat)")
        print("  0) 👋  Exit")
        choice = prompt("Choose a module:")
 
        if choice == "1":
            tasks_menu()
        elif choice == "2":
            schedule_menu()
        elif choice == "3":
            wellness_menu()
        elif choice == "4":
            chat_session()
        elif choice == "0":
            print(f"\n  {GREEN}Stay productive! Goodbye. 👋{RESET}\n")
            break
        else:
            warn("Please enter 1–4 or 0.")
 
 
if __name__ == "__main__":
    main()
```

---

## EXPECTED OUTPUT

### Personal Productivity Assistant Features

### 1. Daily Task Manager
- Accept tasks using natural language
- Organize tasks by priority
- Display pending and completed tasks
- Generate daily summaries

### 2. Smart Scheduler
- Allow reminder and scheduling support
- Help users organize activities efficiently
- Improve time management

### 3. Wellness Tips Generator
- Suggest hydration reminders
- Recommend screen-time breaks
- Provide healthy productivity tips

### 4. Query Assistant
- Answer productivity-related queries
- Suggest study techniques and focus strategies

---

## Output 

### Main Interface

<img width="922" height="957" alt="image_76" src="https://github.com/user-attachments/assets/5b128283-bf57-4a83-8787-80ad70a7f364" />

### Assistant Query Response

<img width="916" height="982" alt="image" src="https://github.com/user-attachments/assets/ac088333-59c6-4a90-bb5c-518de0a71bf8" />

### Daily Summary

<img width="922" height="978" alt="image" src="https://github.com/user-attachments/assets/a44276dd-a088-42e0-8234-71101d6497a9" />


---



## Result

The lab exercise resulted in the successful creation of a prototype personal productivity assistant powered by prompt-based interaction. The experiment helped in:
- Understanding prompt engineering techniques
- Developing creativity and problem-solving skills
- Implementing real-world AI-based applications
- Exploring the usefulness of large language models in productivity management
