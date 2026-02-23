# Git Ignore & npm Install Explained

---

## 📄 .gitignore

### What is it?
A `.gitignore` file tells Git **which files or folders to ignore** — meaning they won't be tracked, staged, or committed to your repository.

### Why use it?
Some files don't belong in version control, such as:
- Dependencies (`node_modules/`)
- Environment variables (`.env`)
- Build outputs (`dist/`, `build/`)
- OS-generated files (`.DS_Store`, `Thumbs.db`)
- Log files (`*.log`)

### Syntax

```gitignore
# This is a comment

# Ignore a specific file
secret.env

# Ignore all .log files
*.log

# Ignore an entire folder
node_modules/

# Ignore a folder only at the root level
/build

# Ignore all files in any "temp" folder
**/temp/

# Negate a rule (do NOT ignore this file)
!important.log
```

### Example `.gitignore` for a Node.js project

```gitignore
# Dependencies
node_modules/

# Environment variables
.env
.env.local

# Build output
dist/
build/

# Logs
*.log
npm-debug.log*

# OS files
.DS_Store
Thumbs.db
```

> **Tip:** GitHub provides ready-made `.gitignore` templates for different languages and frameworks at [gitignore.io](https://www.toptal.com/developers/gitignore).

---

## 📦 npm install

### What is it?
`npm install` (or `npm i`) is a command from the **Node Package Manager (npm)** that downloads and installs JavaScript packages/dependencies into your project.

### How it works
It reads your `package.json` file to find out which packages your project needs, then downloads them into a `node_modules/` folder.

### Common Usage

```bash
# Install all dependencies listed in package.json
npm install

# Install a specific package and save it as a dependency
npm install express

# Install a package as a dev dependency (only needed during development)
npm install --save-dev nodemon

# Install a package globally on your system
npm install -g typescript

# Install a specific version of a package
npm install lodash@4.17.21

# Install packages from package-lock.json exactly (for reproducible builds)
npm ci
```

### Key Files

| File | Purpose |
|------|---------|
| `package.json` | Lists project metadata and required dependencies |
| `package-lock.json` | Locks exact versions of every installed package |
| `node_modules/` | The folder where packages are actually stored |

### `dependencies` vs `devDependencies`

```json
{
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

- **`dependencies`** — packages needed to run your app in production
- **`devDependencies`** — packages only needed during development (testing, building, etc.)

### Why `node_modules/` is in `.gitignore`
The `node_modules/` folder can contain **thousands of files** and is often hundreds of megabytes. Since `package.json` and `package-lock.json` already record everything needed, anyone can recreate `node_modules/` by running `npm install`. There's no reason to commit it.

---

## 🔄 The Typical Workflow

```bash
# 1. Clone a repository
git clone https://github.com/user/project.git

# 2. Navigate into the project
cd project

# 3. Install dependencies (node_modules is NOT in the repo — it's gitignored)
npm install

# 4. Start developing!
npm start
```
