---
name: brb-sentry-update
description: Pull the latest sentry workflow skills from GitHub and update your local install.
allowed-tools: Bash(*)
---

# Update Sentry Workflow Skills

Pull the latest skills from the claude-sentry-workflow GitHub repo and update the local installation.

## Step 1: Detect install location

```bash
# Check global
GLOBAL_DIR="$HOME/.claude/skills"

# Check project
PROJECT_DIR=".claude/skills"

# Find where brb-sentry-workflow is installed
if [ -d "$GLOBAL_DIR/brb-sentry-workflow" ]; then
  TARGET="$GLOBAL_DIR"
  echo "Found global install at $TARGET"
elif [ -d "$PROJECT_DIR/brb-sentry-workflow" ]; then
  TARGET="$PROJECT_DIR"
  echo "Found project install at $TARGET"
else
  echo "No sentry workflow installation found."
  echo "Install first: bash <(curl -fsSL https://raw.githubusercontent.com/graycyrus/claude-sentry-workflow/main/install.sh)"
  exit 1
fi
```

## Step 2: Check if symlink install

```bash
if [ -L "$TARGET/brb-sentry-workflow" ]; then
  LINK_TARGET=$(readlink "$TARGET/brb-sentry-workflow")
  REPO_DIR=$(dirname "$(dirname "$LINK_TARGET")")
  echo "Symlink install detected — just run: cd $REPO_DIR && git pull"
  cd "$REPO_DIR" && git pull
  echo "Updated!"
  exit 0
fi
```

## Step 3: Download and update (copy install)

```bash
TMP_DIR=$(mktemp -d)
git clone --quiet --depth 1 https://github.com/graycyrus/claude-sentry-workflow.git "$TMP_DIR"

SKILLS="brb-sentry-workflow brb-sentry-triage brb-sentry-analyze brb-sentry-raise-issue brb-sentry-update"

echo "Updating skills..."
for skill in $SKILLS; do
  if [ -d "$TMP_DIR/skills/$skill" ]; then
    rm -rf "${TARGET:?}/$skill"
    cp -r "$TMP_DIR/skills/$skill" "$TARGET/$skill"
    echo "  Updated /$skill"
  fi
done

rm -rf "$TMP_DIR"
echo "Done!"
```

## Step 4: Show what changed

After updating, show a brief diff summary of what changed (if git is available in the target).
