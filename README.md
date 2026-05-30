💸 Expense Tracker — MCP API V3
A full-stack personal and group expense tracking system built with FastAPI, SQLAlchemy, PostgreSQL, and the Model Context Protocol (MCP). Supports OAuth 2.0 with PKCE, group management with role-based access control, invite-code-based join flows, debt simplification, and an audit log.

Table of Contents

Features
Architecture
Project Structure
Prerequisites
Setup
Running the Server
Using the Web UI
MCP Tools Reference
REST API Reference
OAuth 2.0 Flow
Group Roles & Permissions
Invite Code & Join Flow
Debt Simplification
Audit Log
Environment Variables
Known Limitations


Features
Personal Expenses

Create, read, update, delete personal expenses
Filter by category, payment method, month, year, recurring status
Summary dashboard: total spend, breakdown by category and payment method

Group Expenses

Create groups with types: family, friends, trip, office, roommates, event, other
Add group expenses with 5 split modes: equal, percentage, fixed, by_days, custom
Member balance tracking and net balance computation
Debt simplification algorithm (minimises number of transactions to settle)
Settlement records with mark-as-settled support

Group Management (V3)

Role-based access: Owner, Admin, Member
Invite-code-based private group discovery (groups are not publicly listed)
Join request workflow: request → approve/reject (admin/owner only)
Bulk approve/reject join requests
Promote/demote members, transfer ownership
Soft deactivate and reactivate members
Archive and unarchive groups
Full audit log of every admin action

Authentication

OAuth 2.0 Authorization Code flow with PKCE (S256 or plain)
Dynamic client registration (/register)
Token-based access with 24-hour expiry
Secure password hashing (SHA-256 + salt)

MCP Integration

Full MCP tool server over HTTP (SSE + POST) for use with Claude and other MCP clients
Separate stdio MCP server (mcp_server.py) for local CLI use with Claude Desktop


Architecture
┌─────────────────────────────────────────────────────┐
│                    Client Layer                      │
│  Browser (Bootstrap UI)  │  Claude (MCP Client)     │
└──────────────┬───────────┴──────────┬───────────────┘
               │                      │
               ▼                      ▼
