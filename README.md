
### 1. Setup VM:
```bash
sudo apt update && sudo apt install -y git openssh-client
```
```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
```
```bash
sudo tee /usr/local/bin/add-repo >/dev/null <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

r="${1:?usage: add-repo repo-name}"

mkdir -p ~/.ssh && chmod 700 ~/.ssh
touch ~/.ssh/config && chmod 600 ~/.ssh/config

if [ -e ~/.ssh/deploy_$r ]; then
  echo "Error: key for '$r' already exists at ~/.ssh/deploy_$r" >&2
  exit 1
fi

# Shared settings block, created only once
if ! grep -q '^Host github-\*' ~/.ssh/config; then
  printf 'Host github-*\n  HostName github.com\n  User git\n  IdentitiesOnly yes\n  StrictHostKeyChecking accept-new\n\n' >> ~/.ssh/config
fi

ssh-keygen -t ed25519 -C "github-$r" -f ~/.ssh/deploy_$r -N ""
printf '\nHost github-%s\n  IdentityFile ~/.ssh/deploy_%s\n' "$r" "$r" >> ~/.ssh/config

echo
echo "Done. Register this public key as a deploy key in the '$r' repository"
echo "(Settings > Deploy keys > Add deploy key, leave write access unchecked):"
echo
cat ~/.ssh/deploy_$r.pub
echo
echo "Clone with: git clone git@github-$r:c-codespace/$r.git /opt/$r"
EOF
sudo chmod +x /usr/local/bin/add-repo
```

```bash
chmod 600 ~/.ssh/config
```

### Testing
```bash
add-repo TESTING
ssh -T git@github-TESTING
```
