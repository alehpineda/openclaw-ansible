# Development Mode Installation

This guide explains how to install OpenClaw in **development mode**, where the application is built from source instead of installed from npm.

## Overview

### Release Mode vs Development Mode

| Feature | Release Mode | Development Mode |
|---------|-------------|------------------|
| Source | npm registry | GitHub repository |
| Installation | `npm install -g openclaw@latest` | `git clone` + `npm run build` |
| Location | `~/.local/lib/node_modules/openclaw/...` | `~/code/openclaw/` |
| Binary | Global npm package | Symlink to `bin/openclaw.js` |
| Updates | `npm install -g openclaw@latest` | `git pull` + `npm run build` |
| Use Case | Production, stable deployments | Development, testing, debugging |
| Recommended For | End users | Developers, contributors |

## Installation

### Quick Install

```bash
# Clone the ansible installer
git clone https://github.com/pasogott/openclaw-ansible.git
cd openclaw-ansible

# Run in development mode
./run-playbook.sh -e openclaw_install_mode=development
```

### Manual Install

```bash
# Install ansible
sudo apt update && sudo apt install -y ansible git

# Clone repository
git clone https://github.com/pasogott/openclaw-ansible.git
cd openclaw-ansible

# Install collections
ansible-galaxy collection install -r requirements.yml

# Run playbook with development mode
ansible-playbook playbook.yml --ask-become-pass -e openclaw_install_mode=development
```

## What Gets Installed

### Directory Structure

```
/home/openclaw/
├── .openclaw/              # Configuration directory
│   ├── sessions/
│   ├── credentials/
│   ├── data/
│   └── logs/
├── .local/
│   └── bin/
│       └── openclaw       # Symlink -> ~/code/openclaw/bin/openclaw.js
└── code/
    └── openclaw/          # Git repository
        ├── bin/
        │   └── openclaw.js
        ├── dist/          # Built files
        ├── src/           # Source code
        └── package.json
```

### Installation Steps

The Ansible playbook performs these steps:

1. **Create `~/code` directory**
   ```bash
   mkdir -p ~/code
   ```

2. **Clone repository**
   ```bash
   cd ~/code
   git clone https://github.com/openclaw/openclaw.git
   ```

3. **Install dependencies**
   ```bash
   cd openclaw
   npm install
   ```

4. **Build from source**
   ```bash
   npm run build
   ```

5. **Create symlink**
   ```bash
   ln -sf ~/code/openclaw/bin/openclaw.js ~/.local/bin/openclaw
   chmod +x ~/code/openclaw/bin/openclaw.js
   ```

6. **Add development aliases** to `.bashrc`:
   ```bash
   alias openclaw-rebuild='cd ~/code/openclaw && npm run build'
   alias openclaw-dev='cd ~/code/openclaw'
   alias openclaw-pull='cd ~/code/openclaw && git pull && npm install && npm run build'
   ```

## Development Workflow

### Making Changes

```bash
# 1. Navigate to repository
openclaw-dev
# or: cd ~/code/openclaw

# 2. Make your changes
vim src/some-file.ts

# 3. Rebuild
openclaw-rebuild
# or: npm run build

# 4. Test immediately
openclaw --version
openclaw doctor
```

### Pulling Updates

```bash
# Pull latest changes and rebuild
openclaw-pull

# Or manually:
cd ~/code/openclaw
git pull
npm install
npm run build
```

### Testing Changes

```bash
# After rebuilding, the openclaw command uses the new code immediately
openclaw status
openclaw gateway

# View daemon logs
openclaw logs
```

### Switching Branches

```bash
cd ~/code/openclaw

# Switch to feature branch
git checkout feature-branch
npm install
npm run build

# Switch back to main
git checkout main
npm install
npm run build
```

## Development Aliases

The following aliases are added to `.bashrc`:

| Alias | Command | Purpose |
|-------|---------|---------|
| `openclaw-dev` | `cd ~/code/openclaw` | Navigate to repo |
| `openclaw-rebuild` | `cd ~/code/openclaw && npm run build` | Rebuild after changes |
| `openclaw-pull` | `cd ~/code/openclaw && git pull && npm install && npm run build` | Update and rebuild |

Plus an environment variable:

```bash
export OPENCLAW_DEV_DIR="$HOME/code/openclaw"
```

## Configuration Variables

You can customize the development installation:

