---
name: generate-sdd
description: Generate a software design document (SDD) from C source code with the ALIRA service and save it as Word or Markdown. Use when the user asks for an SDD, detailed design document, or documentation of a C codebase, mentions ALIRA, or says things like "SDD 만들어줘", "상세설계서 작성해줘", "코드베이스 문서화해줘". Also use to check ALIRA usage or job progress, and to answer questions from a generated SDD.
---

# Generate an SDD with ALIRA

ALIRA analyzes C source code on its own server and writes a function-level software design document in Korean: summary, requirements, input and output specification, call relations, and a flowchart. This skill sets the order of the `alira_*` tool calls and the points where the user must approve. The server does the analysis, generation, and billing.

## Rules

- Source code leaves the user's machine at the upload step. Say what will be sent and where before running the upload, and wait for the user's approval.
- Do not call `alira_generate_sdd` or `alira_export_sdd` until the user has approved the estimate, including on the free trial.
- When a response has a `user_message`, show it unchanged. It carries plan, usage, and payment wording the server keeps accurate.
- The SDD result file is the one ALIRA made. When the user asks for the SDD as a file, a document, Markdown, or Word, call `alira_export_sdd` and hand over that file, as the saved path or as `download.url`. Never write the SDD yourself from `alira_get_result` or `alira_read_sdd` output or from the conversation, whether as a local file, an artifact, or a document in another tool, and never offer such a document in place of the ALIRA file. Those two tools are for summaries and questions only.
- Never add anything ALIRA did not generate to the ALIRA result or to a quote from it: no review comments, evaluations, improvement suggestions, guesses, or commentary. If the user asks for your own opinion, give it apart from the ALIRA result and say it is yours, not ALIRA's.
- Report plan usage as a percentage and a reset date, not as raw numbers.
- Payment, plan changes, and refunds happen only on the website in the user's browser. Never ask for card details.
- Upload and download links contain a token that works as a credential. Use them only in the command that needs them or, when there is no shell, show `download.url` to the user who asked for the file. Do not save them to a file or share them anywhere else.
- Uploading needs a shell on the user's machine. Without one, such as in claude.ai chat, say so and suggest Claude Code. Account, job, and result tools still work, and the user can open `download.url` in a browser to save the file.

## Steps

### 1. Check the connection

Call `alira_account` and report the organization, plan, and remaining usage.

If the `alira_*` tools are not available, or `alira_account` returns `not_connected`, ask the user to connect their ALIRA account. In Claude Code that is `/mcp`, then `alira` (it can appear as `plugin:alira:alira`), then Authenticate. The sign-in page opens in the browser, and new users can create an account there.

### 2. Upload the project

Ask which folder or ZIP file to analyze. To reuse an earlier upload, call `alira_list_projects` instead and go to step 3.

1. Call `alira_prepare_upload(project_name, source)`, with `source` set to `"folder"` or `"zip"`.
2. For a folder, run `preview_command` with `PROJECT_DIR='/absolute/path'; ` in front of it. Tell the user how many `.c` and `.h` files will be sent, which folders are skipped, and that the destination is alira-ai.ai. Wait for approval.
3. Run `commands.bash` with the same prefix, or with `ZIP_PATH='/absolute/path.zip'; ` for a ZIP.
4. Report `project_id`, the function count, and the excluded items from the JSON output. If the output is missing or cut off, call `alira_upload_status(upload_id)`. `PENDING` means the upload has not arrived, so check whether the command ran. `RUNNING` means the server is analyzing the source, so check again shortly. `FAILED` means you report `error`, check the path, and retry with the same link while it is valid or with a new one from `alira_prepare_upload`.

Use `commands.powershell` only on Windows without Git Bash, with `$env:PROJECT_DIR='C:\path'; ` in front, or `$env:ZIP_PATH='C:\path\file.zip'; ` for a ZIP. The folder command sends the whole folder except a fixed list of skipped folders, hidden files and hidden folders such as `.env` and `.ssh` included, and the server keeps only the `.c` and `.h` files. A ZIP is also sent whole. Tell the user this before asking for approval, and suggest Git Bash or a ZIP containing only source files if the folder holds anything sensitive.

