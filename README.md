# Project Version Bumper

**Project Version Bumper** is a lightweight GitHub Action that automatically increments a project version or its build number for a given project (or project variant).

---

## 🚀 Features
- Increments and persists a version or build number for any project.
- Stores data in a separate GitHub repository.
- Outputs the value for use in subsequent steps (e.g. tagging, build, release).

---

## 🛠️ Usage

### 1. **Prepare a storage repository**

Create a separate repository to store version files, e.g. `MyOrganisation/version-bumper-storage`.

### 2. **Prepare a GitHub Token with write access**

You can use your Personal Access Token or other token capable of accessing and writing the storage repository.

### 3. **Add step to your GitHub Action workflow**

```yaml
    steps:
      - name: Bump build number
        id: build_number
        uses: chris-rutkowski/version-bumper@main
        with:
          project: "my-app-build-number"
          repository: "MyOrganisation/version-bumper-storage"
          token: ${{ secrets.GITHUB_PAT }}

      - name: Fastlane (example)
        run: fastlane beta build_number:${{ steps.build_number.outputs.value }}
```

### 4. **Using readonly mode (optional)**

You can use the action in readonly mode to retrieve the current value without incrementing it:

```yaml
    steps:
      - name: Get current version
        id: version
        uses: chris-rutkowski/version-bumper@main
        with:
          project: "my-app-version"
          repository: "MyOrganisation/version-bumper-storage"
          token: ${{ secrets.GITHUB_PAT }}
          readonly: true
      
      # use ${{ steps.version.outputs.value }}
```

### 5. **Using prefix and suffix for version-like numbering (optional)**

You can add a prefix to create version-like numbering:

```yaml
    steps:
      - name: Bump version
        id: version
        uses: chris-rutkowski/version-bumper@main
        with:
          project: "my-app-version"
          repository: "MyOrganisation/version-bumper-storage"
          token: ${{ secrets.GITHUB_PAT }}
          prefix: "4.17."
          suffix: "-alpha"


      # use ${{ steps.version.outputs.value }}
```

This will output a version like `4.17.1-alpha`, incrementing the `.1` each time the action runs.

### 6. **Combining version and build number**

You can get the current version in readonly mode and increment a separate build number:

```yaml
    steps:
      - name: Get current version
        id: version
        uses: chris-rutkowski/version-bumper@main
        with:
          project: "my-app-version"
          repository: "MyOrganisation/version-bumper-storage"
          token: ${{ secrets.GITHUB_PAT }}
          prefix: "4.17."
          suffix: "-alpha"
          readonly: true

      - name: Bump build number
        id: build_number
        uses: chris-rutkowski/version-bumper@main
        with:
          project: "my-app-build-number"
          repository: "MyOrganisation/version-bumper-storage"
          token: ${{ secrets.GITHUB_PAT }}

      # use ${{ steps.version.outputs.value }} and ${{ steps.build_number.outputs.value }}
```

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