```yaml
# In playbook or command line
openclaw_install_mode: "development"
openclaw_repo_url: "https://github.com/openclaw/openclaw.git"
openclaw_repo_branch: "main"
openclaw_code_dir: "/home/openclaw/code"
openclaw_repo_dir: "/home/openclaw/code/openclaw"
```

### Using a Fork

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e openclaw_install_mode=development \
  -e openclaw_repo_url=https://github.com/YOUR_USERNAME/openclaw.git \
  -e openclaw_repo_branch=your-feature-branch
```

### Custom Location

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e openclaw_install_mode=development \
  -e openclaw_code_dir=/home/openclaw/projects
```

## Switching Between Modes

### From Release to Development

```bash
# Uninstall global package
npm uninstall -g openclaw

# Run ansible in development mode
ansible-playbook playbook.yml --ask-become-pass -e openclaw_install_mode=development
```

### From Development to Release

```bash
# Remove symlink
rm ~/.local/bin/openclaw

# Remove repository (optional)
rm -rf ~/code/openclaw

# Install from npm
npm install -g openclaw@latest
```

## Troubleshooting

### Build Fails

```bash
cd ~/code/openclaw

# Check Node.js version (needs 22.x)
node --version

# Clean install
rm -rf node_modules
npm install
npm run build
```

### Symlink Not Working

```bash
# Check symlink
ls -la ~/.local/bin/openclaw

# Recreate symlink
rm ~/.local/bin/openclaw
ln -sf ~/code/openclaw/bin/openclaw.js ~/.local/bin/openclaw
chmod +x ~/code/openclaw/bin/openclaw.js
```

### Command Not Found

```bash
# Ensure ~/.local/bin is in PATH
echo $PATH | grep -q ".local/bin" || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Git Issues

```bash
cd ~/code/openclaw

# Reset to clean state
git reset --hard origin/main
git clean -fdx

# Rebuild
npm install
npm run build
```

## Performance Considerations

### Build Time

First build takes longer (~1-2 minutes depending on system):
```bash
npm install    # Downloads dependencies
npm run build  # Compiles TypeScript
```

Subsequent rebuilds are faster (~10-30 seconds):
```bash
npm run build  # Only recompiles changed files
```

### Disk Usage

Development mode uses more disk space:

- **Release mode**: ~150 MB (npm global cache)
- **Development mode**: ~400 MB (repo + node_modules + dist)

### Memory Usage

No difference in runtime memory usage between modes.

## CI/CD Integration

### Testing Before Merge

```bash
# Test specific commit
cd ~/code/openclaw
git fetch origin pull/123/head:pr-123
git checkout pr-123
npm install
npm run build

# Test it
openclaw doctor
```

### Automated Testing

```bash
#!/bin/bash
# test-openclaw.sh

cd ~/code/openclaw
git pull
npm install
npm run build

# Run tests
npm test

# Integration test
openclaw doctor
```

## Best Practices

### Development Workflow

1. ✅ **Always rebuild after code changes**
   ```bash
   openclaw-rebuild
   ```

2. ✅ **Test changes before committing**
   ```bash
   npm run build && openclaw doctor
   ```

3. ✅ **Keep dependencies updated**
   ```bash
   npm update
   npm run build
   ```

4. ✅ **Use feature branches**
   ```bash
   git checkout -b feature/my-feature
   ```

### Don't Do

- ❌ Editing code without rebuilding
- ❌ Running `npm link` manually (breaks setup)
- ❌ Installing global packages while in dev mode
- ❌ Modifying symlink manually

## Advanced Usage

### Multiple Repositories

You can have multiple clones:

```bash
# Main development
~/code/openclaw/          # main branch

# Experimental features
~/code/openclaw-test/     # testing branch

# Switch binary symlink
ln -sf ~/code/openclaw-test/bin/openclaw.js ~/.local/bin/openclaw
```

### Custom Build Options

```bash
cd ~/code/openclaw

# Development build (faster, includes source maps)
NODE_ENV=development npm run build

# Production build (optimized)
NODE_ENV=production npm run build
```

### Debugging

```bash
# Run with debug output
DEBUG=* openclaw gateway

# Or specific namespaces
DEBUG=openclaw:* openclaw gateway
```

## See Also

- [Main README](../README.md)
- [Security Architecture](security.md)
- [Troubleshooting Guide](troubleshooting.md)
- [OpenClaw Repository](https://github.com/openclaw/openclaw)
