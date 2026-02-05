# Academic Management Platform - System Design

## System Architecture Overview

The platform follows a modern web architecture with clear separation of concerns:
- **Frontend**: Next.js with server-side rendering and static generation
- **Backend**: Supabase providing database, authentication, and storage
- **Automation**: n8n for workflow orchestration and integrations
- **AI Services**: Google Gemini via backend-only API calls with caching

## High-Level Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Next.js App   │    │   Supabase       │    │      n8n        │
│   (Frontend)    │◄──►│   (Backend)      │◄──►│  (Automation)   │
│                 │    │                  │    │                 │
│ • Admin Portal  │    │ • PostgreSQL     │    │ • Google APIs   │
│ • Faculty Portal│    │ • Auth + RLS     │    │ • Notifications │
│ • Student Portal│    │ • Storage        │    │ • Workflows     │
│ • Shared UI     │    │ • Edge Functions │    │ • Integrations  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         │              ┌────────▼────────┐             │
         │              │  Google Gemini  │             │
         └──────────────►│   (AI Service)  │◄────────────┘
                        │                 │
                        │ • Content Gen   │
                        │ • Q&A Engine    │
                        │ • Summarization │
                        └─────────────────┘
```

## Frontend Architecture

### Next.js App Structure
```
src/
├── app/                    # App Router pages
│   ├── (auth)/            # Authentication routes
│   ├── admin/             # Admin portal
│   ├── faculty/           # Faculty portal
│   ├── student/           # Student portal
│   └── api/               # API routes (minimal)
├── components/            # Reusable UI components
│   ├── ui/               # ShadCN components
│   ├── forms/            # Form components
│   └── layouts/          # Layout components
├── lib/                  # Utilities and configurations
│   ├── supabase/         # Supabase client
│   ├── auth/             # Authentication helpers
│   └── utils/            # General utilities
└── hooks/                # Custom React hooks
```

### Key Frontend Patterns
- **Server Components**: Default for data fetching and static content
- **Client Components**: Interactive elements and real-time features
- **Route Protection**: Middleware-based authentication checks
- **Role-Based Rendering**: Conditional UI based on user roles
- **Optimistic Updates**: Immediate UI feedback with server reconciliation

## Backend Architecture

### Supabase Configuration
```
Database (PostgreSQL)
├── Authentication
│   ├── Users table (extended with profiles)
│   └── Role-based access policies
├── Core Tables
│   ├── profiles (user metadata)
│   ├── institutions
│   ├── departments
│   ├── classes
│   ├── assignments
│   ├── submissions
│   └── resources
├── Storage Buckets
│   ├── documents (course materials)
│   ├── assignments (student submissions)
│   └── media (images, videos)
└── Edge Functions
    ├── ai-integration
    ├── file-processing
    └── notification-triggers
```

### Row-Level Security (RLS) Policies
```sql
-- Example RLS policies
CREATE POLICY "Admin full access" ON profiles
  FOR ALL USING (auth.jwt() ->> 'role' = 'admin');

CREATE POLICY "Faculty own data" ON assignments
  FOR ALL USING (faculty_id = auth.uid());

CREATE POLICY "Students view assigned" ON assignments
  FOR SELECT USING (
    class_id IN (
      SELECT class_id FROM student_classes 
      WHERE student_id = auth.uid()
    )
  );
```

## Database Schema

### Core Tables

```sql
-- User Profiles (extends Supabase auth.users)
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  role user_role NOT NULL,
  full_name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  institution_id UUID REFERENCES institutions(id),
  department_id UUID REFERENCES departments(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Institutions
CREATE TABLE institutions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  code TEXT UNIQUE NOT NULL,
  settings JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Departments/Branches
CREATE TABLE departments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  institution_id UUID REFERENCES institutions(id),
  name TEXT NOT NULL,
  code TEXT NOT NULL,
  head_faculty_id UUID REFERENCES profiles(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Classes
CREATE TABLE classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  department_id UUID REFERENCES departments(id),
  name TEXT NOT NULL,
  code TEXT NOT NULL,
  faculty_id UUID REFERENCES profiles(id),
  academic_year TEXT NOT NULL,
  semester INTEGER NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Student-Class Relationships
CREATE TABLE student_classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES profiles(id),
  class_id UUID REFERENCES classes(id),
  enrolled_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(student_id, class_id)
);

-- Assignments
CREATE TABLE assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id UUID REFERENCES classes(id),
  faculty_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  due_date TIMESTAMPTZ,
  max_score INTEGER DEFAULT 100,
  file_attachments TEXT[],
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Assignment Submissions
CREATE TABLE submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  assignment_id UUID REFERENCES assignments(id),
  student_id UUID REFERENCES profiles(id),
  content TEXT,
  file_attachments TEXT[],
  score INTEGER,
  feedback TEXT,
  submitted_at TIMESTAMPTZ DEFAULT NOW(),
  graded_at TIMESTAMPTZ,
  UNIQUE(assignment_id, student_id)
);

