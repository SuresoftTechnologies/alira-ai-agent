# Privacy Policy

This policy covers the ALIRA plugin in this repository and what happens when you use it with the ALIRA service. The [ALIRA SDD Privacy Policy](https://alira-ai.ai/legal?doc=privacy) is the full policy for the service, and it prevails if the two differ.

Last updated: 2026-11-06

## What is collected

The plugin contains no executable code of its own: a server address, a Skill, and documentation. It adds no telemetry. Everything below is collected by the ALIRA service, which you connect to at `https://alira-ai.ai/mcp`, once you sign in and use it.

- **Account details:** name, email address, username, company or organization, job title, and, if you sign in with Google or GitHub, the account identifier, email address, and profile image URL that provider returns.
- **Uploaded content:** the C source code you choose to upload and its parsing results, and the design documents and exported files generated from it.
- **Service records:** organization identifier, job history, usage ledger, payment order history, and whether the free trial has been used.
- **Connection records:** the registration details your AI client sends when it connects, hashed sign-in tokens with their expiry and last-use times, and upload and download link records.
- **Payment details:** order number, payment method type, card issuer name, and the masked card number and billing key identifier returned by the payment provider. ALIRA does not collect or store full card numbers, expiry dates, or CVCs.

## What leaves your machine

**Your source code is uploaded to the ALIRA server.** This happens only at the upload step, when your agent runs a command that the server provides and you allow it to run. What is sent depends on the command:

| Upload method | What is sent |
|---|---|
| Folder, using `bash` (Linux, macOS, Git Bash on Windows) | Only `.c` and `.h` files. Hidden files, hidden folders, and the folders `build`, `dist`, `out`, `bin`, `obj`, `node_modules`, `venv`, `__pycache__`, `cmake-build-debug`, and `cmake-build-release` are skipped on your machine. |
| Folder, using PowerShell (Windows without Git Bash) | Everything in the folder except `.git`, `.svn`, `.vs`, `.vscode`, `.idea`, and the folders listed above. This includes other file types and hidden files and folders such as `.env` and `.ssh`. |
| ZIP file | The whole ZIP file. |

For the PowerShell and ZIP methods, the server keeps only `.c` and `.h` files and does not retain the rest. If your code contains personal or sensitive information, remove it before uploading, or confirm what your organization permits.

**Your AI client also sends data, to its own provider.** Tool results, which can include file paths, function names, and generated document text, go to the AI model behind your client so that it can act on them. That transfer is between you and your AI provider and is governed by their privacy policy (for Claude, Anthropic's). Suresoft Technologies is not a party to it.

## How data is used

Uploaded source code and generated files are used only to generate documents and provide them for download. They are not resold, shared with third parties, or used to train AI models. Rights to your source code and output files remain with you.

The AI model that generates documents runs on servers operated by Suresoft Technologies. ALIRA does not provide your source code or output files to any external AI provider, and does not itself transfer them overseas.

## Third parties

ALIRA outsources two tasks:

- **Toss Payments Co., Ltd.:** payment processing. Your email address, name or organization name, and order details are passed on when you pay.
- **Daou Technology Inc. (DaouOffice):** delivery of service email such as verification and password reset messages.

## Retention

Uploaded source code, parsing results, generated results, and exported files are deleted automatically 30 days after their last use. In My account, under Data management, you can set your own period from 1 to 3,650 days, turn automatic deletion off, or delete projects and files immediately. Deleting your account deletes this data immediately.

Upload and download links expire after a short period. Your AI client holds the sign-in tokens for its connection, and you can clear them there. Other records follow the retention periods in the full policy, including those required by law.

## Changes to this policy

Changes to this file are visible in this repository's commit history. Changes to the service's policy are announced on the ALIRA website.

## Contact

Questions about this policy or about your data: [alira@suresofttech.com](mailto:alira@suresofttech.com)

Suresoft Technologies Inc. — <https://www.suresofttech.com>
