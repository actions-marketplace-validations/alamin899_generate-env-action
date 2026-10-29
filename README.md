# Generate Environment File Action ⚙️

**Automatically generate `.env` files from GitHub Secrets and Variables during your CI/CD workflow.**

Stop manually creating or updating environment files during deployment.

This GitHub Action collects your repository/environment **GitHub Secrets** and **GitHub Variables**, converts them into environment variables, and can automatically generate files such as:

```text
.env
.env.production
.env.staging
.env.local
```

It can also export those values to `$GITHUB_ENV`, making them immediately available to all subsequent GitHub Actions steps.

## 🔄 How It Works

```text
GitHub Secrets
      +
GitHub Variables
      ↓
generate-env-action
      ↓
┌──────────────────────────────┐
│ Generate .env file           │
│ Export to $GITHUB_ENV        │
└──────────────────────────────┘
      ↓
Docker / Server / Kubernetes / Application
```

For example, suppose your GitHub repository contains:

```text
Secrets:
DB_PASSWORD=super-secret-password
API_KEY=secret-api-key

Variables:
APP_ENV=production
APP_URL=https://example.com
DB_HOST=mysql
```

The action can automatically generate:

```env
APP_ENV=production
APP_URL=https://example.com
DB_HOST=mysql
DB_PASSWORD=super-secret-password
API_KEY=secret-api-key
```

No need to manually maintain the production `.env` values inside your repository.

---

## ✨ Features

### 📄 Automatic `.env` File Generation

Generate one or multiple environment files automatically from GitHub Secrets and Variables.

```yaml
generate_env_files: '.env'
```

Or:

```yaml
generate_env_files: '.env, .env.local, .env.production'
```

### 🚀 Export Variables to GitHub Actions Environment

By default, variables are also exported to `$GITHUB_ENV`.

That means subsequent workflow steps can directly use them:

```bash
echo "$APP_ENV"
echo "$APP_URL"
```

### 🧩 `.env.example` Friendly

Use your existing `.env.example` as the application's environment structure while populating values dynamically during CI/CD.

This is useful when you want your application to maintain a predictable environment configuration without committing real credentials.

### 🔒 Exclude Sensitive CI/CD Credentials

Not every GitHub Secret belongs in your application `.env`.

For example, you may have:

```text
DOCKER_PASSWORD
SSH_PRIVATE_KEY
GITHUB_TOKEN
```

Exclude them easily:

```yaml
exclude_keys: 'DOCKER_PASSWORD,SSH_PRIVATE_KEY,GITHUB_TOKEN'
```

### 🐳 Perfect for Automated Deployments

Useful for:

- Docker builds and deployments
- Docker Compose
- Laravel deployments
- Node.js applications
- Python applications
- Kubernetes deployments
- Automated server deployments
- Staging and production environments
- CI/CD testing environments

---

# 🚀 Quick Start

## 1. Add Your GitHub Secrets and Variables

Go to your GitHub repository:

```text
Settings
→ Secrets and variables
→ Actions
```

Add your sensitive values under **Secrets** and normal configuration values under **Variables**.

For example:

```text
Secrets
├── DB_PASSWORD
├── REDIS_PASSWORD
└── API_KEY

Variables
├── APP_ENV
├── APP_URL
├── DB_HOST
└── DB_DATABASE
```

---

## 2. Generate `.env` Automatically

Add the action to your GitHub Actions workflow:

```yaml
steps:
  - name: Generate application environment
    uses: alamin899/generate-env-action@v1
    with:
      secrets_context: ${{ toJson(secrets) }}
      vars_context: ${{ toJson(vars) }}
      generate_env_files: '.env'
```

During the workflow, the action automatically generates:

```text
.env
```

using your GitHub Secrets and Variables.

---

# 📚 Usage Examples

## 1. Export Variables Only — No `.env` File

If your workflow only needs environment variables and your application does not require a physical `.env` file:

