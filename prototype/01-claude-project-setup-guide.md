# Step 3 setup guide: build the prototype in a Claude Project

You've never used Claude Projects, here's every click. Confirmed against Claude's current Help Center article (support.claude.com) on 2026-07-16.

## 1. Create the project (2 min)

1. Go to claude.ai/projects (log in first).
2. Click **"+ New Project"**, upper right.
3. Name it **Sable Pulse Prototype**. Description is optional, Claude never reads it, it's just a label for you.
4. Click **Create**.

## 2. Set the project's behavior (2 min)

1. Inside the project, click **"Set project instructions."**
2. Open `prototype/02-claude-project-instructions.md` in this folder, copy everything below the line that says "The text to paste," and paste it into the instructions box.
3. Click **Save instructions.**

## 3. Upload the knowledge files (2 min)

1. On the right side of the project page, find the **Project knowledge** panel.
2. Click **"+"** and upload two files:
   - `context/context-and-tools.md` (your Step 2 schema and semantic layer, already approved)
   - `prototype/03-qa-sql-pairs.md` (the hand-verified example SQL, new)
3. Claude indexes both automatically, no further action needed.

## 4. Start testing (inside the project, not a regular chat)

1. From the project page, click **"New chat."** It must be a chat started from inside the project, a regular chat outside it won't see your instructions or files.
2. Open `prototype/04-test-script-and-screenshots.md` and run the five prompts one at a time, in order.
3. After each answer, take a full-window screenshot of the exchange (question and Pulse's full response, including the SQL). Save it into a new `prototype/screenshots/` folder using the file names suggested in the test script.

## 5. Record the 60-second demo (do this last, once the five tests pass)

The capstone guide calls this out as the single highest-value artifact, worth more than any slide. Use your phone or Loom.
1. Show Pulse correctly answering one clean question (gold question #1 is the natural pick, it's short).
2. Show Pulse correctly refusing the PII request (gold question #8).
3. Keep it under 60 seconds. Don't narrate the setup, just show the two exchanges happening.
4. Save the link (Loom) or the file (phone recording) somewhere you can drop into the README next.

## What NOT to do

- Don't test in a chat outside the project, it won't have your instructions or files loaded.
- Don't skip the "Set project instructions" step and just paste everything into the first chat message, instructions set at the project level apply to every chat automatically; a one-off message doesn't.
- Don't chase the stretch goal (a real SQLite/CSV query path via Make.com or a notebook) unless the five tests are done with time to spare. The capstone guide states explicitly: "the conversational prototype is enough to pass."
