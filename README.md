# ahub-lint-config

A centralized repository for Automation Hub lint configurations, ensuring consistent commit message conventions and linting rules across multiple projects and CI systems. Lint configs are published as a public npm package so that they can be easily used in both CI pipelines and local development setups.

---

## Conventional Commits Linter

**Goal**: Enforce [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) to keep commit messages organized and meaningful.

- **Based on**: [@commitlint/config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional)  
- **Rules**: See the [commitlint reference rules](https://commitlint.js.org/reference/rules.html)  
- **Custom Rules**: Our additional or overriding rules can be found here in the [commitlint.config.js](https://github.com/br-automation-com/ahub-lint-config/blob/main/commitlint.config.js). For instance, we may enforce max body length or other custom constraints.

### Why Conventional Commits?

1. **Standardized** commit messages improve readability and automation.  
2. **Automatic** versioning and changelog generation become simpler.  
3. **Consistent** commit messages help the entire team quickly understand the nature of changes.

---

## Usage

### 1. GitHub Actions

You can fail your PR or push build if a commit doesn’t follow the conventional commit specification. Here’s a sample job that uses [@ahub-public/commitlint-config](https://www.npmjs.com/package/@ahub-public/commitlint-config) to verify commit messages:

```yaml
commitlint:
  runs-on: ubuntu-latest
  steps:
    - name: Check out code
      uses: actions/checkout@v3
      with:
        fetch-depth: 0

    - name: Install Node.js
      uses: actions/setup-node@v4
      with:
        node-version: 20

    - name: Install commitlint & public ahub lint config
      run: |
        npm install commitlint@latest @commitlint/config-conventional @ahub-public/commitlint-config
        # Dynamically create a .commitlintrc.js file that extends your config
        echo "module.exports = { extends: ['@ahub-public/commitlint-config'] };" > .commitlintrc.js

    - name: Validate current commit (last commit) with commitlint
      if: github.event_name == 'push'
      run: npx commitlint --last --verbose

    - name: Validate PR commits with commitlint
      if: github.event_name == 'pull_request'
      run: npx commitlint --from ${{ github.event.pull_request.base.sha }} --to ${{ github.event.pull_request.head.sha }} --verbose
```

#### What’s happening:

We install Commitlint, the conventional config, and our ahub config from npm.
We generate a temporary .commitlintrc.js extending @ahub-public/commitlint-config.
Depending on whether it’s a push or a pull request, we lint the appropriate commits.

### 2. Local Usage (Pre-Commit Hook with Husky)

To catch invalid commit messages **before** pushing to GitHub, you can install Husky to manage local Git hooks.

#### Install Husky and commitlint in your project:
```bash
npm install -D husky commitlint @commitlint/config-conventional @ahub-public/commitlint-config
```
#### Add a local .commitlintrc.js referencing the ahub config:

```js
// .commitlintrc.js
module.exports = {
  extends: ['@ahub-public/commitlint-config']
};
```
#### Create a commit-msg hook:

```bash
npx husky add .husky/commit-msg "npx commitlint --edit $1"
git add .husky/commit-msg
```
Now, whenever you run git commit, Husky calls Commitlint to validate the message. If it doesn’t pass, it rejects the commit locally.

## Additional Info

- **VS Code Extension**: [Conventional Commits by vivaxy](https://marketplace.visualstudio.com/items?itemName=vivaxy.vscode-conventional-commits) helps you craft valid commit messages directly in VS Code.

**Benefits**:
- Automated release notes and versioning  
- Streamlined collaboration and clearer commit history  

**Happy Linting!**  
Help your team produce consistent commit messages for cleaner repos and smoother release workflows. Feel free to submit issues or pull requests to this repository if you have ideas or improvements.
