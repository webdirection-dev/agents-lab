# jenkins-ci-analyst: instruction file

This file is executed by a general-purpose subagent that Kraken launches with the command «2 sub». You are that subagent: follow these instructions exactly.
Tool restrictions below are rules you must obey, not technical limits. You have no network access: work only with local files.

## Inputs
- Saved Jenkins config.xml files: config/jenkins/ (one or more files, in the project root)
- Repository map (URL to local path), one pair per line: config/repo-map.txt
  The URL is the first word of the line, everything after the first space is the local path (it may contain spaces and may be a Windows path like C:\work\repo). Ignore lines starting with #.

## Objective
Explain how CI/CD works from the point of view of someone diagnosing a Kubernetes cluster: which Jenkins jobs exist, where they take code from, which steps they run and where they deploy. Write the findings to the knowledge file.

## What to find
From each config.xml:
- Job name and type (freestyle, pipeline, multibranch)
- Source repository URLs and branches
- Build and deploy steps, scripts, Jenkinsfile references
- Targets: registries, namespaces, clusters, external URLs
- Credential IDs used (IDs only)

From each locally cloned repository found in the map:
- Pipeline definitions (Jenkinsfile, scripts, Helm charts, manifests)
- Where images are built and pushed, and how deployments are triggered

## Missing repositories
For every repository URL found in a config.xml that is NOT in the repository map, or whose local path does not exist:
- Do NOT try to download or search for it.
- List it in the not_cloned section of the knowledge file.
- Report it on the very first line of your reply.

## Constraints
- Do NOT modify any file except the knowledge file.
- Bash is allowed only for read-only git commands such as git rev-parse HEAD. Never run commit, checkout, pull, fetch, push.
- Write ONLY to: knowledge/jenkins-ci.md (in the project root, the folder that contains AGENTS.md).
- Never copy secrets, passwords, tokens or private keys into the knowledge file. Record credential IDs only.
- If something cannot be determined, write unknown instead of guessing.

## Knowledge file format
Plain dense lines for a machine reader: no markdown, no bold, no tables, no indentation, no blank lines.
Header lines are key: value. Each section starts with a line section: <name>. One fact per line, ending with | src: <file>. Use unknown: <what> for anything you could not determine.

source: jenkins-ci
analyzed_at: <YYYY-MM-DD>
configs_analyzed: <comma-separated config.xml files>
repo: <REPO_URL> | commit: <git commit hash analyzed>
section: jenkins.job.<job name>
type: <freestyle|pipeline|multibranch> | src: <config.xml file>
source_repo: <REPO_URL> branch: <branch> | src: <config.xml file>
step: <build or deploy step> | src: <config.xml file or script>
target: <registry, namespace, cluster, URL> | src: <file>
credential_id: <id> | src: <config.xml file>
section: repo.<repository name>
<pipeline or deploy fact> | src: <path>
section: not_cloned
missing: <REPO_URL>
section: unknowns
unknown: <what could not be determined>

## Report (your reply to the main agent)
First line, exactly one of:
- Missing local repositories: <N>   (followed by the list of URLs)
- All repositories found
Then a summary of 3-5 lines only.
