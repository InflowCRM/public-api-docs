# n8n Community Node: Inflow CRM

The [n8n-nodes-inflow-crm](https://www.npmjs.com/package/n8n-nodes-inflow-crm) package is an official community node that enables seamless integration of [Inflow CRM](https://inflowcrm.pl) with your n8n workflows.

## Features

- Create, read, update, and delete CRM records (CRUD)
- Real-time webhook triggers for CRM events
- Dynamic field mapping and custom field support
- File upload and attachment handling
- Advanced filtering, sorting, and pagination
- Multi-module support (contacts, companies, deals, etc.)

## Installation

For detailed instructions, see the [official n8n documentation](https://docs.n8n.io/integrations/community-nodes/installation/).

**n8n Admin Panel (Recommended)**
1. Open n8n, go to **Settings → Community Nodes**
2. Click **Install a community node**
3. Enter: `n8n-nodes-inflow-crm`
4. Click **Install**

---

## Node Actions

The Inflow CRM node provides the following main operations:

### 📝 Create Record
- Create new records in any CRM module
- Dynamic field mapping based on selected module
- File upload support for attachments and documents
- Automatic required field validation

### ✏️ Update Record
- Update existing records by ID
- Only specify fields you want to change
- Add or replace attachments
- Partial updates supported

### 🔍 Search Records
- Find multiple records with advanced filtering
- Dynamic fields, pagination, and sorting

### 🔎 Find One Record
- Retrieve a single record matching criteria
- Flexible filtering without requiring ID

### 🗑️ Delete Record
- Delete records by ID
- Permanent deletion with confirmation

### 🔢 Count Records
- Count records matching filter criteria
- Efficient for large datasets

---

## Webhook Triggers

The Inflow CRM Trigger node responds to real-time CRM events:

### 🆕 Record Created
- Triggered when new records are created in a selected module
- Complete record data in webhook payload

### 📝 Record Updated
- Triggered when records are modified in a selected module
- Updated record data with changes

### 🗑️ Record Deleted
- Triggered when records are deleted from a selected module
- Information about deleted record

---

## Dynamic Field Support

- Automatic field discovery based on your CRM configuration
- Type-safe mapping (text, number, date, boolean, etc.)
- Module-specific and custom field support

---

## Known Issues & Roadmap

- Adding potentials (leads) with pipeline/stage assignments may require manual steps
- Improved support for complex field relationships and multiselect fields is planned



## More Information

- [n8n-nodes-inflow-crm on npm](https://www.npmjs.com/package/n8n-nodes-inflow-crm)
