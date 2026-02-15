# Postgres Automation with Ansible & Docker
This project demonstrates how to orchestrate a PostgreSQL instance using Docker Compose and automate user/role management (Owner, Writer, Reader) using Ansible.

## 🚀 Getting Started
### 1. Prerequisites

Ensure you have Docker and Python installed. It is recommended to use a virtual environment:

```Bash
python3 -m venv .venv
source .venv/bin/activate
```
### 2. Install Dependencies

Install the required Python libraries and Ansible collections:

```Bash
pip install -r requirements.txt
ansible-galaxy collection install community.postgresql
```

## 🔐 Security Configuration (Local Only)
To keep this project secure, sensitive credentials are not stored in the repository.

### A. Environment Variables (`.env`)

Create a file named `.env` in the root directory. This file is ignored by Git.

```Bash
# .env
DB_ADMIN_USER=admin
DB_ADMIN_PASS=your_secure_password
DB_NAME=app_db
```
### B. Ansible Vault (`vars/vault.yml`)

Ansible Vault encrypts your user passwords.

1. Create the vault file:

```Bash
ansible-vault create vars/vault.yml
```
2. Enter a vault password when prompted.

3. Add the following content to the file:

```YAML
vault_alice_pass: "alice_password_here"
vault_bob_pass: "bob_password_here"
vault_charlie_pass: "charlie_password_here"
```

### C. Editing Existing Secrets

If you need to change the passwords for `Alice`, `Bob`, or `Charlie` later, use the edit command:

```Bash
ansible-vault edit vars/vault.yml
```

This will prompt for your Vault password and open the decrypted file in your default terminal editor (usually Vim or Nano). Once you save and exit, Ansible re-encrypts the file automatically.

## 🛠 Usage
### Step 1: Start the Database

Bring up the PostgreSQL container:

```Bash
docker-compose up -d
```

### Step 2: Load Environment Variables

Load your `.env` variables into your current terminal session so Ansible can access them:


```Bash
export $(grep -v '^#' .env | xargs)
```
### Step 3: Run the Playbook

Run the configuration. You will be prompted for the Vault Password:

```Bash
ansible-playbook setup_postgres.yml --ask-vault-pass
```

## 🏗 Project Architecture
Docker: Handles the lifecycle of the DB engine.

Ansible: Connects via `localhost:5432` to create:

Group Roles: `db_owner`, `db_writer`, `db_reader`.

Users: `alice` (Owner), `bob` (Writer), `charlie` (Reader).

Permissions: Granular access control applied to the public schema.

## 🔍 Verification
You can verify the permissions by logging in as the different users:

```Bash
# Test as Charlie (Reader) - Should only be able to SELECT
psql -h localhost -U charlie -d app_db

# Test as Bob (Writer) - Should be able to INSERT/UPDATE
psql -h localhost -U bob -d app_db
```