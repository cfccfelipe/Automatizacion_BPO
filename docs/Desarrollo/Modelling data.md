
## Step-by-Step: Data Modeling (Users, Chats, Agents)

### 1. Define Entity Requirements
Clarify what each entity must represent and store:

- **Users**: system actors (admin, client - last users-), sector adaptation.
- **Chats**: messages exchanged, sender.
- **Agents**: virtual agents with configuration, logic type (rules/LLM),.

---

### 2. Draft Entity Attributes

#### Users
- `id`: UUID or serial primary key
- `name`: string
- `email`: string (unique)
- `role`: enum (`admin`, `client`)
- `sector`: string
- `created_at`: timestamp

#### Agents

- `id`: UUID or SERIAL PRIMARY KEY  
- `name`: VARCHAR(100)  
- `type`: ENUM (`rule_based`, `llm`, `hybrid`)  
- `api_key`: TEXT (used for external model access if applicable)  
- `system_prompt`: TEXT (base prompt for agent behavior)  
- `active`: BOOLEAN DEFAULT TRUE  
- `created_at`: TIMESTAMP DEFAULT CURRENT_TIMESTAMP  


#### Chats
- `id`: UUID or serial
- `user_id`: foreign key → `users.id`
- `agent_id`: foreign key → `agents.id`
- `message`: text
- `sender`: enum (`user`, `agent`)
- `timestamp`: timestamp


---

### 3. Normalize Relationships
- One user can have many chats.
- One agent can handle many chats.
- Agents can be reused across sectors or customized per client.

---

### 4. Write SQL Schema

```sql
-- users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100) UNIQUE NOT NULL,
  role VARCHAR(20) CHECK (role IN ('admin', 'operator', 'client')),
  sector VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- agents table
CREATE TABLE agents (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  type VARCHAR(20) CHECK (type IN ('rule_based', 'llm', 'hybrid')),
  sector VARCHAR(50),
  config JSONB,
  active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- chats table
CREATE TABLE chats (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  agent_id INTEGER REFERENCES agents(id) ON DELETE SET NULL,
  message TEXT NOT NULL,
  sender VARCHAR(10) CHECK (sender IN ('user', 'agent')),
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  session_id VARCHAR(50)
);
