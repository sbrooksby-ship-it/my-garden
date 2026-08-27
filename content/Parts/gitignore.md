
2026-08-27 11:04

Status:

Tags: [[Github]] [[DP13]]


# gitignore

The `.gitignore` file tells Git which untracked files or directories to intentionally ignore, preventing unwanted files from being added to your project's version control history. - **Security:** Prevents sensitive data—such as API keys, database credentials, secret tokens, and `.env` files—from being accidentally committed and exposed publicly on platforms like GitHub.
    
- **Excluding Dependencies & Artifacts:** Keeps heavy, auto-generated files like `node_modules/`, `build/`, `dist/`, or compiled binaries out of the repository, as these can easily be restored using package managers or build tools.
    
- **Cleaning Up System Overhead:** Ignores OS-generated metadata (`.DS_Store`, `Thumbs.db`) and local editor settings (`.vscode/`, `.idea/`) so team members don't constantly overwrite each other's local environments.
    
- **Performance & Readability:** Reduces total repository size, keeps `git status` output clean, and ensures code reviews focus exclusively on meaningful code changes rather than auto-generated noise.

Below is my code for the gitignore for adverse events:

token.pickle
state.json
client_secret.json
__pycache__/
*.png



### References
[[Adverse Events Automation]]