┌─────────────────────────────────────────────────────┐
│               FastAPI Application (backend.py)       │
│                                                      │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐  │
│  │  OAuth 2.0  │  │  MCP Server  │  │  REST API  │  │
│  │  /oauth/*   │  │  POST /      │  │  /groups/* │  │
│  │  /register  │  │  GET  / SSE  │  │  /expenses │  │
│  └─────────────┘  └──────────────┘  └────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │            Frontend (frontend.py)            │    │
│  │            Bootstrap 5 Web UI                │    │
│  │            mounted at /app/*                 │    │
│  └──────────────────────────────────────────────┘    │
└───────────────────────┬─────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│              SQLAlchemy ORM (db.py)                  │
│                  PostgreSQL                          │
└─────────────────────────────────────────────────────┘

Project Structure
mcp_expense_tracker/
├── backend.py          # Main FastAPI app: OAuth, MCP HTTP server, REST API
├── frontend.py         # Bootstrap 5 web UI (mounted onto FastAPI at /app/*)
├── db.py               # SQLAlchemy models, enums, DB init
├── mcp_server.py       # Standalone stdio MCP server (for Claude Desktop)
├── requirements.txt    # Python dependencies
└── README.md

Prerequisites

Python 3.11+
PostgreSQL 14+ running locally
A database named expense_tracker


Setup
1. Clone and create virtual environment
bashgit clone <your-repo-url>
cd mcp_expense_tracker
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
2. Install dependencies
bashpip install fastapi uvicorn sqlalchemy psycopg2-binary httpx python-multipart pydantic mcp
Or if you have a requirements.txt:
bashpip install -r requirements.txt
3. Create the PostgreSQL database
sqlCREATE DATABASE expense_tracker;
4. Configure the database URL
In db.py, update the connection string if needed:
pythonDATABASE_URL = "postgresql://postgres:12345@localhost:5432/expense_tracker"
5. Add missing PostgreSQL enum values (run once in pgAdmin Query Tool)
sqlALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_added';
ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_reactivated';
ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'member_deactivated';
ALTER TYPE auditactiontype ADD VALUE IF NOT EXISTS 'group_unarchived';
The tables are created automatically on first startup via init_db().

Running the Server
bashuvicorn backend:app --port 8001 --reload
The server starts at http://localhost:8001.
Running the stdio MCP server (Claude Desktop)
bashpython mcp_server.py
Set EXPENSE_USER_ID environment variable to control which user the stdio server acts as (defaults to user ID 1):
bashEXPENSE_USER_ID=2 python mcp_server.py

Using the Web UI
URLDescriptionhttp://localhost:8001/app/loginSign inhttp://localhost:8001/appPersonal expense dashboardhttp://localhost:8001/app/expensesList / filter expenseshttp://localhost:8001/app/addAdd a personal expensehttp://localhost:8001/app/groupsMy groupshttp://localhost:8001/app/groups/createCreate a grouphttp://localhost:8001/app/groups/joinJoin a group via invite codehttp://localhost:8001/app/groups/{id}Group dashboardhttp://localhost:8001/app/groups/{id}/expensesGroup expense listhttp://localhost:8001/app/groups/{id}/settleSettle uphttp://localhost:8001/app/groups/{id}/membersMember managementhttp://localhost:8001/app/groups/{id}/rolesRole management (admin/owner)http://localhost:8001/app/groups/{id}/requestsJoin requests (admin/owner)http://localhost:8001/app/groups/{id}/inviteInvite code (admin/owner)http://localhost:8001/app/groups/{id}/auditAudit log (admin/owner)http://localhost:8001/app/profileUser profile
Creating your first account

Go to http://localhost:8001/app/login
Click "Create one" to register
Fill in name, email, and password (min 8 characters)
You are automatically signed in after registration


MCP Tools Reference
Personal Expense Tools
ToolDescriptionRequired Argscreate_expenseCreate a personal expenseamount, category, datelist_expensesList expenses with filters—get_expenseGet expense by IDexpense_idupdate_expenseUpdate expense fieldsexpense_iddelete_expenseDelete an expenseexpense_idget_summaryTotal + breakdown by category/payment—get_profileGet current user profile—
Group Tools
ToolDescriptionRequired Argscreate_groupCreate a group (caller becomes owner)nameadd_group_memberAdd a user (admin/owner)group_id, user_idremove_group_memberRemove a member (admin/owner)group_id, user_idadd_group_expenseAdd expense with splitgroup_id, title, amount, paid_by, split_type, dateget_group_summaryDashboard: balances, statsgroup_idlist_group_expensesList group expenses with filtersgroup_idcalculate_group_settlementCompute + persist simplified debtsgroup_idsimplify_debtsPreview simplified debts (no persist)group_idget_member_balanceNet balance for one membergroup_id, user_idarchive_groupArchive group (owner only)group_idunarchive_groupUnarchive group (owner only)group_idlist_my_groupsAll groups caller belongs to—
V3 Invite & Join Tools
ToolDescriptionRequired Argsget_invite_codeGet/generate invite code (admin/owner)group_idregenerate_invite_codeRotate invite code (admin/owner)group_idrequest_group_joinSubmit join request via invite codeinvite_codeapprove_join_requestApprove a join request (admin/owner)group_id, join_request_idreject_join_requestReject a join request (admin/owner)group_id, join_request_idlist_join_requestsList join requests (admin/owner)group_id
V3 Role & Membership Tools
ToolDescriptionRequired Argschange_member_rolePromote/demote member (owner only)group_id, user_id, new_roletransfer_ownershipTransfer ownership (owner only)group_id, new_owner_user_iddeactivate_memberSoft-remove a member (admin/owner)group_id, user_idreactivate_memberRe-enable a member (admin/owner)group_id, user_idget_group_audit_logView audit log (member+)group_id
Split Types
TypeDescriptionRequired InputequalSplit evenly among participantsparticipant_user_ids: [int]percentageSplit by percentage (must sum to 100)participants: [{user_id, share_value}]fixedFixed INR amount per person (must sum to total)participants: [{user_id, share_value}]by_daysSplit proportional to days stayedparticipants: [{user_id, share_value}]customArbitrary INR amount per personparticipants: [{user_id, share_value}]

REST API Reference
Users
MethodPathDescriptionPOST/usersCreate user
Personal Expenses
MethodPathDescriptionPOST/expensesCreate expenseGET/expensesList expensesGET/expenses/{id}Get expensePUT/expenses/{id}Update expenseDELETE/expenses/{id}Delete expenseGET/summaryExpense summary
Groups
MethodPathDescriptionPOST/groupsCreate groupGET/groupsList my groupsGET/groups/{id}Get groupGET/groups/{id}/membersList membersPOST/groups/{id}/membersAdd memberDELETE/groups/{id}/members/{uid}Remove memberPOST/groups/{id}/expensesAdd group expenseGET/groups/{id}/expensesList group expensesGET/groups/{id}/expenses/{eid}Get group expenseGET/groups/{id}/dashboardGroup dashboardPOST/groups/{id}/settlements/calculateCalculate settlementsGET/groups/{id}/settlementsList settlementsPATCH/groups/{id}/settlements/{sid}/settleMark settledPOST/groups/{id}/archiveArchive groupPOST/groups/{id}/unarchiveUnarchive groupGET/my/groupsMy groups (private)
V3 — Invite & Join
MethodPathDescriptionGET/groups/{id}/invite-codeGet invite codePOST/groups/{id}/invite-code/regenerateRegenerate codeGET/groups/join/{code}Look up group by codePOST/groups/joinSubmit join requestGET/groups/{id}/join-requestsList join requestsPOST/groups/{id}/join-requests/{jid}/approveApprove requestPOST/groups/{id}/join-requests/{jid}/rejectReject request
V3 — Roles & Membership
MethodPathDescriptionPATCH/groups/{id}/members/{uid}/roleChange member rolePOST/groups/{id}/transfer-ownershipTransfer ownershipPATCH/groups/{id}/members/{uid}/deactivateDeactivate memberPATCH/groups/{id}/members/{uid}/reactivateReactivate memberGET/groups/{id}/audit-logGet audit log

OAuth 2.0 Flow
The server implements OAuth 2.0 Authorization Code with PKCE, compatible with Claude's MCP OAuth requirements.
1. Client registers at POST /register → gets client_id + client_secret
2. Client redirects user to GET /oauth/authorize with PKCE challenge
3. User logs in (or registers) at the sign-in page
4. Server redirects back to redirect_uri with ?code=...
5. Client exchanges code at POST /oauth/token → gets access_token
6. Client uses Bearer token in Authorization header for all API calls
Discovery endpoints:

GET /.well-known/oauth-authorization-server
GET /.well-known/oauth-protected-resource


Group Roles & Permissions
PermissionOwnerAdminMemberAdd expenses✅✅✅View expenses & balances✅✅✅View audit log✅✅✅Approve / reject join requests✅✅❌View & regenerate invite code✅✅❌Add / remove members✅✅❌Promote members to admin✅✅❌Demote admins to member✅❌❌Transfer ownership✅❌❌Archive / unarchive group✅❌❌

Invite Code & Join Flow
Groups are private by default — they do not appear in any public listing. The only way to discover a group is via an invite code.
Owner/Admin                          New User
     │                                   │
     │  Get invite code                  │
     │  (GET /app/groups/{id}/invite)    │
     │                                   │
     │  Share 8-char code out-of-band    │
     │ ─────────────────────────────────▶│
     │                                   │ Enter code at /app/groups/join
     │                                   │ Preview group info
     │                                   │ Click "Request to Join"
     │                                   │
     │◀─────── Join request created ─────│
     │                                   │
     │  Review at /app/groups/{id}/requests
     │  Click Approve / Reject           │
     │                                   │
     │──── If approved: member added ───▶│

Invite codes are 8 characters, uppercase alphanumeric
Codes can be regenerated at any time (old code immediately invalidated)
Archived groups do not accept new join requests
A user cannot have two pending requests to the same group


Debt Simplification
The settlement algorithm minimises the number of transactions needed to clear all debts in a group.
Example:
Alice paid ₹300 for dinner (3 people equal split → ₹100 each)
Bob paid ₹150 for taxi (3 people equal split → ₹50 each)

Net balances:
  Alice: +200  (paid 300, owes 100)
  Bob:   +100  (paid 150, owes 50)
  Carol: -300  (paid 0, owes 150)

Simplified: Carol pays Alice ₹200, Carol pays Bob ₹100
→ 2 transactions instead of potentially more
Use simplify_debts to preview, then calculate_group_settlement to persist.

Audit Log
Every admin/owner action in a group is recorded:
ActionTriggergroup_createdGroup is createdgroup_archivedGroup is archivedgroup_unarchivedGroup is unarchivedmember_addedMember directly addedmember_removedMember removedmember_reactivatedDeactivated member re-enabledmember_deactivatedMember soft-removedmember_promotedMember → Adminmember_demotedAdmin → Memberownership_transferredOwner changedjoin_requestedJoin request submittedjoin_approvedJoin request approvedjoin_rejectedJoin request rejectedinvite_code_regeneratedInvite code rotated
Audit log is append-only and visible to all group members.

Environment Variables
VariableDefaultDescriptionSELF_URLhttp://localhost:8001Base URL for OAuth redirects (set in frontend.py)EXPENSE_USER_ID1Default user ID for stdio MCP server

Known Limitations

REST stubs use user_id=1 — The REST endpoints (not MCP) use a hardcoded caller_id = 1 stub instead of token-based auth. The MCP endpoints are fully authenticated. In production, replace with proper token extraction on all REST routes.
No refresh tokens — Access tokens expire after 24 hours; the user must re-authenticate.
No email notifications — Join request approvals/rejections are not notified by email.
Single-server only — No distributed session support; _VERIFIERS dict is in-process memory.
Windows ConnectionResetError — The WinError 10054 log entries are a known Windows/asyncio quirk with Chrome DevTools and do not affect functionality.
