# ALIRA — Software Design Documents from C Source Code

> 한국어: [README.ko.md](README.ko.md)

ALIRA turns C source code into a software design document (SDD) from inside your coding agent. Point it at a project folder, approve the estimate, and download a Word or Markdown document with a summary, requirements, input and output specification, call relations, and a flowchart for each function. Documents are written in Korean.

ALIRA is a hosted service operated by Suresoft Technologies. This repository connects your agent to it: a server address, a Skill that guides the workflow, and documentation. It contains no executable code and does not include the ALIRA service itself. ALIRA is a paid service with a free trial; the MIT License covers only this repository, not the service.

Generated documents are a draft for engineers to review. Check them against the code and your project's requirements before relying on them.

## What ALIRA adds

| Need | What ALIRA does |
|---|---|
| Design documents for existing code | Writes a document per function from the source, covering summary, requirements, input and output, call relations, and a flowchart. |
| Documents your team can use | Exports Word (`.docx`) or Markdown (`.md`) files. |
| Large codebases | Generates up to 100 functions per run, and you can split a project by file or folder. |
| Answers from the documents | Answers questions about a function or module from the generated text. |
| Control over usage | Shows the expected usage before anything starts and waits for your approval. |

## Workflow at a glance

1. Connect your ALIRA account in the browser.
2. Choose a folder or ZIP file, review what will be sent, and approve the upload.
3. Review the estimate and approve generation.
4. Follow progress, read the summary, and save the document.

## Install

You need an ALIRA account, which you can [create on the website](https://alira-ai.ai/login?signup=1).

In Claude Code:

```text
/plugin marketplace add SuresoftTechnologies/alira-ai-agent
/plugin install alira@suresofttech-alira
```

Then run `/mcp`, choose `alira` (it can appear as `plugin:alira:alira`), and select Authenticate to sign in.

To connect the server without the Skill, add it directly:

```text
claude mcp add --transport http --scope user alira https://alira-ai.ai/mcp
```

Without the Skill, your agent decides the order of calls and when to ask for approval. The server still returns the estimate, and the tools that change data are marked destructive so clients that honor the marking ask before each call.

Other MCP clients can connect to `https://alira-ai.ai/mcp` as a remote server using OAuth. Uploading needs an agent that can run shell commands on your machine, such as Claude Code. In claude.ai chat you can use the account, job, and result tools and open a file's download link in your browser, but you cannot upload a local folder.

## Example requests

Responses can vary with the AI model and the conversation. If a result isn't what you expected, rephrase the request with more detail.

| Request | What happens |
|---|---|
| `Make an SDD from the C code in ./firmware.` | Uploads after your approval, then estimates and generates. |
| `SDD 만들어줘.` | Same workflow, in Korean. |
| `How much ALIRA usage do I have left?` | Reports your plan and remaining usage. |
| `How far along is it?` | Reports job progress. |
| `What does init_sensor do?` | Answers from the generated document. |
| `Download the result as Word.` | Saves the file and reports where. |

## Skill

| Skill | Purpose |
|---|---|
| `generate-sdd` | Guides the workflow from connection to download, including the approval points. |

## MCP tools

The tools live on the ALIRA server. Their titles and responses are in Korean.

| Tool | Access | Purpose |
|---|---|---|
| `alira_service_info` | Read | Describe the service, plans, and free trial. |
| `alira_account` | Read | Show the account, organization, plan, and remaining usage. |
| `alira_prepare_upload` | Destructive | Create an upload link and the command to run. |
| `alira_upload_status` | Read | Check the result of an upload. |
| `alira_list_projects` | Read | List uploaded projects. |
| `alira_list_functions` | Read | List the functions in a project. |
| `alira_estimate` | Read | Estimate usage for a generation. |
| `alira_generate_sdd` | Destructive | Start generating a document. |
| `alira_job_status` | Read | Show job progress. |
| `alira_list_jobs` | Read | List jobs. |
| `alira_resume_job` | Destructive | Resume a paused or failed job. |
| `alira_cancel_job` | Destructive | Cancel a job. |
| `alira_get_result` | Read | Summarize a finished document. |
| `alira_read_sdd` | Read | Read the document text. |
| `alira_export_sdd` | Destructive | Create a Word or Markdown file and a download link. |

Any tool that changes data is marked destructive, so clients that honor the marking ask before each call. Read tools can run without a prompt.

## What this plugin runs and sends

The plugin adds no hooks, scripts, or background processes. Using it does the following:

- **Connects** your agent to `https://alira-ai.ai/mcp`. You sign in with OAuth in your browser, and your client holds the token.
- **Runs shell commands the server returns.** The upload command uses `find`, `tar`, and `curl`, or `tar.exe` and `curl.exe` on PowerShell. The download command uses `mkdir`, `curl`, and `ls`, or the PowerShell equivalents. Your agent's normal permission prompts apply, so you can review each command before it runs.
- **Uploads your source code** to the ALIRA server when you approve the upload. With `bash`, only `.c` and `.h` files are sent. **With PowerShell, or with a ZIP file, other files in the folder or ZIP are sent too, including hidden files and folders such as `.env` and `.ssh`.** The server keeps only the `.c` and `.h` files and does not retain the rest. See [PRIVACY.md](PRIVACY.md) for the details.
- **Saves the document** to `~/ALIRA/outputs/<project>/` by default.

## Requirements and pricing

- An ALIRA account. Your organization's plan and usage apply.
- C source code (`.c` and `.h`). Other languages are not supported.
- A shell on your machine for uploading files.

ALIRA is a paid service with monthly plans (Starter, Pro, and Ultra) that differ only in usage level. Organizations that sign up on the website get a free trial: one generation of up to 15 functions and one Markdown download. You choose and pay for plans on the website, in your browser. See [pricing](https://alira-ai.ai/pricing). Your coding agent's own fees are separate.

## Privacy Policy

Using this plugin sends your source code to Suresoft Technologies' ALIRA server, so read this before uploading code you may not disclose.

- Your source code leaves your machine only at the upload step, after you approve it. [PRIVACY.md](PRIVACY.md) lists exactly what each upload method sends.
- Uploaded code and generated files are used only to generate documents and provide them for download. They are not resold, shared with third parties, or used to train AI models.
- The AI model that generates documents runs on servers operated by Suresoft Technologies.
- Uploaded code and generated files are deleted 30 days after their last use by default. You can change the period or delete them at any time in My account.
- Your AI client sends tool results, which can include file paths and document text, to its own model provider under that provider's terms.

Read [PRIVACY.md](PRIVACY.md) and the full [ALIRA SDD Privacy Policy](https://alira-ai.ai/legal?doc=privacy).

## Support

Questions, demos, and purchases: [alira@suresofttech.com](mailto:alira@suresofttech.com) or the [contact page](https://alira-ai.ai/contact). Report problems with this plugin in [GitHub Issues](https://github.com/SuresoftTechnologies/alira-ai-agent/issues).

## License

The contents of this repository, other than the ALIRA name and logo, are under the [MIT License](LICENSE). The ALIRA service is a separate commercial product under its own [Terms of Service](https://alira-ai.ai/legal?doc=terms). This license grants no rights to the service or to the ALIRA name and logo.

Claude and Claude Code are trademarks of Anthropic. Word and PowerShell are trademarks of Microsoft. ALIRA is not affiliated with or endorsed by either company. Other product names may be trademarks of their respective owners.
