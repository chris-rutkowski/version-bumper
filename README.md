
# Project Version Bumper

**Project Version Bumper** is a lightweight GitHub Action that automatically increments a build number for a given project (or project variant).

This is useful for maintaining auto-incremented build numbers, synchronised versioning across projects, or tracking deploy counts.

It works by cloning a designated version repository, updating a numeric counter (e.g., `42` → `43`), and committing the updated value back.

---

## 🚀 Features
- Increments and persists a build number for any project.
- Stores version data in a separate GitHub repository.
- Outputs the new build number for use in subsequent steps (e.g. tagging, build, release).

---

## 🛠️ Usage

### 1. **Prepare a version storage repository**

Create a separate repository to store version files, e.g. `MyOrganisation/version-bumper-storage`.

### 2. **Prepare a GitHub Token with write access**
You can use your Personal Access Token or other token capable of accessing and writing the storage repository.

### 3. **Add two steps to your GitHub Action workflow**

```yaml
    steps:
      - name: Bump version
        id: version_bumper
        uses: chris-rutkowski/version-bumper@main
        with:
          project: "my-app" # or "my-app-test" - filesafe name
          repository: "MyOrganisation/version-bumper-storage"
          token: ${{ secrets.GITHUB_PAT }}

      - name: Use new build number
        run: echo "Build number: ${{ steps.version_bumper.outputs.value }}"
```

### 4. **Using readonly mode (optional)**

You can use the action in readonly mode to retrieve the current build number without incrementing it:

```yaml
    steps:
      - name: Get current version
        id: version_bumper
        uses: chris-rutkowski/version-bumper@main
        with:
          project: "my-app"
          repository: "MyOrganisation/version-bumper-storage"
          token: ${{ secrets.GITHUB_PAT }}
          readonly: true

      - name: Use current build number
        run: echo "Current build number: ${{ steps.version_bumper.outputs.build_number }}"
```

### 5. **Using prefix for version-like numbering (optional)**

You can add a prefix to create version-like numbering:

```yaml
    steps:
      - name: Bump version
        id: version_bumper
        uses: chris-rutkowski/version-bumper@main
        with:
          project: "my-app"
          repository: "MyOrganisation/version-bumper-storage"
          token: ${{ secrets.GITHUB_PAT }}
          prefix: "4.17."

      - name: Use versioned build number
        run: echo "Version: ${{ steps.version_bumper.outputs.value }}"
        # This will output: Version: 4.17.1, 4.17.2, 4.17.3...
```

**Examples of prefix usage:**
- `prefix: "1."` → outputs: `1.1`, `1.2`, `1.3`...
- `prefix: "5.1."` → outputs: `5.1.1`, `5.1.2`, `5.1.3`...
- `prefix: "v"` → outputs: `v1`, `v2`, `v3`...
- No prefix (default) → outputs: `1`, `2`, `3`...

---

## 📝 Inputs

| Input        | Description                                                                   | Required | Default      |
| ------------ | ----------------------------------------------------------------------------- | -------- | ------------ |
| `project`    | Project name (used as filename for version storage)                           | Yes      | -            |
| `repository` | Repository to store version data (format: "owner/repo")                       | Yes      | -            |
| `token`      | GitHub token with write access to the repository                              | Yes      | -            |
| `readonly`   | If true, returns current value without incrementing or saving                 | No       | `false`      |
| `prefix`     | Prefix to prepend to the build number (e.g., "1." for version-like numbering) | No       | `''` (empty) |

## 📤 Outputs

| Output  | Description                                                               |
| ------- | ------------------------------------------------------------------------- |
| `value` | The version value with optional prefix (incremented unless readonly=true) |

---

## 🧾 Setting initial versions

After running this workflow for the first time, the version storage repository will contain a file named after your `project` input, such as `my-app.txt`. This file holds a single number that represents the current build version and is **incremented by 1** every time the workflow runs. You can manually edit this file at any time to set an initial version, reset the counter (by deleting a file), or jump to any specific version number as needed, giving you full control over versioning across your environments or deployment flows.

## 🗂️ Supports Multiple Projects
A single storage repository can track versions for any number of projects. Each project is assigned its own `.txt` file based on the `project` input and these files are managed independently.

To mitigate potential concurrency issues, you can either split storage across multiple repositories or enhance this action by introducing locking, retry logic, or queueing mechanisms depending on your scale and criticality.

## 📄 License
This project is licensed under the [MIT License](LICENSE).
