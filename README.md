# 💸 Expense Tracker MCP API V3

A full-stack personal and group expense tracking system built using **FastAPI**, **PostgreSQL**, **SQLAlchemy**, and the **Model Context Protocol (MCP)**.

Supports:

* Personal expense tracking
* Group expense sharing
* Debt simplification
* OAuth 2.0 Authentication with PKCE
* Invite-code-based group joining
* Role-based access control
* Audit logging
* MCP integration for AI agents

---

# 🚀 Features

## Personal Expenses

* Create, update, delete personal expenses
* Filter by category, payment method, month, year, recurring status
* Expense summaries and analytics
* Category-wise breakdown
* Payment method breakdown

## Group Expenses

### Group Types

* Family
* Friends
* Trip
* Office
* Roommates
* Event
* Other

### Split Types

* Equal
* Percentage
* Fixed Amount
* By Days Stayed
* Custom

### Group Features

* Member balance tracking
* Settlement management
* Debt simplification
* Expense history
* Member management

---

## Authentication

### OAuth 2.0 + PKCE

* Authorization Code Flow
* PKCE (S256 & Plain)
* Dynamic Client Registration
* Access Tokens
* Secure Password Hashing

---

## MCP Integration

### HTTP MCP Server

* SSE Transport
* POST Transport

### Stdio MCP Server

Compatible with:

* Claude Desktop
* MCP Clients
* Local AI Agents

---

# 🏗️ Architecture

```text
┌─────────────────────────────────────────────┐
│                 Client Layer                │
│                                             │
│ Browser UI      Claude / MCP Clients        │
└───────────────┬───────────────┬─────────────┘
                │               │
                ▼               ▼
┌─────────────────────────────────────────────┐
│           FastAPI Backend Server            │
│                                             │
│ OAuth 2.0                                  │
│ REST API                                   │
│ MCP HTTP Server                            │
│ Bootstrap Frontend                         │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│         SQLAlchemy + PostgreSQL             │
└─────────────────────────────────────────────┘
```

---

# 📁 Project Structure

```text
mcp_expense_tracker/
│
├── backend.py
├── frontend.py
├── db.py
├── mcp_server.py
├── requirements.txt
└── README.md
```

---

# ⚙️ Prerequisites

* Python 3.11+
* PostgreSQL 14+
* Git

---

# 🔧 Installation

## Clone Repository

```bash
git clone <repository-url>
cd mcp_expense_tracker
```

## Create Virtual Environment

### Windows

```cmd
python -m venv .venv
.venv\Scripts\activate
```

### Linux / macOS

```bash
python -m venv .venv
source .venv/bin/activate
```

## Install Dependencies

```bash
pip install -r requirements.txt
```

Or:

```bash
pip install fastapi uvicorn sqlalchemy psycopg2-binary httpx python-multipart pydantic mcp
```

## Create Database

```sql
CREATE DATABASE expense_tracker;
```

## Configure Database

Update `db.py`:

```python
DATABASE_URL = "postgresql://postgres:12345@localhost:5432/expense_tracker"
```

## Add PostgreSQL Enum Values

```sql
ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_added';
ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_reactivated';
ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_deactivated';
ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'group_unarchived';
```

---

# ▶️ Running the Application

## FastAPI Server

```bash
uvicorn backend:app --host 0.0.0.0 --port 8001 --reload
```

Application URL:

```text
http://localhost:8001
```

## MCP Stdio Server

```bash
python mcp_server.py
```

Using a specific user:

### Windows

```cmd
set EXPENSE_USER_ID=2
python mcp_server.py
```

### Linux / macOS

```bash
EXPENSE_USER_ID=2 python mcp_server.py
```

---

# 🌐 Web UI Routes

| Page         | Route              |
| ------------ | ------------------ |
| Login        | /app/login         |
| Dashboard    | /app               |
| Expenses     | /app/expenses      |
| Add Expense  | /app/add           |
| Groups       | /app/groups        |
| Create Group | /app/groups/create |
| Join Group   | /app/groups/join   |
| Profile      | /app/profile       |

---

# 🔐 OAuth Flow

1. Client registers via `/register`
2. User authorizes via `/oauth/authorize`
3. Authorization code is generated
4. Client exchanges code at `/oauth/token`
5. Access token is issued
6. Authenticated API access begins

Discovery Endpoints:

```text
/.well-known/oauth-authorization-server
/.well-known/oauth-protected-resource
```

---

# 💰 Debt Simplification

Example:

* Alice paid ₹300
* Bob paid ₹150
* Carol paid ₹0

Balances:

```text
Alice +200
Bob +100
Carol -300
```

Settlements:

```text
Carol → Alice ₹200
Carol → Bob ₹100
```

Only two transactions are required.

---

# 📜 Audit Log

Recorded actions include:

* Group Created
* Group Archived
* Group Unarchived
* Member Added
* Member Removed
* Member Reactivated
* Member Deactivated
* Member Promoted
* Member Demoted
* Ownership Transferred
* Join Approved
* Join Rejected
* Invite Code Regenerated

---

# 🌍 Environment Variables

| Variable        | Default               |
| --------------- | --------------------- |
| SELF_URL        | http://localhost:8001 |
| EXPENSE_USER_ID | 1                     |

---

# ⚠️ Known Limitations

### REST Authentication

Current REST endpoints use a temporary hardcoded user ID:

```python
caller_id = 1
```

Replace with token-based authentication in production.

### Other Limitations

* No refresh tokens
* No email notifications
* Single-server session storage
* Windows may show harmless WinError 10054 logs

---

# 📄 License

This project is intended for educational and development purposes.

---

**Built with FastAPI, PostgreSQL, SQLAlchemy, MCP, and OAuth 2.0 🚀**
