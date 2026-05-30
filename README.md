💸 Expense Tracker MCP API V3

A full-stack personal and group expense tracking system built using FastAPI, PostgreSQL, SQLAlchemy, and the Model Context Protocol (MCP).

The platform supports:

Personal expense management
Group expense sharing
Debt simplification
OAuth 2.0 Authentication with PKCE
Invite-code-based group joining
Role-based access control
Audit logging
MCP integration for AI agents (Claude, MCP Clients)
🚀 Features
Personal Expense Management
Create, update, delete expenses
Filter by:
Category
Payment method
Month/Year
Recurring status
Expense summaries
Category-wise analytics
Payment method breakdown
Group Expense Management
Supported Group Types
Family
Friends
Trip
Office
Roommates
Event
Other
Supported Split Types
Equal Split
Percentage Split
Fixed Amount Split
By Days Stayed
Custom Split
Group Features
Group balances
Net balance calculation
Settlement tracking
Debt simplification
Expense history
Member management
Role-Based Access Control
Roles
Role	Description
Owner	Full control
Admin	Manage members and requests
Member	Participate in expenses
Capabilities
Permission	Owner	Admin	Member
Add Expense	✅	✅	✅
View Expenses	✅	✅	✅
View Audit Log	✅	✅	✅
Approve Join Requests	✅	✅	❌
Manage Invite Codes	✅	✅	❌
Add/Remove Members	✅	✅	❌
Promote Members	✅	✅	❌
Demote Admins	✅	❌	❌
Transfer Ownership	✅	❌	❌
Archive Group	✅	❌	❌
Authentication
OAuth 2.0 Authorization Code Flow + PKCE

Features:

Dynamic Client Registration
PKCE (S256 and Plain)
Access Tokens
Secure Password Hashing
24-hour Token Expiry
MCP Integration

Supports:

HTTP MCP Server
SSE Transport
POST Transport
Stdio MCP Server

Compatible with:

Claude Desktop
MCP CLI Clients
Local AI Agent Workflows
🏗️ Architecture
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
📁 Project Structure
mcp_expense_tracker/
│
├── backend.py
│   ├── FastAPI Application
│   ├── OAuth Endpoints
│   ├── REST APIs
│   └── MCP HTTP Server
│
├── frontend.py
│   └── Bootstrap Web UI
│
├── db.py
│   └── SQLAlchemy Models
│
├── mcp_server.py
│   └── Stdio MCP Server
│
├── requirements.txt
│
└── README.md
⚙️ Prerequisites
Python 3.11+
PostgreSQL 14+
Git
🔧 Installation
1. Clone Repository
git clone <repository-url>
cd mcp_expense_tracker
2. Create Virtual Environment
Windows
python -m venv .venv

.venv\Scripts\activate
Linux / macOS
python -m venv .venv

source .venv/bin/activate
3. Install Dependencies
pip install -r requirements.txt

Or:

pip install \
fastapi \
uvicorn \
sqlalchemy \
psycopg2-binary \
httpx \
python-multipart \
pydantic \
mcp
4. Create PostgreSQL Database
CREATE DATABASE expense_tracker;
5. Configure Database

Update db.py:

DATABASE_URL = (
    "postgresql://postgres:12345@localhost:5432/expense_tracker"
)
6. Add Missing PostgreSQL Enum Values

Run once in pgAdmin:

ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_added';

ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_reactivated';

ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_deactivated';

ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'group_unarchived';
▶️ Running the Application
Start FastAPI Server
uvicorn backend:app --host 0.0.0.0 --port 8001 --reload

Application URL:

http://localhost:8001
Run MCP Stdio Server
python mcp_server.py

Specify user:

Windows
set EXPENSE_USER_ID=2

python mcp_server.py
Linux / macOS
EXPENSE_USER_ID=2 python mcp_server.py
🌐 Web UI Routes
Page	URL
Login	/app/login
Dashboard	/app
Expenses	/app/expenses
Add Expense	/app/add
Groups	/app/groups
Create Group	/app/groups/create
Join Group	/app/groups/join
Profile	/app/profile
🧰 MCP Tools
Personal Expense Tools
Tool
create_expense
list_expenses
get_expense
update_expense
delete_expense
get_summary
get_profile
Group Tools
Tool
create_group
add_group_member
remove_group_member
add_group_expense
list_group_expenses
get_group_summary
simplify_debts
calculate_group_settlement
get_member_balance
archive_group
unarchive_group
list_my_groups
Invite & Join Tools
Tool
get_invite_code
regenerate_invite_code
request_group_join
approve_join_request
reject_join_request
list_join_requests
Membership & Role Tools
Tool
change_member_role
transfer_ownership
deactivate_member
reactivate_member
get_group_audit_log
🔐 OAuth Flow
Client
   │
   ├─ POST /register
   │
   ▼
Receives Client Credentials
   │
   ├─ GET /oauth/authorize
   │
   ▼
User Login
   │
   ▼
Authorization Code
   │
   ├─ POST /oauth/token
   │
   ▼
Access Token
   │
   ▼
Authenticated API Calls

Discovery Endpoints:

/.well-known/oauth-authorization-server

/.well-known/oauth-protected-resource
🤝 Invite Code Workflow
Owner/Admin
      │
      │ Generate Invite Code
      ▼
 Share Code
      │
      ▼
 New User
      │
      │ Request Join
      ▼
Join Request Created
      │
      ▼
Admin Reviews Request
      │
 ┌────┴────┐
 │         │
Approve  Reject
 │
 ▼
User Added
💰 Debt Simplification

The system minimizes settlement transactions.

Example
Alice paid ₹300

Bob paid ₹150

Carol paid ₹0

Balances:

Alice +200

Bob +100

Carol -300

Simplified Settlements:

Carol → Alice ₹200

Carol → Bob ₹100

Only two transactions are required.

📜 Audit Log

Every administrative action is recorded:

Group Created
Group Archived
Group Unarchived
Member Added
Member Removed
Member Reactivated
Member Deactivated
Member Promoted
Member Demoted
Ownership Transferred
Join Request Approved
Join Request Rejected
Invite Code Regenerated
🌍 Environment Variables
Variable	Default
SELF_URL	http://localhost:8001
EXPENSE_USER_ID	1
⚠️ Known Limitations
REST Authentication

Current REST endpoints use:

caller_id = 1

Replace with token-based authentication for production use.

No Refresh Tokens

Users must re-authenticate after token expiry.

No Email Notifications

Join requests currently do not trigger email notifications.

Single Server Deployment

OAuth verifier storage is currently in-memory and not distributed.

Windows Asyncio Warning

WinError 10054 logs may appear on Windows and can be safely ignored.

📄 License

This project is intended for educational and development purposes.