### 3. Narrow the scope (optional)

To cover only part of the project, call `alira_list_functions(project_id, file_filter)`. A target is `file.c` for a whole file or `file.c:function` for one function, with paths relative to the uploaded folder.

### 4. Estimate and approve

Call `alira_estimate(project_id, targets, options)`. Show `user_message`, which gives the expected usage and what remains, then ask whether to proceed with these settings.

- `too_many_units`: one run covers up to 100 functions. Split the targets by file or folder and estimate and generate each batch in turn.
- Usage limit reached or no plan: show `user_message` and end the turn. Do not ask again for the folder or settings when the user returns. After a usage limit stop, the user has seen the expected usage, so call `alira_generate_sdd` with the same `retry_args` without asking again. After a no-plan stop, the estimate showed no usage figure, so call `alira_estimate` with the same arguments, show `user_message`, and ask once.

### 5. Generate

After approval, call `alira_generate_sdd` with the `retry_args` from the estimate and a new unique `request_id`, and keep the `job_id`. On `insufficient_credit`, show `user_message` and end the turn. When the user returns, call `alira_generate_sdd` with `retry.args` from that response. When retrying the same request, pass the same `request_id` so no second job starts.

### 6. Follow progress

Call `alira_job_status(job_id)` and report `progress_text`. If the user asks you to wait, repeat it with `wait_seconds=60`.

- `PAUSED_INSUFFICIENT_CREDIT`: show `user_message`. The job continues on its own from the next function when usage resets or a payment is confirmed. With no plan or a scheduled cancellation, usage does not reset, so only a new payment continues it. If `auto_resume` is false, the job continues only through `alira_resume_job(job_id)`.
- `PAUSED_ESTIMATE_EXCEEDED`: the job passed its maximum estimate. Show `paused_reason` and `remaining_estimate`, which gives the remaining functions and the expected and maximum usage as a percentage, and call `alira_resume_job(job_id)` only after the user approves.
- `PAUSED_INTERRUPTED` or a failed job: offer `alira_resume_job(job_id)`. A failed job's `error` is a code, and the reason is in `paused_reason`.
- `alira_cancel_job(job_id)` needs the user's confirmation first. On a running job it returns `cancel_requested=true`: the job stops after the current batch and can still finish as completed.

### 7. Report the result

When the job is complete, call `alira_get_result(job_id)` and report the document list, the main content, the analyzed scope and exclusions, the usage, and the files available.

For questions about a function or module, call `alira_read_sdd(job_id, target=..., file=..., query=...)` and answer from the returned text. If a name is not found, look it up in the document list from `alira_get_result`. If the document does not cover it, say that any further answer is a guess.

Both tools return lookup data, not the result file. If the user asks for the result as a file or a document at this point, go to step 8 instead of composing one from these responses.

### 8. Download

Call `alira_export_sdd(job_id, format)`. Omit `format` for the default, which is Markdown on the free trial and Word on a paid plan. Run `download.commands.bash` once, or `download.commands.powershell` on Windows without Git Bash, and report the saved path. The default location is `~/ALIRA/outputs/<project>/`. If the user wants another location, change only the output path in the command. Save inside the project folder only if the user allows it.

Deliver the file as ALIRA made it. Do not edit it, append to it, or rebuild it in another format with added content. If the user wants the SDD in another tool or format, start from the downloaded ALIRA file and keep its content as it is. Without a shell the file cannot be read here, so give `download.url` and say that the ALIRA file is the result.

- The free trial provides one Markdown download and no re-download, so run the command only once. If the user asks for Word on the free trial, say that the free trial provides `.md` files only.
- `plan_required`: show `user_message` and end the turn. Do not call `alira_export_sdd` again with another format unless the user asks.
- For an unfinished job, set `allow_partial=true` only if the user asks for a partial result.

## Limits

- C source only (`.c` and `.h`).
- Documents are written in Korean.
- Up to 100 functions per run.
- Free trial: one generation of up to 15 functions and one Markdown download, for organizations that signed up on the website.
- ALIRA is a paid service with monthly plans after the free trial. Questions and purchases: alira@suresofttech.com or https://alira-ai.ai/contact.
