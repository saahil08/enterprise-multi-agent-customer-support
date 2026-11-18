# Git version control guide

Use a private GitHub repository for this project. Do not commit credential secrets, customer records, approval payloads or n8n execution exports.

## Suggested repository layout

```text
enterprise-multi-agent-customer-support/
├── workflows/       # reviewed n8n workflow JSON exports
├── originals/       # pre-refactor exports; keep private
├── docs/
├── tests/
├── README.md
└── .gitignore
```

## Initialize locally (PowerShell)

```powershell
cd D:\
mkdir enterprise-multi-agent-customer-support
cd enterprise-multi-agent-customer-support
git init
code .
```

Copy the contents of this release package into the project folder, then:

```powershell
git add .
git commit -m "chore: baseline enterprise customer support workflows"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/enterprise-multi-agent-customer-support.git
git push -u origin main
```

Create a development branch:

```powershell
git checkout -b development
git push -u origin development
```

## Change process

1. Make and test changes in n8n development.
2. Export the changed workflow JSON.
3. Replace the corresponding file under `workflows/`.
4. Review the Git diff in VS Code.
5. Commit to `development`.
6. Run unit tests and the integration checklist.
7. Open a Pull Request to `main`.
8. Merge only after tests pass.
9. Tag a release after production deployment, for example `v1.0.0`.

```powershell
git add workflows docs tests
git commit -m "feat: extract customer support subworkflow"
git push origin development
```

## .gitignore

The package includes a `.gitignore` starter. Keep `.env`, logs, local temporary files and customer data out of Git. Review every workflow JSON before pushing because node parameters can include environment-specific identifiers and internal URLs.
