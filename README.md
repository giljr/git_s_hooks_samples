
# 🪝 Git's pre-commit & commit-msg Hooks  
### From Zero to Your First Hook — The Essential Starting Point  

[Tutorial](https://medium.com/jungletronics/gits-pre-commit-commit-msg-hooks-9d541bb6dffd)

This project is a hands-on introduction to **Git Hooks** — the automated scripts that help keep your repository clean and consistent. 

You'll learn how to build two essential ones:  
- **pre-commit hook** — blocks commits containing secrets or unwanted data.  
- **commit-msg hook** — enforces commit message patterns (like automatic prefixes).  

---

## 🚀 Getting Started  

### 1. Clone this repository  
```bash
git clone git@github.com:giljr/git_s_hooks_samples.git
cd git_s_hooks_samples
```

2. Explore the hooks

Hooks are located inside .git/hooks/ — they run automatically when triggered by a Git event.

    pre-commit: runs before every commit

    commit-msg: runs after you write your commit message

3. Make them executable
```bash
chmod +x .git/hooks/pre-commit
chmod +x .git/hooks/commit-msg
```
4. Test it out

Try committing a file with hardcoded credentials or an invalid message.

Watch the hook prevent it before it ever reaches your repo!

🧠 Learn More

For detailed tutorials, examples, and explanations, visit:

👉 Jungletronics on Medium:

🪝 Git's pre-commit & commit-msg Hooks
From Zero to Your First Hook - The Essential Starting Point

🧩 Tech Stack

    Bash scripting

    Git CLI

    ggshield (soon in future post)

📜 License

MIT © 2025 Jungletronics
