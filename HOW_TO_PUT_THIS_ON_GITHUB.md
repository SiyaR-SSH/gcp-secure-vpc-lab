# Getting this repo onto GitHub

Two ways — pick one. Do the easy one now; the CLI one is worth learning soon because
"comfortable with Git" is itself a plus for cloud/security roles.

Before you start, edit two placeholders:
- In `LICENSE`, replace `[YOUR NAME]`.
- In `README.md`, the "What I learned" section is a draft — reword it in your own voice.

---

## Option 1 — Web upload (easiest, ~2 minutes)

1. Go to **github.com** → click **+** (top right) → **New repository**.
2. Repository name: `gcp-secure-vpc-lab`
3. Description: *Secure two-tier VPC network on GCP with least-privilege firewall rules.*
4. Set to **Public** (you want recruiters to see it).
5. **Do not** check "Add a README" (you already have one). Click **Create repository**.
6. On the empty repo page, click **"uploading an existing file"**.
7. Drag in `README.md`, `LICENSE`, `.gitignore`, and the `images` folder.
8. Scroll down, click **Commit changes**. Done.

Your CV link becomes: `github.com/<your-username>/gcp-secure-vpc-lab`

---

## Option 2 — Git command line (better for your portfolio)

First install Git (git-scm.com) and sign in. Then, from inside the `gcp-secure-vpc-lab`
folder:

```bash
git init
git add .
git commit -m "Phase 1: secure two-tier VPC network on GCP"
git branch -M main

# Create the empty repo on github.com first (steps 1-5 above, don't add a README),
# then copy its URL and run:
git remote add origin https://github.com/<your-username>/gcp-secure-vpc-lab.git
git push -u origin main
```

For subsequent phases you'll repeat `git add . && git commit -m "..." && git push`.

---

## After it's up — 3 quick polish steps

1. On the repo page, click the ⚙️ next to **About** and add topics: `gcp`, `cloud-security`,
   `networking`, `vpc`, `firewall`.
2. Go to your **profile** → pin this repo so it shows on your front page.
3. Make sure your screenshots actually render in the README (check the image paths match
   the filenames you uploaded).

That's a real, visible portfolio piece. Repeat the pattern for each phase.