```yaml
steps:
  - name: Load environment variables
    uses: alamin899/generate-env-action@v1
    with:
      secrets_context: ${{ toJson(secrets) }}
      vars_context: ${{ toJson(vars) }}

  - name: Use environment variables
    run: |
      echo "Environment: $APP_ENV"
      echo "Application URL: $APP_URL"
```

By default:

```text
GitHub Secrets + Variables
          ↓
      $GITHUB_ENV
          ↓
Subsequent workflow steps
```

No physical `.env` file is generated.

---

## 2. Automatically Generate `.env`

If your application requires a physical `.env` file:

```yaml
steps:
  - name: Generate .env
    uses: alamin899/generate-env-action@v1
    with:
      secrets_context: ${{ toJson(secrets) }}
      vars_context: ${{ toJson(vars) }}
      generate_env_files: '.env'

  - name: Deploy application
    run: |
      docker compose up -d
```

The workflow automatically creates:

```text
.env
```

before the deployment command runs.

---

## 3. Generate Multiple Environment Files

You can generate multiple files at once:

```yaml
steps:
  - name: Generate environment files
    uses: alamin899/generate-env-action@v1
    with:
      secrets_context: ${{ toJson(secrets) }}
      vars_context: ${{ toJson(vars) }}
      generate_env_files: '.env, .env.local, .env.production'
```

This creates:

```text
.env
.env.local
.env.production
```

---

## 4. Generate `.env` but Don't Export to `$GITHUB_ENV`

If you only need the physical file:

```yaml
steps:
  - name: Generate .env
    uses: alamin899/generate-env-action@v1
    with:
      secrets_context: ${{ toJson(secrets) }}
      vars_context: ${{ toJson(vars) }}
      generate_env_files: '.env'
      is_set_process_env: 'false'
```

Result:

```text
GitHub Secrets + Variables
          ↓
         .env
```

The variables will not be exported to `$GITHUB_ENV`.

---

## 5. Exclude Deployment Credentials

Your repository may contain CI/CD secrets that should not be added to the application's `.env`.

```yaml
steps:
  - name: Generate safe application .env
    uses: alamin899/generate-env-action@v1
    with:
      secrets_context: ${{ toJson(secrets) }}
      vars_context: ${{ toJson(vars) }}
      generate_env_files: '.env'
      exclude_keys: 'DOCKER_USERNAME,DOCKER_PASSWORD,SSH_PRIVATE_KEY,GITHUB_TOKEN'
```

Only application-related values will be written to the generated environment.

---

# ⚙️ Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `secrets_context` | **Yes** | — | GitHub Secrets as JSON. Use `${{ toJson(secrets) }}`. |
| `vars_context` | **Yes** | — | GitHub Variables as JSON. Use `${{ toJson(vars) }}`. |
| `generate_env_files` | No | `''` | Comma-separated files to generate, such as `.env, .env.production`. Empty means no physical file is generated. |
| `is_set_process_env` | No | `'true'` | Export variables to `$GITHUB_ENV` so subsequent workflow steps can use them. |
| `exclude_keys` | No | `''` | Comma-separated keys that should not be written to files or exported to `$GITHUB_ENV`. |

---

# 🔐 Security Recommendation

Keep sensitive credentials in **GitHub Secrets**, not directly inside your workflow or repository.

For example:

```text
✅ GitHub Secret → DB_PASSWORD
✅ GitHub Secret → API_KEY
✅ GitHub Secret → REDIS_PASSWORD

❌ Hardcoded password inside workflow
❌ Production .env committed to Git
❌ API keys committed to repository
```

You can also use `exclude_keys` to prevent CI/CD-specific credentials from being included in the application's generated environment file.

---

# 💡 Common Deployment Flow

A typical production deployment can look like this:

```text
Developer pushes code
        ↓
GitHub Actions starts
        ↓
Checkout repository
        ↓
GitHub Secrets + Variables
        ↓
generate-env-action
        ↓
Automatically generate .env
        ↓
Build / Deploy application
        ↓
Docker / Server / Kubernetes
```

This keeps environment configuration centralized in GitHub while eliminating the need to manually maintain deployment `.env` values.

---

## 📝 License

This project is licensed under the MIT License.
