# Deploying RagVerify for free (Streamlit Community Cloud)

This guide puts the app on a public link such as
`https://ragverify-demo.streamlit.app` that anyone can open. No coding is
needed. It takes about 15 minutes.

**How payment works in this setup:** every visitor types their *own* OpenAI
API key into the sidebar. Hosting is free, and you are never charged for
other people's questions. The key stays in that visitor's browser session
only; it is never saved on the server.

---

## What you need

1. A free **GitHub** account: <https://github.com/signup>
   (GitHub stores your code online. The hosting service reads it from there.)
2. Nothing else. Streamlit Community Cloud signs in with your GitHub account.
3. To *try* the app yourself, an OpenAI API key from
   <https://platform.openai.com/api-keys> with some credit on the account.

---

## Step 1: Put the code on GitHub

Choose **one** of the two options.

### Option A: in the browser (easiest)

1. Sign in to GitHub and click **+** (top right) → **New repository**.
2. **Repository name:** for example `ragverify`.
   Choose **Public** (the free hosting plan needs a public repository, or a
   private one you grant access to).
   Leave "Add a README" **unticked** (the project already has one).
   Click **Create repository**.
3. On the next page, click the link **uploading an existing file**.
4. Open the project folder on your computer (the folder that contains
   `app.py`). Select **everything inside it**, including the `.github` and
   `.streamlit` folders, and drag it into the browser window.
   - Drag the *contents*, not the outer folder. `app.py` and
     `requirements.txt` must end up at the top level of the repository, or the
     host cannot find them.
   - Do **not** upload a file named `.env` if you created one. It may hold
     your key. (The project's `.gitignore` protects you from this when using
     Git, but not when dragging files by hand.)
5. Wait for the uploads to finish, then click **Commit changes**.
6. Check: the repository's main page should list `app.py`,
   `requirements.txt`, `ragverify/` and the other files directly.

### Option B: with Git on the command line

Run these in a terminal opened **inside** the project folder (replace
`YOUR-NAME/ragverify` with your repository's address from GitHub):

```powershell
git init
```
```powershell
git add .
```
```powershell
git commit -m "Initial commit"
```
```powershell
git branch -M main
```
```powershell
git remote add origin https://github.com/YOUR-NAME/ragverify.git
```
```powershell
git push -u origin main
```

`git add .` respects `.gitignore`, so a `.env` file is never uploaded.

---

## Step 2: Deploy on Streamlit Community Cloud

1. Go to <https://share.streamlit.io> and click **Continue with GitHub**.
   Approve the access request (Streamlit needs to read your repositories).
2. Click **Create app** → **Deploy a public app from GitHub**
   (wording may vary slightly).
3. Fill in:
   | Field | Value |
   |---|---|
   | Repository | `YOUR-NAME/ragverify` |
   | Branch | `main` |
   | Main file path | `app.py` |
   | App URL | pick a name, e.g. `ragverify-demo` |
4. Open **Advanced settings**:
   - **Python version:** `3.12`
   - **Secrets:** leave **empty**. See "Important" below.
   Click **Save**.
5. Click **Deploy**. The first build installs the packages and takes 2–5
   minutes. You can watch the progress log.
6. When it finishes, the app opens at `https://<your-name>.streamlit.app`.
   That link is what you share.

### Important: never add `OPENAI_API_KEY` to Secrets

The app reads `OPENAI_API_KEY` from the server if one is set, and uses it
whenever a visitor leaves the key box empty. On a public link that means
**anyone could run questions on your account**. Keep Secrets empty for this
setup. For the same reason, do not add an `OPENAI_API_KEY` secret in the
GitHub repository's settings (it would also make the weekly "Evals" workflow
spend money).

---

## Step 3: Check it works

1. Open your app link.
2. In the sidebar, paste your OpenAI API key.
3. Upload a small PDF or `.txt` file and ask a question about it, e.g.
   "What is the main conclusion of this document?"
4. You should see the steps stream in, followed by an answer with sources.

---

## Updating the app later

Change the files on GitHub (edit in the browser, upload new versions, or
`git push`). Streamlit Cloud notices the change and redeploys automatically
within a minute or two.

## Good to know

- **The app sleeps when unused.** After a period with no visitors the free
  plan puts the app to sleep. The next visitor sees a button such as
  "Yes, get this app back up!", and it wakes in under a minute.
- **Memory resets on restart.** Uploaded files and past answers live only in
  the visitor's session. A restart or a page refresh clears them; nothing is
  stored long-term.
- **Upload limit:** 50 MB per file, set in `.streamlit/config.toml`.
- **Logs:** on your app page, click **Manage app** (bottom right) to see the
  server log. Copy any red error text from there when asking for help.
- **Reboot:** **Manage app** → **⋮** → **Reboot app**.

---

## Troubleshooting

| What you see | Likely cause | Fix |
|---|---|---|
| Build log: `ModuleNotFoundError` or "No such file app.py" | Files were uploaded inside an extra folder, so `requirements.txt` / `app.py` are not at the top level | Re-upload so `app.py` sits at the repository root, or set "Main file path" to the subfolder path (e.g. `ragverify-main/app.py`) |
| Build log: errors installing `numpy` / `tiktoken` | Unusual Python version selected | Delete the app and redeploy with Python 3.12 |
| "Add your OpenAI API key in the sidebar." | No key entered | Paste a key in the sidebar |
| `LLM call failed ... 401` / "Incorrect API key" | Key mistyped or revoked | Create a new key at platform.openai.com |
| `LLM call failed ... 429` / "insufficient_quota" | The OpenAI account has no credit, or hit its rate limit | Add credit in OpenAI billing, or wait a minute |
| `model_not_found` or "does not have access to model" | That OpenAI account cannot use the selected model | Pick another model in the sidebar, e.g. `gpt-4o-mini` |
| Web-only questions keep abstaining or say no web results | Free public search services often block cloud servers | Upload documents instead. Advanced: run your own SearxNG and add a secret `RAGVERIFY_SEARX_ENDPOINTS = "https://your-searx/search"` |
| "Stopped by a budget cap" | The question hit the per-run spend or 180-second time limit | Raise "Max spend per run" in the sidebar, or turn off Entailment / Rerank / Multi-hop |
| App shows "Oh no." or crashes on large uploads | Out of memory on the free plan (~1 GB) | Use smaller or fewer files; reboot the app |
| "Rerank: cross-encoder" behaves like "llm" | The cross-encoder package is not installed (too large for free hosting) | Expected; it falls back automatically |
| Red ❌ "CI" check on GitHub | The automated tests failed on GitHub | Open the **Actions** tab, read the failing step; the app itself may still deploy fine |

## Alternative free hosts

- **Render** (<https://render.com>): New → Web Service → connect the repo.
  Build command `pip install -r requirements.txt`; start command
  `streamlit run app.py --server.port $PORT --server.address 0.0.0.0`.
  Free instances sleep after 15 minutes and take about a minute to wake.
- **Hugging Face Spaces**: needs a Docker-based Space with a `Dockerfile`.
