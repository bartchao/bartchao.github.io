---
title: Azure DevOps Build Pipelines to Github Actions via Github Copilot
date: 2026-01-18 11:31:21
tags: 
    - AI
    - GitHub Copilot
    - Azure DevOps
    - GitHub Actions
categories: 
    - Technical Notes
excerpt: An AI helper repository which help to convert Azure DevOps Build Pipeline to GitHub Actions
---
# Project Overview

This ADO to GitHub Agent Framework is designed to migrate repositories and CI pipelines from Azure DevOps to GitHub with GitHub Actions. Many CI pipelines use a fixed set of tasks, so the rewriting process can be delegated to AI. The AI can convert Build Pipeline definitions into GitHub Action Workflows.

## Environment Analysis

- Dozens of Build Pipelines of roughly the same type
    - Most of the work involves Docker Build & Push to Artifactory, and copying OpenShift Template YAML files to Azure Artifacts (now moving to GitHub Artifacts → GitHub Release)
- Due to company security restrictions, only internal GitHub Actions or official GitHub Actions can be used. This means AI's general knowledge is less useful, so we need to first figure out which GitHub Actions to use for this type of Build Pipeline. (In the future, an MCP Server should provide available GitHub Actions to the AI)

# Technical Architecture

- GitHub Copilot in VSCode
- Python, Windows PowerShell, Azure CLI with Azure DevOps extension, GitHub CLI
- Azure DevOps PAT Token + GitHub account

# Development Process

1. **Create GitHub Workflow Template**: I manually created a GitHub Action Workflow Template suitable for this scenario. After testing, it works when changing variable values. Comments were added where AI needs to fill in variables.
    
    The Template follows these general fixed steps:
    
    1. Generate Build Number (ex: 26010.1, DayOfYear)
    2. Build docker image and push to Artifactory
    Many inputs here depend on ADO information and are not fixed
    3. Copy certain YAML files from the repo to become GitHub Release files
2. **Develop Python Script to download ADO Build Pipeline definition**: I asked AI to help write a Python Script that uses Azure CLI to find the corresponding Build Pipeline based on Repository Name. Since the company uses the legacy Build Pipeline (GUI interface, not YAML), the downloaded file is in JSON format with many fields represented by GUIDs. Besides Build Definition, Variables mapping also needs to be downloaded.
3. **Help AI understand conversion logic**: At this point, both source and target files are ready. Next, the AI needs to understand the conversion requirements. Since the Build Pipeline and Variables contain many GUIDs, I explained the meaning of these GUIDs to the AI through Copilot Chat, such as which Artifactory URL a certain GUID represents, repository locations (developers need to manually convert GUIDs to their corresponding meanings), and which repository to push to when building different branches. Finally, I had Copilot generate `copilot-instuctions.md`, detailing how to read the ADO Build Pipeline and convert it to a Workflow.
4. **Discovered insufficient automation**: Although all components needed for migration were ready, there was still a gap to full automation—we still needed to manually execute the Python Script, load `copilot-instuctions.md`, input prompts, validate, and open PRs.
5. **Build Agent Framework**: Just as GitHub Copilot opened up custom Agent functionality (similar to Claude Code's Agent Skills), I designed several Agents:
    1. Orchestrator Agent: Manages the following 4 Agents
    2. Download Agent: Executes Python Script
    3. Analysis Agent: Reads the downloaded JSON and outputs a plain-language `ANALYSIS.md` based on the definition format
    4. Conversion Agent: Reads `ANALYSIS.md` and fills in the Template according to `copilot-instuctions.md`
    5. Git Agent: Opens PRs only for the newly added YAML
    
    ```
    .
    ├── .github/
    │   └── agents/              # Stores task descriptions for each Agent
    │   └── copilot-instuctions.md  # Guidelines for converting analysis documents to Workflows
    ├── .ado-gh/                 # Stores Python Scripts, downloaded JSON, and ANALYSIS.md
    │   └── templates/           # Stores predefined GitHub Action Workflow Templates
    └── .vscode/    
        └── settings.json        # Contains Copilot settings and Auto Approve Command List
    ```
    
6. **Complete automation flow**: After the architecture was complete, simply placing these files in the repository to be converted, switching to the Orchestrator Agent, and entering a simple prompt automatically completes the entire process.
7. **Create deployment Script**: To facilitate deploying these files, I again asked AI to help write a PowerShell Script, from environment checks to downloading AI-required files from GitHub. This way I can repeatedly call this Script in each Repository to complete environment checks and download the Agent Framework, making it convenient for the team to use. This part was inspired by Spec Kit's installation process.
8. **Testing**: Tested with several slightly different repos on hand, and each result may vary slightly.
For example: AI might delete or modify the fixed Template instead of just filling in blanks, requiring going back to add restrictions in `copilot-instuctions.md`

## Benefits

For similar Build Pipelines, we now have a complete AI workflow. When doing large-scale migration in a short time, we can open multiple workspaces simultaneously to let AI complete the work. We only need to review the PRs, and can even use the Actions Workflow execution results to verify whether it's a usable GitHub Actions Workflow.

# Reference Resources

1. [Creating custom agents - GitHub Docs](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
2. [About custom agents - GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-custom-agents)
3. [🔥Sub Agents in VS Code just got a massive upgrade](https://www.reddit.com/r/GithubCopilot/comments/1oyk7ig/sub_agents_in_vs_code_just_got_a_massive_upgrade/)
4. [code.visualstudio.com](http://code.visualstudio.com) 
5. https://github.com/github/spec-kit
6. [開始使用 Azure DevOps CLI - Azure DevOps](https://learn.microsoft.com/zh-tw/azure/devops/cli/?view=azure-devops)
7. [GitHub CLI](https://cli.github.com/)