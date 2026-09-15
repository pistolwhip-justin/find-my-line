# Create the Find-My-Line GitHub Project

The repository contains a manually triggered GitHub Actions workflow at:

`.github/workflows/create-project.yml`

The workflow creates **Find-My-Line Development** as a user-owned GitHub Project and creates the initial development backlog as repository Issues, then adds those Issues to the Project.

## One-time setup

GitHub's normal `GITHUB_TOKEN` cannot access GitHub Projects. The workflow therefore expects a separate token in a repository secret named:

`FIND_MY_LINE_PROJECT_TOKEN`

GitHub's documentation states that a classic personal access token with the `project` scope can be used for user-owned Projects.

Create the token, then add it to:

**Repository → Settings → Secrets and variables → Actions → New repository secret**

Name:

`FIND_MY_LINE_PROJECT_TOKEN`

Value:

Your GitHub personal access token.

Do not put the token in this repository or in this file.

## Run it

On GitHub:

1. Open the **Find-My-Line** repository.
2. Open **Actions**.
3. Select **Create Find-My-Line Project**.
4. Select **Run workflow**.
5. Leave `recreate` set to `false`.
6. Select **Run workflow**.

The workflow will create the Project and the initial backlog.

If the Project already exists, a normal run safely stops after reporting the existing Project URL.

## What it creates

Project:

**Find-My-Line Development**

Initial Issues:

1. Define route data model
2. Select initial geographic test region
3. Obtain and normalize OpenStreetMap routing data
4. Evaluate BRouter integration
5. Build first bikepacking routing profile
6. Implement start and destination routing
7. Implement alternative-route generation
8. Calculate route statistics
9. Build MVP route-results UI
10. Implement GPX export
11. Test generated routes against real-world riding conditions
12. Improve routing weights from real-world failures
13. Add authoritative trail and land-management datasets
14. Expand export formats
15. Evaluate navigation-app integrations

The Project is intentionally centered on **route planning and route generation**, not turn-by-turn navigation.
