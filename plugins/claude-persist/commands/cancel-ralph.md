---
description: "Cancel active Ralph Wiggum loop"
allowed-tools: ["Bash(jq:*)", "Bash(ls:*)", "Bash(rm:*)", "Bash(test:*)", "Bash(grep:*)", "Bash(stat:*)", "Bash(date:*)", "Bash(basename:*)", "Bash(sed:*)"]
hide-from-slash-command-tool: "true"
---

# Cancel Ralph

Cancel the active Ralph loop. Find YOUR session ID first:

```bash
# ⭐ Claude Code substitutes ${CLAUDE_SESSION_ID} into this command text with THIS session's id
# (2026-11-16). The old discovery looked only in the isaac-life-corpus project dir, whose sessions
# no longer exist, so from every other corpus SESSION_ID was empty and the Stop hook (which keys
# /tmp/ralph-loop-<session_id>.md by its real id) never found a state file: the loop never armed.
# Fallback for a build without the substitution: newest transcript in ANY project dir.
SESSION_ID="${CLAUDE_SESSION_ID}"
case "$SESSION_ID" in *[!0-9a-f-]*|"") SESSION_ID="" ;; esac
NEWEST_AGE=999999
[ -n "$SESSION_ID" ] || for f in /tmp/statusline-*.json; do
  sid=$(jq -r '.session_id // ""' "$f" 2>/dev/null)
  transcript=$(ls "$HOME"/.claude/projects/*/"${sid}".jsonl 2>/dev/null | head -1)
  if [ -n "$sid" ] && [ -f "$transcript" ]; then
    age=$(( $(date +%s) - $(stat -c %Y "$transcript") ))
    if [ "$age" -lt 60 ] && [ "$age" -lt "$NEWEST_AGE" ]; then
      NEWEST_AGE="$age"
      SESSION_ID="$sid"
    fi
  fi
done

STATE_FILE="/tmp/ralph-loop-${SESSION_ID}.md"
if [ -f "$STATE_FILE" ]; then
  ITERATION=$(grep '^iteration:' "$STATE_FILE" | sed 's/iteration: *//')
  rm -f "$STATE_FILE"
  echo "Cancelled Ralph loop (was at iteration $ITERATION)."
else
  echo "No active Ralph loop for this session."
fi
```
