# 🪝 Git's pre-commit & commit-msg Hooks  
### From Zero to Your First Hook — The Essential Starting Point  

[WebPage Tutorial](https://medium.com/jungletronics/gits-pre-commit-commit-msg-hooks-9d541bb6dffd)

This project is a hands-on introduction to **Git Hooks** — the automated scripts that help keep your repository clean and consistent. 

You'll learn how to build two essential ones:  
- **pre-commit hook** — blocks commits containing secrets or unwanted data.  
- **commit-msg hook** — enforces commit message patterns (like automatic prefixes).
- **[Semantic Versioning](https://semver.org/)**

---

## 🚀 Getting Started  

### 1. Clone this repository  
```bash
git clone git@github.com:giljr/git_s_hooks_samples.git
cd git_s_hooks_samples
code .
```

### 2. Now create two files:

 a) **secrets.json**:
```bash
touch config/secrets.json
```
Inside paste:
```json
{
  "aws_access_key_id": "AKJJHGBDBasadaaSLS",
  "aws_secret_access_key": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" 
}
```

 b) **encrypt_secrets.rb**:

```bash
touch encrypt_secrets.rb
```
Inside paste:
```ruby
#!/usr/bin/env ruby
require "json"
require "lockbox"

# --- Configuration ---
KEY = "c0a4b3ff1e4e1a22b89b0b93783d4ab9ff9f47b739b2d9a9df70e33a0b7f3a3e" # replace with your own
FILE = "./config/secrets.enc"

lockbox = Lockbox.new(key: KEY)

def usage
  puts "Usage:"
  puts "  ruby encrypt_secrets.rb --encrypt   # Encrypt config/secrets.json -> secrets.enc"
  puts "  ruby encrypt_secrets.rb --decrypt   # Decrypt config/secrets.enc -> print values"
  exit
end

mode = ARGV[0]
usage unless ["--encrypt", "--decrypt"].include?(mode)

if mode == "--encrypt"
  unless File.exist?("./config/secrets.json")
    puts "❌ secrets.json not found. Create one first, e.g.:"
    puts '{ "api_key": "abc123", "password": "supersecret" }'
    exit
  end

  data = File.read("./config/secrets.json")
  encrypted = lockbox.encrypt(data)
  File.write(FILE, encrypted)
  puts "✅ Encrypted and saved to #{FILE}"

elsif mode == "--decrypt"
  unless File.exist?(FILE)
    puts "❌ #{FILE} not found."
    exit
  end

  decrypted = lockbox.decrypt(File.read(FILE))
  secrets = JSON.parse(decrypted)
  puts "🔑 Decrypted secrets:"
  secrets.each { |k, v| puts "  #{k}: #{v}" }
end
```
### 3. Then Two Hooks:

  a) **pre-commit**:

On Terminal:
```bash
vim .github/hooks/pre-commit
```
Inside, paste:
```bash
#!/bin/sh

echo "Hello hooks!"

# --- Check for TODOs in staged files ---
if git diff --cached --name-only | xargs grep -Hn 'TODO' 2>/dev/null; then
  echo "⚠️ Commit warning: TODO comments found in the staged changes."
  # no exit here - just a warning
fi

# --- Check for hardcoded secrets in staged content only ---
if git diff --cached | grep -I -n -i -E "(key|secret|token|password)\s*[:=]\s*['\"]?[A-Za-z0-9/+=._-]+"; then
  echo "❌ Commit aborted: Hardcoded secret detected!"
  exit 1
fi

exit 0
```

On Terminal type, to quit: 
```bash
:q
```



 b)  **commit-msg**:

```
vim .git/hooks/commit-msg
```

Inside paste:
```bash
#!/usr/bin/env bash
# Simple commit message check for beginners

msg_file="$1"
msg=$(cat "$msg_file")

# Check if message is empty
if [[ -z "$msg" ]]; then
  echo "❌ Commit message cannot be empty."
  exit 1
fi

# Check if message is too short
if [[ ${#msg} -lt 10 ]]; then
  echo "⚠️ Commit message too short. Please describe your change briefly."
  exit 1
fi

# Enforce starting with a type like fix:, feat:, etc.
if ! [[ "$msg" =~ ^(feat|fix|docs|chore|test|refactor|style|perf): ]]; then
  echo "❌ Commit message must start with a valid type (feat:, fix:, ci:, docs:, chore:, test:, etc.)"
  echo "Example: feat: add user login form"
  exit 1
fi

# Optional: encourage a simple format like "type: short description"
# if ! [[ "$msg" =~ ^[a-zA-Z]+: ]]; then
#   echo "💡 Tip: Start your message with a type, e.g.: fix:, feat:, docs:, chore:, test:"
#   echo "Example: feat: add user login form"
# fi

exit 0
```

On Terminal type, to quit: 
```bash
:q
```

### 4. Make them executable
```bash
chmod +x .git/hooks/pre-commit
chmod +x .git/hooks/commit-msg
```


### 5. Explore the Hooks - Test it out

Try committing a file with **hardcoded credentials** or an **invalid message**.

Watch the hook prevent it before it ever reaches your repo!

Git hooks live inside the `.git/hooks/` directory and run automatically when triggered by specific Git events.

a) **pre-commit** — runs before every commit

Edit `main.rb` and intentionally add some credentials. Then try committing:
```bash
git add main.rb
git commit -m "committing secrets :/"

```
🚫 The hook should block this commit.

b) **commit-msg** — runs after the commit message is entered

Now try a simple commit:
```bash
git add main.rb
git commit -m "simple commit"
```
🙅‍♂️ This one will be rejected because your message doesn’t include a valid prefix.
Use a valid Semantic Commit type (like `feat:`, `fix:`, or `test:`) instead:
```bash
git commit -m "test: simple commit"
```
✅ This commit will pass successfully!


### 6.SemVer

In **Semantic Versioning** ( [SemVer](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://semver.org/&ved=2ahUKEwjo07X-ieuQAxVnq5UCHZ4YMxoQFnoECAwQAQ&usg=AOvVaw2wqeU7SPQk7aq7nuXGCrz-) ), only commits that change the public API (meaning features or breaking changes) affect the version numbers:

`MAJOR` → incremented when backward-incompatible changes are made.

`MINOR` → incremented when new functionality is added in a backward-compatible manner.

`PATCH` → incremented when backward-compatible bug fixes are made.

Prefixes like:
`ci:`, `docs:`, `test:`, `chore:`, `env:`, `build:`, `refac:` (or `refactor:`)
are all **non-functional changes** — they don’t alter the runtime behavior or API surface of your code.

So by SemVer convention, these commit types do not change the version number at all.
They’re typically grouped under "no version bump" or "metadata-only" updates in automated release workflows (like when using semantic-release or standard-version).

✅ only these **change SemVer**:

`fix:` → Patch

`feat:` → Minor

`BREAKING CHANGE:` (or `!` after type, e.g. feat!:) → Major


### 7. 🧩 Tech Stack

    Bash scripting

    Git CLI

    ggshield (soon in future post)


### 8. 🧠 Learn More

For detailed tutorials, examples, and explanations, visit:

### 9. Jungletronics on Medium:

[🪝 Git's pre-commit & commit-msg Hooks](https://medium.com/jungletronics/gits-pre-commit-commit-msg-hooks-9d541bb6dffd) - 
From Zero to Your First Hook - The Essential Starting Point


### 10. 📜 License

MIT © 2025 **[Jungletronics](https://medium.com/jungletronics/gits-pre-commit-commit-msg-hooks-9d541bb6dffd)**