-- Resources/Documents
CREATE TABLE resources (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id UUID REFERENCES classes(id),
  faculty_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  file_path TEXT NOT NULL,
  file_type TEXT NOT NULL,
  file_size BIGINT,
  is_public BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- AI Cache
CREATE TABLE ai_cache (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  cache_key TEXT UNIQUE NOT NULL,
  prompt_hash TEXT NOT NULL,
  response JSONB NOT NULL,
  model_version TEXT NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Enums and Types
```sql
CREATE TYPE user_role AS ENUM ('admin', 'faculty', 'student');
CREATE TYPE assignment_status AS ENUM ('draft', 'published', 'closed');
CREATE TYPE submission_status AS ENUM ('pending', 'submitted', 'graded');
```

## Authentication & Authorization Flow

### Authentication Process
```
1. User visits protected route
2. Next.js middleware checks Supabase session
3. If no session → redirect to login
4. If session exists → verify role and permissions
5. Route access granted based on role
```

### Authorization Matrix
| Resource | Admin | Faculty | Student |
|----------|-------|---------|---------|
| User Management | Full | None | None |
| Class Management | Full | Own Classes | View Enrolled |
| Assignments | Full | Own Classes | View/Submit |
| Resources | Full | Own Classes | View Shared |
| AI Tools | Limited | Full | Limited |

### RLS Implementation
- **Database Level**: All queries filtered by user role and ownership
- **API Level**: Additional validation in Edge Functions
- **Frontend Level**: UI elements conditionally rendered by role

## n8n Workflow Responsibilities

### Core Workflows
```
Google Integration Workflows:
├── Drive File Sync
│   ├── Monitor faculty uploads
│   ├── Sync to Google Drive
│   └── Update file permissions
├── Calendar Integration
│   ├── Create meeting events
│   ├── Send calendar invites
│   └── Sync assignment deadlines
├── Forms Integration
│   ├── Generate quiz forms
│   ├── Collect responses
│   └── Auto-grade submissions
└── Email Notifications
    ├── Assignment reminders
    ├── Grade notifications
    └── System announcements
```

### Workflow Triggers
- **Database Changes**: Supabase webhooks trigger workflows
- **Scheduled Tasks**: Daily/weekly maintenance and notifications
- **API Events**: External service integrations
- **Manual Triggers**: Admin-initiated bulk operations

### Example Workflow: Assignment Notification
```
Trigger: New assignment created in Supabase
↓
1. Fetch enrolled students for class
2. Create Google Calendar events
3. Send email notifications via Gmail API
4. Update notification log in database
5. Schedule reminder notifications
```

## AI Integration Design

### AI Service Architecture
```
Frontend Request → Supabase Edge Function → Cache Check → Gemini API
                                      ↓
                  Response ← Cache Store ← AI Response Processing
```

### AI Features Implementation

#### Document Chat
```typescript
// Edge Function: ai-document-chat
export async function POST(request: Request) {
  const { documentId, question, userId } = await request.json();
  
  // Check cache first
  const cacheKey = `doc-chat:${documentId}:${hashQuestion(question)}`;
  const cached = await getCachedResponse(cacheKey);
  if (cached) return cached;
  
  // Fetch document content
  const document = await getDocument(documentId);
  
  // Call Gemini API
  const response = await callGeminiAPI({
    context: document.content,
    question: question,
    model: 'gemini-pro'
  });
  
  // Cache response
  await cacheResponse(cacheKey, response, '24h');
  
  return response;
}
```

#### Quiz Generation
```typescript
// Edge Function: ai-quiz-generator
export async function POST(request: Request) {
  const { content, questionCount, difficulty } = await request.json();
  
  const prompt = `Generate ${questionCount} ${difficulty} questions from: ${content}`;
  
  const response = await callGeminiAPI({
    prompt,
    model: 'gemini-pro',
    temperature: 0.7
  });
  
  // Parse and validate quiz format
  const quiz = parseQuizResponse(response);
  
  return quiz;
}
```

### AI Rate Limiting
- **Students**: 10 AI requests/hour
- **Faculty**: 50 AI requests/hour  
- **Admin**: 100 AI requests/hour
- **Caching**: 24-hour cache for similar requests

## Error Handling Strategy

### Frontend Error Handling
```typescript
// Global error boundary
export function GlobalErrorBoundary({ children }) {
  return (
    <ErrorBoundary
      fallback={<ErrorFallback />}
      onError={(error, errorInfo) => {
        logError(error, errorInfo);
        showToast('Something went wrong. Please try again.');
      }}
    >
      {children}
    </ErrorBoundary>
  );
}

// API error handling
export async function apiCall(endpoint, options) {
  try {
    const response = await fetch(endpoint, options);
    if (!response.ok) throw new Error(response.statusText);
    return await response.json();
  } catch (error) {
    handleApiError(error);
    throw error;
  }
}
```

### Backend Error Handling
```sql
-- Database constraints and triggers
CREATE OR REPLACE FUNCTION handle_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
EXCEPTION
  WHEN OTHERS THEN
    RAISE LOG 'Error in handle_updated_at: %', SQLERRM;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

### Error Categories
- **User Errors**: Validation failures, permission denied
- **System Errors**: Database failures, API timeouts
- **Integration Errors**: Third-party service failures
- **AI Errors**: Model failures, rate limit exceeded

## Caching Strategy

### Multi-Level Caching
```
Browser Cache (Static Assets)
↓
CDN Cache (Supabase Storage)
↓
Application Cache (AI Responses)
↓
Database Query Cache (PostgreSQL)
```

### Cache Implementation
```typescript
// AI Response Caching
interface CacheEntry {
  key: string;
  data: any;
  expiresAt: Date;
  tags: string[];
}

export class AICache {
  async get(key: string): Promise<any> {
    const entry = await supabase
      .from('ai_cache')
      .select('*')
      .eq('cache_key', key)
      .gt('expires_at', new Date().toISOString())
      .single();
    
    return entry?.data?.response;
  }
  
  async set(key: string, data: any, ttl: string): Promise<void> {
    const expiresAt = new Date(Date.now() + parseTTL(ttl));
    
    await supabase.from('ai_cache').upsert({
      cache_key: key,
      response: data,
      expires_at: expiresAt.toISOString(),
      model_version: 'gemini-pro-v1'
    });
  }
}
```

### Cache Invalidation
- **Time-based**: TTL for AI responses (24h), static content (1h)
- **Event-based**: Clear cache on data updates
- **Manual**: Admin cache management interface

## Deployment Architecture

### Production Environment
```
Vercel (Frontend)
├── Next.js App
├── Static Assets
├── Edge Functions (minimal)
└── Environment Variables

Supabase Cloud (Backend)
├── PostgreSQL Database
├── Authentication Service
├── Storage Buckets
├── Edge Functions
└── Real-time Subscriptions

n8n Cloud (Automation)
├── Workflow Engine
├── Google API Integrations
├── Scheduled Tasks
└── Webhook Endpoints
```

### Environment Configuration
```typescript
// Environment variables
const config = {
  supabase: {
    url: process.env.NEXT_PUBLIC_SUPABASE_URL,
    anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
    serviceKey: process.env.SUPABASE_SERVICE_ROLE_KEY, // Server-only
  },
  ai: {
    geminiApiKey: process.env.GEMINI_API_KEY, // Server-only
    projectId: process.env.GOOGLE_CLOUD_PROJECT_ID,
  },
  n8n: {
    webhookUrl: process.env.N8N_WEBHOOK_URL,
    apiKey: process.env.N8N_API_KEY,
  }
};
```

### Deployment Pipeline
```
1. Code Push → GitHub
2. Vercel Auto-Deploy (Frontend)
3. Supabase Migration (Database)
4. n8n Workflow Update (Automation)
5. Environment Variable Sync
6. Health Check Validation
```

## Future Scalability Plan

### Phase 1: Current Architecture (0-5K users)
- Single Supabase instance
- Vercel hobby/pro plan
- n8n starter plan
- Basic monitoring

### Phase 2: Optimization (5K-50K users)
- Database connection pooling
- CDN optimization
- Advanced caching layers
- Performance monitoring
- Load testing

### Phase 3: Scale-Out (50K+ users)
- Database read replicas
- Microservice extraction
- Advanced AI caching
- Multi-region deployment
- Enterprise monitoring

### Scaling Bottlenecks & Solutions
```
Database Connections
└── Solution: PgBouncer connection pooling

AI API Costs
└── Solution: Intelligent caching + rate limiting

File Storage
└── Solution: CDN + compression + cleanup jobs

Real-time Features
└── Solution: Supabase Realtime optimization

Search Performance
└── Solution: Full-text search + indexing
```

### Migration Considerations
- **Database**: PostgreSQL scaling patterns
- **Storage**: Multi-region file distribution
- **AI**: Model versioning and A/B testing
- **Monitoring**: Comprehensive observability stack
- **Security**: Advanced threat detection