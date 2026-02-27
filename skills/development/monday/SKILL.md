---
name: monday
description: "Read and query Monday.com boards, items, workspaces, and users directly via the Monday.com GraphQL API. Use when you need project/task data, board contents, or team info. Calls api.monday.com directly with no third-party proxy."
metadata:
  openclaw:
    requires:
      env:
        - MONDAY_API_TOKEN
      bins:
        - python3
    primaryEnv: MONDAY_API_TOKEN
    files:
      - "scripts/*"
---

# Monday.com

Read boards, items, and workspaces directly via `api.monday.com` (GraphQL).

## Setup (one-time)

1. In Monday.com, click your **profile picture** (top right)
2. Select **Developers** — this opens the Developer Center
3. Click **API token → Show**
4. Copy your personal token
5. Set the environment variable:
   ```
   MONDAY_API_TOKEN=your_token_here
   ```

## Commands

### Get your account info
```bash
python3 /mnt/skills/user/monday/scripts/monday.py me
```

### List all boards
```bash
python3 /mnt/skills/user/monday/scripts/monday.py list-boards
python3 /mnt/skills/user/monday/scripts/monday.py list-boards --limit 50
```

### Get board details (columns, groups)
```bash
python3 /mnt/skills/user/monday/scripts/monday.py get-board <board_id>
```

### List items on a board
```bash
python3 /mnt/skills/user/monday/scripts/monday.py list-items <board_id>
python3 /mnt/skills/user/monday/scripts/monday.py list-items <board_id> --limit 50
```

### List workspaces
```bash
python3 /mnt/skills/user/monday/scripts/monday.py list-workspaces
```

### List users
```bash
python3 /mnt/skills/user/monday/scripts/monday.py list-users
```

## Notes

- Free plan: 2 seats, unlimited boards. API access works on free.
- Board IDs are numeric — find them in the board URL or via `list-boards`.
- Monday uses GraphQL only (single endpoint). No REST API.
- API version pinned to `2024-04`.
