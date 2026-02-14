# SparkLink – Unified AI Campus Platform
## System Design Document

**Version**: 1.0  
**Last Updated**: February 14, 2026  
**Status**: Design Review  
**Document Owner**: Architecture Team

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Component-Level Design](#component-level-design)
4. [Data Flow Explanation](#data-flow-explanation)
5. [Database Design](#database-design)
6. [API Design Approach](#api-design-approach)
7. [Security Architecture](#security-architecture)
8. [Scalability Considerations](#scalability-considerations)
9. [Deployment Architecture](#deployment-architecture)
10. [Future Extensibility](#future-extensibility)

---

## System Overview

### Purpose

SparkLink is a cloud-native, unified AI campus platform designed to consolidate fragmented academic tools into a single intelligent ecosystem. The system provides role-based portals for Admins, Faculty, and Students with AI-powered assistance, automated workflows, and seamless Google Workspace integration.

### Design Principles

**Modularity**: Clear separation of concerns with independent, replaceable components  
**Security-First**: Zero-trust architecture with role-based access control at every layer  
**Serverless-Native**: Leverage managed services for automatic scaling and reduced operational overhead  
**Event-Driven**: Asynchronous processing for background tasks and integrations  
**API-First**: Well-defined interfaces enabling future extensibility  
**Performance**: Sub-3-second page loads with intelligent caching strategies  
**Cost-Optimized**: Free-tier friendly with clear upgrade paths

### Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Frontend | Next.js 14+ (App Router) | Server-side rendering, static generation, routing |
| UI Framework | Tailwind CSS + ShadCN UI | Responsive design, component library |
| Backend | Supabase | PostgreSQL, Authentication, Storage, Real-time |
| Database | PostgreSQL 15+ | Relational data with JSONB support |
| Authentication | Supabase Auth | JWT-based auth with RLS |
| Storage | Supabase Storage | Object storage with CDN |
| AI Services | Google Gemini (Vertex AI) | Content generation, Q&A, summarization |
| Automation | n8n | Workflow orchestration, integrations |
| Integrations | Google Workspace APIs | Drive, Meet, Calendar, Forms |
| Hosting | Vercel | Serverless deployment, edge functions |
| Monitoring | Vercel Analytics + Supabase Logs | Performance and error tracking |

### System Boundaries

**In Scope**:
- Web application (desktop and mobile-responsive)
- Role-based access control (Admin, Faculty, Student)
- AI-powered learning assistance
- Assignment and content management
- Google Workspace integrations
- Automated notifications and workflows

**Out of Scope**:
- Native mobile applications
- Video conferencing infrastructure (integration only)
- Payment processing
- Multi-tenant architecture (single institution)
- Real-time video/audio communication

---

## Architecture Diagram

### High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │    Admin     │  │   Faculty    │  │   Student    │              │
│  │   Portal     │  │   Portal     │  │   Portal     │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                  │                  │                       │
│         └──────────────────┼──────────────────┘                       │
│                            │                                          │
└────────────────────────────┼──────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      PRESENTATION LAYER                              │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    Next.js Application                         │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │  │
│  │  │   Server    │  │   Client    │  │     API     │           │  │
│  │  │ Components  │  │ Components  │  │   Routes    │           │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘           │  │
│  │  ┌─────────────────────────────────────────────────┐          │  │
│  │  │         Middleware (Auth, RBAC, Logging)        │          │  │
│  │  └─────────────────────────────────────────────────┘          │  │
│  └───────────────────────────────────────────────────────────────┘  │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                               │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    Supabase Backend                           │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐             │   │
│  │  │    Auth    │  │ PostgreSQL │  │  Storage   │             │   │
│  │  │  Service   │  │  Database  │  │  Buckets   │             │   │
│  │  └────────────┘  └────────────┘  └────────────┘             │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐             │   │
│  │  │    RLS     │  │   Edge     │  │ Real-time  │             │   │
│  │  │  Policies  │  │ Functions  │  │  Channels  │             │   │
│  │  └────────────┘  └────────────┘  └────────────┘             │   │
│  └──────────────────────────────────────────────────────────────┘   │
└───────────┬──────────────────────────────────┬──────────────────────┘
            │                                  │
            ▼                                  ▼
┌─────────────────────────┐      ┌─────────────────────────────────┐
│   INTEGRATION LAYER     │      │      AI SERVICES LAYER          │
│  ┌───────────────────┐  │      │  ┌───────────────────────────┐  │
│  │   n8n Workflows   │  │      │  │    Google Gemini API      │  │
│  │                   │  │      │  │   (Vertex AI)             │  │
│  │ • Notifications   │  │      │  │                           │  │
│  │ • Google APIs     │  │      │  │ • Document Chat           │  │
│  │ • Data Sync       │  │      │  │ • Quiz Generation         │  │
│  │ • Scheduled Jobs  │  │      │  │ • Summarization           │  │
│  └───────────────────┘  │      │  │ • Study Plans             │  │
│           │              │      │  │ • Doubt Resolution        │  │
│           ▼              │      │  └───────────────────────────┘  │
│  ┌───────────────────┐  │      │              │                   │
│  │  Google Workspace │  │      │              ▼                   │
│  │                   │  │      │  ┌───────────────────────────┐  │
│  │ • Drive API       │  │      │  │     AI Cache Layer        │  │
│  │ • Calendar API    │  │      │  │  (PostgreSQL + Redis)     │  │
│  │ • Meet API        │  │      │  └───────────────────────────┘  │
│  │ • Forms API       │  │      │                                  │
│  └───────────────────┘  │      └──────────────────────────────────┘
└─────────────────────────┘
```

### Component Interaction Flow

```
User Request Flow:
1. User → Next.js Middleware (Auth Check)
2. Middleware → Supabase Auth (Verify JWT)
3. Next.js → Supabase Database (Query with RLS)
4. Database → Next.js (Filtered Data)
5. Next.js → User (Rendered Page)

AI Request Flow:
1. User → Next.js API Route
2. API Route → Cache Check (PostgreSQL)
3. If Miss → Supabase Edge Function
4. Edge Function → Google Gemini API
5. Gemini → Edge Function (Response)
6. Edge Function → Cache Store
7. Edge Function → Next.js API
8. Next.js API → User

Automation Flow:
1. Database Event → Supabase Webhook
2. Webhook → n8n Workflow Trigger
3. n8n → Google Workspace API
4. n8n → Email/Notification Service
5. n8n → Database Update (Log)
```

---

## Component-Level Design

### 1. Frontend Layer (Next.js Application)

#### 1.1 Application Structure

```
sparklink/
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── (auth)/                   # Authentication routes
│   │   │   ├── login/
│   │   │   ├── logout/
│   │   │   └── callback/
│   │   ├── (protected)/              # Protected route group
│   │   │   ├── admin/                # Admin portal
│   │   │   │   ├── dashboard/
│   │   │   │   ├── users/
│   │   │   │   ├── institutions/
│   │   │   │   ├── analytics/
│   │   │   │   └── settings/
│   │   │   ├── faculty/              # Faculty portal
│   │   │   │   ├── dashboard/
│   │   │   │   ├── classes/
│   │   │   │   ├── assignments/
│   │   │   │   ├── resources/
│   │   │   │   ├── ai-tools/
│   │   │   │   └── analytics/
│   │   │   └── student/              # Student portal
│   │   │       ├── dashboard/
│   │   │       ├── assignments/
│   │   │       ├── resources/
│   │   │       ├── ai-assistant/
│   │   │       ├── study-planner/
│   │   │       └── forum/
│   │   ├── api/                      # API routes (minimal)
│   │   │   ├── health/
│   │   │   └── webhooks/
│   │   ├── layout.tsx                # Root layout
│   │   └── middleware.ts             # Auth & RBAC middleware
│   ├── components/                   # React components
│   │   ├── ui/                       # ShadCN UI components
│   │   ├── layouts/                  # Layout components
│   │   ├── forms/                    # Form components
│   │   ├── admin/                    # Admin-specific components
│   │   ├── faculty/                  # Faculty-specific components
│   │   └── student/                  # Student-specific components
│   ├── lib/                          # Utilities and configurations
│   │   ├── supabase/                 # Supabase client & helpers
│   │   │   ├── client.ts             # Browser client
│   │   │   ├── server.ts             # Server client
│   │   │   └── middleware.ts         # Middleware client
│   │   ├── auth/                     # Authentication utilities
│   │   ├── api/                      # API client functions
│   │   ├── utils/                    # General utilities
│   │   └── constants/                # App constants
│   ├── hooks/                        # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── useUser.ts
│   │   ├── useAssignments.ts
│   │   └── useAI.ts
│   ├── types/                        # TypeScript types
│   │   ├── database.ts               # Database types
│   │   ├── api.ts                    # API types
│   │   └── models.ts                 # Domain models
│   └── styles/                       # Global styles
│       └── globals.css
├── supabase/                         # Supabase configuration
│   ├── migrations/                   # Database migrations
│   ├── functions/                    # Edge functions
│   │   ├── ai-chat/
│   │   ├── ai-quiz-generator/
│   │   ├── ai-summarize/
│   │   └── file-processor/
│   └── config.toml                   # Supabase config
├── public/                           # Static assets
└── package.json
```

#### 1.2 Key Frontend Components

**Middleware Layer**
```typescript
// src/app/middleware.ts
export async function middleware(request: NextRequest) {
  // 1. Verify authentication
  const supabase = createMiddlewareClient({ req, res });
  const { data: { session } } = await supabase.auth.getSession();
  
  if (!session) {
    return NextResponse.redirect('/login');
  }
  
  // 2. Check role-based access
  const userRole = session.user.user_metadata.role;
  const path = request.nextUrl.pathname;
  
  if (!hasAccess(userRole, path)) {
    return NextResponse.redirect('/unauthorized');
  }
  
  // 3. Add security headers
  const response = NextResponse.next();
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  
  return response;
}
```

**Server Components (Default)**
```typescript
// src/app/(protected)/student/dashboard/page.tsx
export default async function StudentDashboard() {
  const supabase = createServerClient();
  
  // Fetch data server-side with RLS
  const { data: assignments } = await supabase
    .from('assignments')
    .select('*')
    .order('due_date', { ascending: true })
    .limit(5);
  
  return <DashboardView assignments={assignments} />;
}
```

**Client Components (Interactive)**
```typescript
// src/components/student/AIAssistant.tsx
'use client';

export function AIAssistant() {
  const [query, setQuery] = useState('');
  const [response, setResponse] = useState('');
  const [loading, setLoading] = useState(false);
  
  const handleSubmit = async () => {
    setLoading(true);
    const result = await fetch('/api/ai/chat', {
      method: 'POST',
      body: JSON.stringify({ query }),
    });
    const data = await result.json();
    setResponse(data.response);
    setLoading(false);
  };
  
  return (
    <div>
      <textarea value={query} onChange={(e) => setQuery(e.target.value)} />
      <button onClick={handleSubmit} disabled={loading}>
        {loading ? 'Thinking...' : 'Ask AI'}
      </button>
      {response && <div>{response}</div>}
    </div>
  );
}
```

---

### 2. Backend Layer (Supabase)

#### 2.1 Supabase Services Architecture

**Authentication Service**
- JWT-based authentication with refresh tokens
- Role assignment via user metadata
- Session management with configurable timeout
- Password reset and email verification

**Database Service**
- PostgreSQL 15+ with JSONB support
- Row-Level Security (RLS) for multi-tenant isolation
- Automatic timestamps and soft deletes
- Full-text search capabilities

**Storage Service**
- Object storage with CDN distribution
- Bucket-level access policies
- Automatic image optimization
- Signed URLs for secure access

**Real-time Service**
- WebSocket-based subscriptions
- Row-level change notifications
- Presence tracking for online users
- Broadcast channels for messaging

**Edge Functions**
- Deno-based serverless functions
- AI service integration layer
- File processing and validation
- Webhook handlers

#### 2.2 Edge Functions Design

**AI Chat Function**
```typescript
// supabase/functions/ai-chat/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from '@supabase/supabase-js';

serve(async (req) => {
  const { query, documentId, userId } = await req.json();
  
  // 1. Verify authentication
  const authHeader = req.headers.get('Authorization');
  const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY, {
    global: { headers: { Authorization: authHeader } }
  });
  
  // 2. Check cache
  const cacheKey = `chat:${documentId}:${hashQuery(query)}`;
  const { data: cached } = await supabase
    .from('ai_cache')
    .select('response')
    .eq('cache_key', cacheKey)
    .gt('expires_at', new Date().toISOString())
    .single();
  
  if (cached) {
    return new Response(JSON.stringify(cached.response), {
      headers: { 'Content-Type': 'application/json' }
    });
  }
  
  // 3. Fetch document content
  const { data: document } = await supabase
    .from('resources')
    .select('content')
    .eq('id', documentId)
    .single();
  
  // 4. Call Gemini API
  const geminiResponse = await fetch(GEMINI_API_URL, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${GEMINI_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      contents: [{
        parts: [{
          text: `Context: ${document.content}\n\nQuestion: ${query}`
        }]
      }],
      generationConfig: {
        temperature: 0.7,
        maxOutputTokens: 1024
      }
    })
  });
  
  const aiResponse = await geminiResponse.json();
  const answer = aiResponse.candidates[0].content.parts[0].text;
  
  // 5. Cache response
  await supabase.from('ai_cache').insert({
    cache_key: cacheKey,
    response: answer,
    expires_at: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString(),
    user_id: userId
  });
  
  // 6. Log usage
  await supabase.from('ai_usage_logs').insert({
    user_id: userId,
    feature: 'document_chat',
    tokens_used: aiResponse.usageMetadata.totalTokenCount
  });
  
  return new Response(JSON.stringify({ answer }), {
    headers: { 'Content-Type': 'application/json' }
  });
});
```

**Quiz Generator Function**
```typescript
// supabase/functions/ai-quiz-generator/index.ts
serve(async (req) => {
  const { content, questionCount, difficulty, userId } = await req.json();
  
  // Rate limiting check
  const { count } = await supabase
    .from('ai_usage_logs')
    .select('*', { count: 'exact', head: true })
    .eq('user_id', userId)
    .eq('feature', 'quiz_generator')
    .gte('created_at', new Date(Date.now() - 60 * 60 * 1000).toISOString());
  
  if (count >= RATE_LIMIT) {
    return new Response(JSON.stringify({ error: 'Rate limit exceeded' }), {
      status: 429
    });
  }
  
  // Generate quiz using Gemini
  const prompt = `Generate ${questionCount} ${difficulty} multiple-choice questions from the following content:\n\n${content}\n\nFormat as JSON array with structure: [{ question, options: [], correctAnswer, explanation }]`;
  
  const geminiResponse = await callGeminiAPI(prompt);
  const quiz = JSON.parse(geminiResponse);
  
  // Validate quiz format
  if (!validateQuizFormat(quiz)) {
    throw new Error('Invalid quiz format generated');
  }
  
  // Store quiz
  const { data: savedQuiz } = await supabase
    .from('quizzes')
    .insert({
      created_by: userId,
      questions: quiz,
      difficulty,
      source_content_hash: hashContent(content)
    })
    .select()
    .single();
  
  return new Response(JSON.stringify(savedQuiz), {
    headers: { 'Content-Type': 'application/json' }
  });
});
```

---

### 3. Integration Layer (n8n Workflows)

#### 3.1 Workflow Architecture

**Workflow Categories**:
1. **Notification Workflows**: Email, in-app, push notifications
2. **Google Integration Workflows**: Drive sync, Calendar events, Forms
3. **Data Processing Workflows**: Bulk imports, exports, cleanup
4. **Monitoring Workflows**: Health checks, usage reports

#### 3.2 Key Workflow Designs

**Assignment Notification Workflow**
```
Trigger: Database Webhook (new assignment created)
↓
1. Fetch enrolled students for class
   - Query: SELECT student_id FROM student_classes WHERE class_id = ?
↓
2. Create Google Calendar events
   - For each student: Create calendar event with due date
   - Add assignment link to event description
↓
3. Send email notifications
   - Template: Assignment notification with details
   - Batch send to all students
↓
4. Create in-app notifications
   - Insert into notifications table
   - Trigger real-time update via Supabase
↓
5. Schedule reminder workflow
   - Schedule 24h before due date
   - Schedule 1h before due date
↓
6. Log workflow execution
   - Update workflow_logs table
   - Track success/failure metrics
```

**Google Drive Sync Workflow**
```
Trigger: File uploaded to Supabase Storage
↓
1. Download file from Supabase Storage
   - Get signed URL
   - Download file content
↓
2. Determine target folder
   - Based on class_id and resource type
   - Create folder structure if not exists
↓
3. Upload to Google Drive
   - Use Drive API v3
   - Set file permissions based on sharing settings
↓
4. Store Drive file ID
   - Update resources table with drive_file_id
   - Enable two-way sync
↓
5. Handle errors
   - Retry on transient failures (3 attempts)
   - Alert admin on permanent failures
```

**Daily Digest Workflow**
```
Trigger: Cron schedule (daily at 8 AM)
↓
1. Fetch users with digest enabled
   - Query: SELECT * FROM profiles WHERE email_digest = true
↓
2. For each user, compile digest
   - Upcoming assignments (next 7 days)
   - New resources added
   - Unread notifications
   - AI usage summary
↓
3. Generate personalized email
   - Use HTML template
   - Include user-specific data
↓
4. Send via email service
   - Batch send with rate limiting
   - Track delivery status
↓
5. Update last_digest_sent timestamp
```

---

### 4. AI Services Layer

#### 4.1 AI Service Architecture

**Components**:
- **API Gateway**: Edge functions handling AI requests
- **Cache Layer**: PostgreSQL + Redis for response caching
- **Rate Limiter**: Per-user, per-feature rate limiting
- **Usage Tracker**: Token consumption and cost monitoring
- **Quality Filter**: Response validation and safety checks

#### 4.2 AI Feature Implementations

**Document Summarization**
```typescript
interface SummarizationRequest {
  documentId: string;
  maxLength: number;
  style: 'concise' | 'detailed' | 'bullet-points';
}

async function summarizeDocument(req: SummarizationRequest) {
  // 1. Fetch document
  const document = await getDocument(req.documentId);
  
  // 2. Check cache
  const cacheKey = `summary:${req.documentId}:${req.style}`;
  const cached = await getFromCache(cacheKey);
  if (cached) return cached;
  
  // 3. Prepare prompt
  const prompt = `Summarize the following document in ${req.style} style, maximum ${req.maxLength} words:\n\n${document.content}`;
  
  // 4. Call Gemini
  const response = await callGemini({
    prompt,
    temperature: 0.3,
    maxTokens: req.maxLength * 2
  });
  
  // 5. Cache and return
  await cacheResponse(cacheKey, response, '7d');
  return response;
}
```

**Study Plan Generator**
```typescript
interface StudyPlanRequest {
  studentId: string;
  subjects: string[];
  examDate: Date;
  hoursPerDay: number;
}

async function generateStudyPlan(req: StudyPlanRequest) {
  // 1. Fetch student context
  const studentData = await getStudentContext(req.studentId);
  
  // 2. Calculate available days
  const daysUntilExam = Math.floor(
    (req.examDate.getTime() - Date.now()) / (1000 * 60 * 60 * 24)
  );
  
  // 3. Build prompt with context
  const prompt = `
    Create a personalized study plan for a student with:
    - Subjects: ${req.subjects.join(', ')}
    - Days until exam: ${daysUntilExam}
    - Available hours per day: ${req.hoursPerDay}
    - Current performance: ${JSON.stringify(studentData.grades)}
    - Weak areas: ${studentData.weakAreas.join(', ')}
    
    Provide a day-by-day breakdown with topics, time allocation, and study techniques.
    Format as JSON: { days: [{ date, subjects: [{ name, topics, hours, techniques }] }] }
  `;
  
  // 4. Generate plan
  const response = await callGemini({
    prompt,
    temperature: 0.7,
    responseFormat: 'json'
  });
  
  // 5. Validate and store
  const plan = JSON.parse(response);
  await supabase.from('study_plans').insert({
    student_id: req.studentId,
    plan_data: plan,
    exam_date: req.examDate
  });
  
  return plan;
}
```

**Doubt Resolution Engine**
```typescript
interface DoubtRequest {
  studentId: string;
  question: string;
  subject: string;
  context?: string;
}

async function resolveDoubt(req: DoubtRequest) {
  // 1. Check similar questions cache
  const similarQuestions = await findSimilarQuestions(req.question);
  if (similarQuestions.length > 0) {
    return {
      answer: similarQuestions[0].answer,
      cached: true,
      confidence: similarQuestions[0].similarity
    };
  }
  
  // 2. Fetch relevant course materials
  const relevantMaterials = await searchCourseMaterials(
    req.subject,
    req.question
  );
  
  // 3. Build context-aware prompt
  const prompt = `
    Subject: ${req.subject}
    Question: ${req.question}
    ${req.context ? `Additional Context: ${req.context}` : ''}
    
    Relevant Course Materials:
    ${relevantMaterials.map(m => m.content).join('\n\n')}
    
    Provide a clear, educational answer that:
    1. Directly addresses the question
    2. Explains underlying concepts
    3. Provides examples if applicable
    4. Suggests related topics to explore
  `;
  
  // 4. Generate answer
  const answer = await callGemini({
    prompt,
    temperature: 0.5,
    maxTokens: 1024
  });
  
  // 5. Store for future reference
  await supabase.from('doubt_resolutions').insert({
    student_id: req.studentId,
    question: req.question,
    answer: answer,
    subject: req.subject,
    materials_used: relevantMaterials.map(m => m.id)
  });
  
  return { answer, cached: false };
}
```

---

## Data Flow Explanation

### 1. User Authentication Flow

```
Step 1: User Login Request
User → Next.js Login Page → Supabase Auth API
↓
Step 2: Credential Verification
Supabase Auth → Verify credentials → Generate JWT + Refresh Token
↓
Step 3: Session Creation
JWT stored in httpOnly cookie → User metadata (role) attached
↓
Step 4: Redirect to Portal
Middleware checks role → Redirect to appropriate portal (admin/faculty/student)
↓
Step 5: Subsequent Requests
All requests include JWT → Middleware validates → RLS filters data by role
```

**Sequence Diagram**:
```
User          Next.js       Supabase Auth    Database
 │               │                │              │
 │─Login Form───>│                │              │
 │               │─Auth Request──>│              │
 │               │                │─Verify───────>│
 │               │                │<─User Data───│
 │               │<─JWT + Cookie──│              │
 │<─Redirect─────│                │              │
 │               │                │              │
 │─Page Request─>│                │              │
 │               │─Verify JWT────>│              │
 │               │<─Valid─────────│              │
 │               │─Query (RLS)────────────────────>│
 │               │<─Filtered Data─────────────────│
 │<─Render Page──│                │              │
```

---

### 2. Assignment Creation & Submission Flow

**Faculty Creates Assignment**:
```
1. Faculty → Next.js Form → Validate Input
2. Next.js → Supabase Database → Insert assignment record
3. Database Trigger → Webhook → n8n Workflow
4. n8n → Fetch enrolled students → Create notifications
5. n8n → Google Calendar API → Create calendar events
6. n8n → Email Service → Send notification emails
7. n8n → Supabase → Update notification_logs
8. Supabase Real-time → Push notification to online students
```

**Student Submits Assignment**:
```
1. Student → Upload File → Next.js API Route
2. Next.js → Supabase Storage → Store file (bucket: submissions)
3. Storage → Generate signed URL → Return to Next.js
4. Next.js → Database → Insert submission record with file URL
5. Database → RLS Check → Verify student enrolled in class
6. Database Trigger → n8n Workflow → Notify faculty
7. Faculty Dashboard → Real-time update → Show new submission
```

---

### 3. AI-Powered Feature Flow

**Document Chat Flow**:
```
1. Student → Ask question about document → Next.js UI
2. Next.js → Supabase Edge Function (ai-chat)
3. Edge Function → Check ai_cache table
   ├─ Cache Hit → Return cached response (< 100ms)
   └─ Cache Miss → Continue to step 4
4. Edge Function → Fetch document content from database
5. Edge Function → Build prompt with context
6. Edge Function → Google Gemini API → Generate response
7. Gemini → Return answer → Edge Function
8. Edge Function → Store in ai_cache (24h TTL)
9. Edge Function → Log usage in ai_usage_logs
10. Edge Function → Return response to Next.js
11. Next.js → Display answer to student
```

**Quiz Generation Flow**:
```
1. Faculty → Provide content + parameters → Next.js
2. Next.js → Rate limit check (50 requests/hour for faculty)
3. Next.js → Supabase Edge Function (ai-quiz-generator)
4. Edge Function → Build structured prompt
5. Edge Function → Gemini API (with JSON response format)
6. Gemini → Generate questions → Return JSON
7. Edge Function → Validate quiz format
8. Edge Function → Store quiz in database
9. Edge Function → Optional: Create Google Form via n8n
10. Next.js → Display quiz to faculty for review
```

---

### 4. File Upload & Sync Flow

```
1. User → Select file → Next.js Upload Component
2. Next.js → Validate file (type, size) → Show progress
3. Next.js → Supabase Storage API → Upload to bucket
4. Storage → Store file → Generate file path
5. Storage → Trigger webhook → n8n Workflow
6. n8n → Download file from Supabase
7. n8n → Google Drive API → Upload to institutional Drive
8. Drive → Return file ID → n8n
9. n8n → Update database → Store drive_file_id
10. n8n → Set Drive permissions based on sharing settings
11. Database → Update resources table → Mark as synced
12. Real-time → Notify users → File available
```

---

### 5. Notification & Alert Flow

**Real-time Notification**:
```
Event Trigger (e.g., new grade posted)
↓
Database Trigger → Supabase Webhook → n8n
↓
n8n Parallel Processing:
├─ Insert into notifications table
│  └─ Supabase Real-time → Push to online users
├─ Send email via SMTP/SendGrid
│  └─ Track delivery status
└─ Create Google Calendar reminder
   └─ Update calendar_events table
```

**Scheduled Reminder**:
```
n8n Cron Job (runs every hour)
↓
Query assignments due in next 24 hours
↓
For each assignment:
├─ Fetch students with pending submissions
├─ Check if reminder already sent
├─ Generate personalized reminder
├─ Send via email + in-app notification
└─ Mark reminder as sent
```

---

### 6. Analytics & Reporting Flow

```
Background Job (daily at midnight)
↓
1. Aggregate user activity data
   - Login frequency
   - Feature usage
   - Time spent per portal
↓
2. Calculate engagement metrics
   - Assignment completion rates
   - Resource access patterns
   - AI feature adoption
↓
3. Generate performance insights
   - Student grade trends
   - Faculty workload distribution
   - System usage patterns
↓
4. Store in analytics tables
   - Daily snapshots
   - Weekly aggregates
   - Monthly reports
↓
5. Update dashboard caches
   - Pre-compute common queries
   - Generate chart data
↓
6. Send digest to admins
   - Email with key metrics
   - Link to detailed dashboard
```

---

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

## Database Design

### Entity Relationship Diagram

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│  institutions   │         │   departments    │         │     classes     │
├─────────────────┤         ├──────────────────┤         ├─────────────────┤
│ id (PK)         │────────<│ id (PK)          │────────<│ id (PK)         │
│ name            │         │ institution_id   │         │ department_id   │
│ code            │         │ name             │         │ name            │
│ settings        │         │ code             │         │ code            │
│ created_at      │         │ head_faculty_id  │         │ faculty_id (FK) │
└─────────────────┘         │ created_at       │         │ academic_year   │
                            └──────────────────┘         │ semester        │
                                                          │ created_at      │
                                                          └─────────────────┘
                                                                  │
                                                                  │
┌─────────────────┐         ┌──────────────────┐                │
│    profiles     │         │ student_classes  │                │
├─────────────────┤         ├──────────────────┤                │
│ id (PK, FK)     │────────<│ id (PK)          │>───────────────┘
│ role            │         │ student_id (FK)  │
│ full_name       │         │ class_id (FK)    │
│ email           │         │ enrolled_at      │
│ institution_id  │         └──────────────────┘
│ department_id   │
│ created_at      │                 │
│ updated_at      │                 │
└─────────────────┘                 │
        │                           │
        │                           │
        │         ┌─────────────────┴────────┐
        │         │                          │
        │         ▼                          ▼
        │  ┌──────────────────┐    ┌──────────────────┐
        │  │   assignments    │    │   submissions    │
        │  ├──────────────────┤    ├──────────────────┤
        │  │ id (PK)          │───<│ id (PK)          │
        │  │ class_id (FK)    │    │ assignment_id    │
        │  │ faculty_id (FK)  │    │ student_id (FK)  │
        │  │ title            │    │ content          │
        │  │ description      │    │ file_attachments │
        │  │ due_date         │    │ score            │
        │  │ max_score        │    │ feedback         │
        │  │ file_attachments │    │ submitted_at     │
        │  │ created_at       │    │ graded_at        │
        │  └──────────────────┘    └──────────────────┘
        │
        │
        └──────────────────────────────────────────────┐
                                                       │
                                                       ▼
        ┌──────────────────┐         ┌──────────────────────┐
        │    resources     │         │   notifications      │
        ├──────────────────┤         ├──────────────────────┤
        │ id (PK)          │         │ id (PK)              │
        │ class_id (FK)    │         │ user_id (FK)         │
        │ faculty_id (FK)  │         │ type                 │
        │ title            │         │ title                │
        │ description      │         │ message              │
        │ file_path        │         │ link                 │
        │ file_type        │         │ read                 │
        │ file_size        │         │ created_at           │
        │ drive_file_id    │         └──────────────────────┘
        │ is_public        │
        │ created_at       │
        └──────────────────┘
```

### Core Tables Schema

#### 1. Authentication & User Management

**profiles** (extends auth.users)
```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  role user_role NOT NULL,
  full_name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  phone TEXT,
  avatar_url TEXT,
  institution_id UUID REFERENCES institutions(id),
  department_id UUID REFERENCES departments(id),
  metadata JSONB DEFAULT '{}',
  preferences JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT true,
  last_login_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  CONSTRAINT valid_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')
);

CREATE TYPE user_role AS ENUM ('admin', 'faculty', 'student');

CREATE INDEX idx_profiles_role ON profiles(role);
CREATE INDEX idx_profiles_institution ON profiles(institution_id);
CREATE INDEX idx_profiles_email ON profiles(email);
```

**institutions**
```sql
CREATE TABLE institutions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  code TEXT UNIQUE NOT NULL,
  address TEXT,
  contact_email TEXT,
  contact_phone TEXT,
  logo_url TEXT,
  settings JSONB DEFAULT '{
    "academic_year_start": "08-01",
    "semester_system": true,
    "max_file_size_mb": 100,
    "ai_features_enabled": true
  }',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**departments**
```sql
CREATE TABLE departments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  institution_id UUID NOT NULL REFERENCES institutions(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  code TEXT NOT NULL,
  head_faculty_id UUID REFERENCES profiles(id),
  description TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(institution_id, code)
);

CREATE INDEX idx_departments_institution ON departments(institution_id);
```

#### 2. Academic Structure

**classes**
```sql
CREATE TABLE classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  department_id UUID NOT NULL REFERENCES departments(id) ON DELETE CASCADE,
  faculty_id UUID NOT NULL REFERENCES profiles(id),
  name TEXT NOT NULL,
  code TEXT NOT NULL,
  description TEXT,
  academic_year TEXT NOT NULL,
  semester INTEGER NOT NULL CHECK (semester BETWEEN 1 AND 8),
  section TEXT,
  max_students INTEGER DEFAULT 100,
  schedule JSONB DEFAULT '[]', -- [{ day, start_time, end_time, room }]
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(department_id, code, academic_year, semester, section)
);

CREATE INDEX idx_classes_faculty ON classes(faculty_id);
CREATE INDEX idx_classes_department ON classes(department_id);
CREATE INDEX idx_classes_active ON classes(is_active) WHERE is_active = true;
```

**student_classes** (enrollment)
```sql
CREATE TABLE student_classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  class_id UUID NOT NULL REFERENCES classes(id) ON DELETE CASCADE,
  enrollment_status enrollment_status DEFAULT 'active',
  enrolled_at TIMESTAMPTZ DEFAULT NOW(),
  dropped_at TIMESTAMPTZ,
  final_grade TEXT,
  
  UNIQUE(student_id, class_id)
);

CREATE TYPE enrollment_status AS ENUM ('active', 'dropped', 'completed');

CREATE INDEX idx_student_classes_student ON student_classes(student_id);
CREATE INDEX idx_student_classes_class ON student_classes(class_id);
CREATE INDEX idx_student_classes_status ON student_classes(enrollment_status);
```

#### 3. Content & Assignments

**resources**
```sql
CREATE TABLE resources (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id UUID NOT NULL REFERENCES classes(id) ON DELETE CASCADE,
  faculty_id UUID NOT NULL REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  file_path TEXT NOT NULL,
  file_type TEXT NOT NULL,
  file_size BIGINT NOT NULL,
  drive_file_id TEXT, -- Google Drive sync
  content_text TEXT, -- Extracted text for search/AI
  tags TEXT[] DEFAULT '{}',
  is_public BOOLEAN DEFAULT false,
  view_count INTEGER DEFAULT 0,
  download_count INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_resources_class ON resources(class_id);
CREATE INDEX idx_resources_faculty ON resources(faculty_id);
CREATE INDEX idx_resources_tags ON resources USING GIN(tags);
CREATE INDEX idx_resources_search ON resources USING GIN(to_tsvector('english', title || ' ' || COALESCE(description, '')));
```

**assignments**
```sql
CREATE TABLE assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id UUID NOT NULL REFERENCES classes(id) ON DELETE CASCADE,
  faculty_id UUID NOT NULL REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  instructions TEXT,
  file_attachments TEXT[] DEFAULT '{}',
  due_date TIMESTAMPTZ NOT NULL,
  max_score INTEGER DEFAULT 100,
  submission_type submission_type DEFAULT 'file',
  allow_late_submission BOOLEAN DEFAULT false,
  late_penalty_percent INTEGER DEFAULT 0,
  status assignment_status DEFAULT 'draft',
  published_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  CHECK (due_date > created_at),
  CHECK (max_score > 0),
  CHECK (late_penalty_percent BETWEEN 0 AND 100)
);

CREATE TYPE assignment_status AS ENUM ('draft', 'published', 'closed');
CREATE TYPE submission_type AS ENUM ('file', 'text', 'link', 'quiz');

CREATE INDEX idx_assignments_class ON assignments(class_id);
CREATE INDEX idx_assignments_faculty ON assignments(faculty_id);
CREATE INDEX idx_assignments_due_date ON assignments(due_date);
CREATE INDEX idx_assignments_status ON assignments(status);
```

**submissions**
```sql
CREATE TABLE submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
  student_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  content TEXT,
  file_attachments TEXT[] DEFAULT '{}',
  submission_link TEXT,
  score INTEGER,
  feedback TEXT,
  is_late BOOLEAN DEFAULT false,
  submitted_at TIMESTAMPTZ DEFAULT NOW(),
  graded_at TIMESTAMPTZ,
  graded_by UUID REFERENCES profiles(id),
  version INTEGER DEFAULT 1,
  
  UNIQUE(assignment_id, student_id),
  CHECK (score IS NULL OR (score >= 0 AND score <= (SELECT max_score FROM assignments WHERE id = assignment_id)))
);

CREATE INDEX idx_submissions_assignment ON submissions(assignment_id);
CREATE INDEX idx_submissions_student ON submissions(student_id);
CREATE INDEX idx_submissions_graded ON submissions(graded_at) WHERE graded_at IS NOT NULL;
```

#### 4. AI & Caching

**ai_cache**
```sql
CREATE TABLE ai_cache (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  cache_key TEXT UNIQUE NOT NULL,
  prompt_hash TEXT NOT NULL,
  feature_type TEXT NOT NULL, -- 'chat', 'quiz', 'summary', 'study_plan'
  request_data JSONB NOT NULL,
  response_data JSONB NOT NULL,
  model_version TEXT NOT NULL DEFAULT 'gemini-pro',
  tokens_used INTEGER,
  expires_at TIMESTAMPTZ NOT NULL,
  hit_count INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  last_accessed_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_ai_cache_key ON ai_cache(cache_key);
CREATE INDEX idx_ai_cache_expires ON ai_cache(expires_at);
CREATE INDEX idx_ai_cache_feature ON ai_cache(feature_type);
```

**ai_usage_logs**
```sql
CREATE TABLE ai_usage_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  feature_type TEXT NOT NULL,
  prompt_tokens INTEGER NOT NULL,
  completion_tokens INTEGER NOT NULL,
  total_tokens INTEGER NOT NULL,
  response_time_ms INTEGER,
  was_cached BOOLEAN DEFAULT false,
  cost_usd DECIMAL(10, 6),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_ai_usage_user ON ai_usage_logs(user_id);
CREATE INDEX idx_ai_usage_feature ON ai_usage_logs(feature_type);
CREATE INDEX idx_ai_usage_date ON ai_usage_logs(created_at);
```

**study_plans**
```sql
CREATE TABLE study_plans (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  exam_date DATE,
  subjects TEXT[] NOT NULL,
  hours_per_day INTEGER NOT NULL,
  plan_data JSONB NOT NULL, -- Generated plan structure
  progress JSONB DEFAULT '{}', -- Track completion
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_study_plans_student ON study_plans(student_id);
CREATE INDEX idx_study_plans_active ON study_plans(is_active) WHERE is_active = true;
```

#### 5. Communication & Notifications

**notifications**
```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  type notification_type NOT NULL,
  title TEXT NOT NULL,
  message TEXT NOT NULL,
  link TEXT,
  metadata JSONB DEFAULT '{}',
  is_read BOOLEAN DEFAULT false,
  read_at TIMESTAMPTZ,
  priority notification_priority DEFAULT 'normal',
  expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TYPE notification_type AS ENUM (
  'assignment_created',
  'assignment_due_soon',
  'submission_graded',
  'resource_added',
  'announcement',
  'meeting_scheduled',
  'system_alert'
);

CREATE TYPE notification_priority AS ENUM ('low', 'normal', 'high', 'urgent');

CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_read ON notifications(is_read);
CREATE INDEX idx_notifications_created ON notifications(created_at DESC);
```

**doubt_forum**
```sql
CREATE TABLE doubt_forum (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id UUID NOT NULL REFERENCES classes(id) ON DELETE CASCADE,
  student_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  question TEXT NOT NULL,
  tags TEXT[] DEFAULT '{}',
  is_resolved BOOLEAN DEFAULT false,
  resolved_at TIMESTAMPTZ,
  upvotes INTEGER DEFAULT 0,
  view_count INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_doubt_forum_class ON doubt_forum(class_id);
CREATE INDEX idx_doubt_forum_student ON doubt_forum(student_id);
CREATE INDEX idx_doubt_forum_resolved ON doubt_forum(is_resolved);
CREATE INDEX idx_doubt_forum_tags ON doubt_forum USING GIN(tags);
```

**doubt_responses**
```sql
CREATE TABLE doubt_responses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  doubt_id UUID NOT NULL REFERENCES doubt_forum(id) ON DELETE CASCADE,
  responder_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  response_text TEXT NOT NULL,
  is_accepted BOOLEAN DEFAULT false,
  upvotes INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_doubt_responses_doubt ON doubt_responses(doubt_id);
CREATE INDEX idx_doubt_responses_responder ON doubt_responses(responder_id);
```

#### 6. Analytics & Logging

**activity_logs**
```sql
CREATE TABLE activity_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE SET NULL,
  action TEXT NOT NULL,
  resource_type TEXT,
  resource_id UUID,
  metadata JSONB DEFAULT '{}',
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_activity_logs_user ON activity_logs(user_id);
CREATE INDEX idx_activity_logs_action ON activity_logs(action);
CREATE INDEX idx_activity_logs_created ON activity_logs(created_at DESC);
```

**analytics_snapshots**
```sql
CREATE TABLE analytics_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  snapshot_date DATE NOT NULL,
  snapshot_type TEXT NOT NULL, -- 'daily', 'weekly', 'monthly'
  metrics JSONB NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(snapshot_date, snapshot_type)
);

CREATE INDEX idx_analytics_date ON analytics_snapshots(snapshot_date DESC);
CREATE INDEX idx_analytics_type ON analytics_snapshots(snapshot_type);
```

### Row-Level Security (RLS) Policies

#### Admin Policies (Full Access)
```sql
-- Admins can do everything
CREATE POLICY "admin_all_access" ON profiles
  FOR ALL USING (
    (SELECT role FROM profiles WHERE id = auth.uid()) = 'admin'
  );

CREATE POLICY "admin_all_institutions" ON institutions
  FOR ALL USING (
    (SELECT role FROM profiles WHERE id = auth.uid()) = 'admin'
  );
```

#### Faculty Policies
```sql
-- Faculty can view all students in their classes
CREATE POLICY "faculty_view_students" ON profiles
  FOR SELECT USING (
    role = 'student' AND
    id IN (
      SELECT sc.student_id 
      FROM student_classes sc
      JOIN classes c ON c.id = sc.class_id
      WHERE c.faculty_id = auth.uid()
    )
  );

-- Faculty can manage their own assignments
CREATE POLICY "faculty_own_assignments" ON assignments
  FOR ALL USING (faculty_id = auth.uid());

-- Faculty can view submissions for their assignments
CREATE POLICY "faculty_view_submissions" ON submissions
  FOR SELECT USING (
    assignment_id IN (
      SELECT id FROM assignments WHERE faculty_id = auth.uid()
    )
  );

-- Faculty can grade submissions
CREATE POLICY "faculty_grade_submissions" ON submissions
  FOR UPDATE USING (
    assignment_id IN (
      SELECT id FROM assignments WHERE faculty_id = auth.uid()
    )
  );
```

#### Student Policies
```sql
-- Students can view their own profile
CREATE POLICY "student_own_profile" ON profiles
  FOR SELECT USING (id = auth.uid() AND role = 'student');

-- Students can view assignments for enrolled classes
CREATE POLICY "student_view_assignments" ON assignments
  FOR SELECT USING (
    status = 'published' AND
    class_id IN (
      SELECT class_id FROM student_classes 
      WHERE student_id = auth.uid() AND enrollment_status = 'active'
    )
  );

-- Students can create their own submissions
CREATE POLICY "student_create_submission" ON submissions
  FOR INSERT WITH CHECK (student_id = auth.uid());

-- Students can view and update their own submissions
CREATE POLICY "student_own_submissions" ON submissions
  FOR SELECT USING (student_id = auth.uid());

CREATE POLICY "student_update_own_submission" ON submissions
  FOR UPDATE USING (
    student_id = auth.uid() AND
    graded_at IS NULL -- Can't update after grading
  );

-- Students can view resources for enrolled classes
CREATE POLICY "student_view_resources" ON resources
  FOR SELECT USING (
    (is_public = true OR class_id IN (
      SELECT class_id FROM student_classes 
      WHERE student_id = auth.uid() AND enrollment_status = 'active'
    ))
  );
```

### Database Functions & Triggers

#### Auto-update timestamps
```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables with updated_at
CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_classes_updated_at
  BEFORE UPDATE ON classes
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Repeat for other tables...
```

#### Notification trigger on new assignment
```sql
CREATE OR REPLACE FUNCTION notify_new_assignment()
RETURNS TRIGGER AS $$
BEGIN
  -- Only trigger when status changes to 'published'
  IF NEW.status = 'published' AND (OLD.status IS NULL OR OLD.status != 'published') THEN
    -- Call webhook to n8n
    PERFORM net.http_post(
      url := current_setting('app.n8n_webhook_url'),
      body := jsonb_build_object(
        'event', 'assignment_published',
        'assignment_id', NEW.id,
        'class_id', NEW.class_id,
        'due_date', NEW.due_date
      )
    );
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER assignment_published_trigger
  AFTER INSERT OR UPDATE ON assignments
  FOR EACH ROW EXECUTE FUNCTION notify_new_assignment();
```

#### Increment view count
```sql
CREATE OR REPLACE FUNCTION increment_view_count()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE resources
  SET view_count = view_count + 1
  WHERE id = NEW.resource_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---


## API Design Approach

### API Architecture Principles

**RESTful Design**: Standard HTTP methods (GET, POST, PUT, DELETE)  
**Type Safety**: Full TypeScript support with generated types  
**Error Handling**: Consistent error response format  
**Rate Limiting**: Per-user, per-endpoint limits  
**Versioning**: URL-based versioning (/api/v1/)  
**Documentation**: Auto-generated from TypeScript types

### API Layers

```
┌─────────────────────────────────────────────────────┐
│              Client Application                      │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│         Next.js API Routes (Minimal)                 │
│  • /api/health - Health check                        │
│  • /api/webhooks/* - External webhooks               │
│  • /api/upload - File upload proxy                   │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│         Supabase Client SDK (Primary)                │
│  • Direct database queries with RLS                  │
│  • Real-time subscriptions                           │
│  • Storage operations                                │
│  • Authentication                                    │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│         Supabase Edge Functions (AI/Complex)         │
│  • /ai-chat - Document chat                          │
│  • /ai-quiz-generator - Quiz generation              │
│  • /ai-summarize - Document summarization            │
│  • /ai-study-plan - Study plan generation            │
└─────────────────────────────────────────────────────┘
```

### API Endpoints Specification

#### 1. Authentication APIs

**POST /auth/login**
```typescript
Request:
{
  email: string;
  password: string;
}

Response:
{
  user: {
    id: string;
    email: string;
    role: 'admin' | 'faculty' | 'student';
  };
  session: {
    access_token: string;
    refresh_token: string;
    expires_at: number;
  };
}

Errors:
- 401: Invalid credentials
- 429: Too many login attempts
```

**POST /auth/logout**
```typescript
Request: None (uses session cookie)

Response:
{
  success: boolean;
}
```

#### 2. User Management APIs (Admin Only)

**POST /api/admin/users**
```typescript
Request:
{
  email: string;
  full_name: string;
  role: 'faculty' | 'student';
  department_id?: string;
  metadata?: Record<string, any>;
}

Response:
{
  user: Profile;
  temporary_password: string;
}

Errors:
- 403: Unauthorized (not admin)
- 409: Email already exists
- 422: Validation error
```

**POST /api/admin/users/bulk-import**
```typescript
Request:
{
  users: Array<{
    email: string;
    full_name: string;
    role: 'faculty' | 'student';
    department_id?: string;
  }>;
}

Response:
{
  success_count: number;
  error_count: number;
  errors: Array<{
    row: number;
    email: string;
    error: string;
  }>;
}
```

#### 3. Assignment APIs

**GET /api/assignments**
```typescript
Query Parameters:
{
  class_id?: string;
  status?: 'draft' | 'published' | 'closed';
  due_after?: string; // ISO date
  due_before?: string; // ISO date
  limit?: number;
  offset?: number;
}

Response:
{
  assignments: Assignment[];
  total_count: number;
  has_more: boolean;
}

RLS: Automatically filters based on user role
- Faculty: Own assignments
- Student: Assignments for enrolled classes
```

**POST /api/assignments**
```typescript
Request:
{
  class_id: string;
  title: string;
  description?: string;
  instructions?: string;
  due_date: string; // ISO date
  max_score?: number;
  file_attachments?: string[];
  status?: 'draft' | 'published';
}

Response:
{
  assignment: Assignment;
}

Errors:
- 403: Not authorized for this class
- 422: Validation error (e.g., due_date in past)
```

**POST /api/assignments/:id/submit**
```typescript
Request:
{
  content?: string;
  file_attachments?: string[];
  submission_link?: string;
}

Response:
{
  submission: Submission;
}

Errors:
- 403: Not enrolled in class
- 409: Already submitted
- 422: Past due date (if not allowed)
```

**PUT /api/submissions/:id/grade**
```typescript
Request:
{
  score: number;
  feedback?: string;
}

Response:
{
  submission: Submission;
}

Errors:
- 403: Not the assignment creator
- 422: Score exceeds max_score
```

#### 4. Resource APIs

**GET /api/resources**
```typescript
Query Parameters:
{
  class_id?: string;
  search?: string;
  tags?: string[];
  file_type?: string;
  limit?: number;
  offset?: number;
}

Response:
{
  resources: Resource[];
  total_count: number;
}
```

**POST /api/resources**
```typescript
Request:
{
  class_id: string;
  title: string;
  description?: string;
  file_path: string; // From storage upload
  file_type: string;
  file_size: number;
  tags?: string[];
  is_public?: boolean;
}

Response:
{
  resource: Resource;
}
```

**POST /api/upload**
```typescript
Request: multipart/form-data
{
  file: File;
  bucket: 'documents' | 'submissions' | 'media';
  path?: string;
}

Response:
{
  file_path: string;
  file_url: string;
  file_size: number;
}

Errors:
- 413: File too large
- 415: Unsupported file type
```

#### 5. AI Service APIs (Edge Functions)

**POST /functions/v1/ai-chat**
```typescript
Request:
{
  query: string;
  document_id?: string;
  context?: string;
  conversation_history?: Array<{
    role: 'user' | 'assistant';
    content: string;
  }>;
}

Response:
{
  answer: string;
  cached: boolean;
  tokens_used: number;
  sources?: Array<{
    resource_id: string;
    title: string;
    relevance: number;
  }>;
}

Rate Limit: 10 requests/hour (students), 50/hour (faculty)

Errors:
- 429: Rate limit exceeded
- 503: AI service unavailable
```

**POST /functions/v1/ai-quiz-generator**
```typescript
Request:
{
  content: string;
  question_count: number;
  difficulty: 'easy' | 'medium' | 'hard';
  question_types?: Array<'mcq' | 'short_answer' | 'true_false'>;
}

Response:
{
  quiz_id: string;
  questions: Array<{
    id: string;
    question: string;
    type: string;
    options?: string[];
    correct_answer: string;
    explanation: string;
    difficulty: string;
  }>;
}

Rate Limit: 50 requests/hour (faculty only)
```

**POST /functions/v1/ai-summarize**
```typescript
Request:
{
  document_id: string;
  max_length?: number;
  style?: 'concise' | 'detailed' | 'bullet-points';
}

Response:
{
  summary: string;
  key_points: string[];
  word_count: number;
  cached: boolean;
}
```

**POST /functions/v1/ai-study-plan**
```typescript
Request:
{
  subjects: string[];
  exam_date: string; // ISO date
  hours_per_day: number;
  weak_areas?: string[];
}

Response:
{
  plan_id: string;
  plan: {
    days: Array<{
      date: string;
      subjects: Array<{
        name: string;
        topics: string[];
        hours: number;
        techniques: string[];
      }>;
    }>;
  };
  total_hours: number;
}
```

#### 6. Analytics APIs

**GET /api/analytics/dashboard**
```typescript
Query Parameters:
{
  date_from?: string;
  date_to?: string;
  granularity?: 'daily' | 'weekly' | 'monthly';
}

Response (Admin):
{
  user_stats: {
    total_users: number;
    active_users: number;
    new_users: number;
  };
  engagement_stats: {
    avg_session_duration: number;
    avg_logins_per_user: number;
    feature_usage: Record<string, number>;
  };
  academic_stats: {
    total_assignments: number;
    submission_rate: number;
    avg_grade: number;
  };
}

Response (Faculty):
{
  class_stats: Array<{
    class_id: string;
    class_name: string;
    student_count: number;
    assignment_count: number;
    avg_submission_rate: number;
    avg_grade: number;
  }>;
}

Response (Student):
{
  performance: {
    avg_grade: number;
    submission_rate: number;
    assignments_pending: number;
  };
  engagement: {
    resources_accessed: number;
    ai_queries: number;
    forum_participation: number;
  };
}
```

### API Error Response Format

```typescript
interface APIError {
  error: {
    code: string;
    message: string;
    details?: Record<string, any>;
    timestamp: string;
    request_id: string;
  };
}

// Example error responses
{
  error: {
    code: "VALIDATION_ERROR",
    message: "Invalid input data",
    details: {
      fields: {
        email: "Invalid email format",
        due_date: "Must be in the future"
      }
    },
    timestamp: "2026-02-14T10:30:00Z",
    request_id: "req_abc123"
  }
}

{
  error: {
    code: "RATE_LIMIT_EXCEEDED",
    message: "Too many requests",
    details: {
      limit: 10,
      window: "1 hour",
      retry_after: 3600
    },
    timestamp: "2026-02-14T10:30:00Z",
    request_id: "req_xyz789"
  }
}
```

### API Rate Limiting Strategy

```typescript
// Rate limits by user role and endpoint
const RATE_LIMITS = {
  student: {
    'ai-chat': { requests: 10, window: '1h' },
    'ai-summarize': { requests: 5, window: '1h' },
    'upload': { requests: 20, window: '1h' },
    'default': { requests: 100, window: '1h' }
  },
  faculty: {
    'ai-chat': { requests: 50, window: '1h' },
    'ai-quiz-generator': { requests: 50, window: '1h' },
    'ai-summarize': { requests: 30, window: '1h' },
    'upload': { requests: 100, window: '1h' },
    'default': { requests: 500, window: '1h' }
  },
  admin: {
    'default': { requests: 1000, window: '1h' }
  }
};

// Implementation using Supabase
async function checkRateLimit(userId: string, endpoint: string) {
  const user = await getUser(userId);
  const limit = RATE_LIMITS[user.role][endpoint] || RATE_LIMITS[user.role].default;
  
  const { count } = await supabase
    .from('api_rate_limits')
    .select('*', { count: 'exact', head: true })
    .eq('user_id', userId)
    .eq('endpoint', endpoint)
    .gte('created_at', new Date(Date.now() - parseWindow(limit.window)));
  
  if (count >= limit.requests) {
    throw new RateLimitError(limit);
  }
  
  // Log request
  await supabase.from('api_rate_limits').insert({
    user_id: userId,
    endpoint: endpoint
  });
}
```

---


## Security Architecture

### Security Layers

```
┌─────────────────────────────────────────────────────────┐
│                    Layer 1: Network                      │
│  • HTTPS/TLS 1.3 encryption                             │
│  • DDoS protection (Vercel)                             │
│  • WAF rules                                            │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                Layer 2: Application                      │
│  • CORS policies                                        │
│  • CSP headers                                          │
│  • Rate limiting                                        │
│  • Input validation                                     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              Layer 3: Authentication                     │
│  • JWT-based auth                                       │
│  • Secure session management                            │
│  • Password hashing (bcrypt)                            │
│  • MFA support                                          │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              Layer 4: Authorization                      │
│  • Role-based access control (RBAC)                     │
│  • Row-level security (RLS)                             │
│  • Resource ownership checks                            │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                 Layer 5: Data                            │
│  • Encryption at rest (AES-256)                         │
│  • Encryption in transit (TLS)                          │
│  • Secure key management                                │
│  • Data masking for PII                                 │
└─────────────────────────────────────────────────────────┘
```

### Authentication Implementation

#### JWT Token Structure
```typescript
interface JWTPayload {
  sub: string; // user_id
  email: string;
  role: 'admin' | 'faculty' | 'student';
  institution_id: string;
  iat: number; // issued at
  exp: number; // expiration (24 hours)
}

// Token stored in httpOnly cookie
// Refresh token stored separately with 7-day expiration
```

#### Session Management
```typescript
// Middleware authentication check
export async function authMiddleware(req: NextRequest) {
  const supabase = createMiddlewareClient({ req, res });
  
  // 1. Get session from cookie
  const { data: { session }, error } = await supabase.auth.getSession();
  
  if (error || !session) {
    return NextResponse.redirect('/login');
  }
  
  // 2. Check session expiration
  if (session.expires_at < Date.now() / 1000) {
    // Attempt refresh
    const { data: refreshed } = await supabase.auth.refreshSession();
    if (!refreshed.session) {
      return NextResponse.redirect('/login');
    }
  }
  
  // 3. Verify user is active
  const { data: profile } = await supabase
    .from('profiles')
    .select('is_active')
    .eq('id', session.user.id)
    .single();
  
  if (!profile?.is_active) {
    return NextResponse.redirect('/account-disabled');
  }
  
  // 4. Update last_login_at
  await supabase
    .from('profiles')
    .update({ last_login_at: new Date().toISOString() })
    .eq('id', session.user.id);
  
  return NextResponse.next();
}
```

#### Password Security
```typescript
// Password requirements
const PASSWORD_POLICY = {
  minLength: 8,
  requireUppercase: true,
  requireLowercase: true,
  requireNumbers: true,
  requireSpecialChars: false,
  preventCommonPasswords: true,
  preventUserInfo: true // email, name
};

// Password hashing (handled by Supabase Auth)
// Uses bcrypt with automatic salt generation
// Cost factor: 10 (2^10 iterations)

// Password reset flow
async function initiatePasswordReset(email: string) {
  const { error } = await supabase.auth.resetPasswordForEmail(email, {
    redirectTo: `${APP_URL}/auth/reset-password`
  });
  
  // Always return success to prevent email enumeration
  return { success: true };
}
```

### Authorization Implementation

#### Role-Based Access Control (RBAC)
```typescript
// Permission matrix
const PERMISSIONS = {
  admin: {
    users: ['create', 'read', 'update', 'delete'],
    institutions: ['create', 'read', 'update', 'delete'],
    classes: ['create', 'read', 'update', 'delete'],
    assignments: ['create', 'read', 'update', 'delete'],
    analytics: ['read']
  },
  faculty: {
    users: ['read'], // Only students in their classes
    classes: ['read', 'update'], // Only their classes
    assignments: ['create', 'read', 'update', 'delete'], // Only their assignments
    submissions: ['read', 'update'], // Grade submissions
    resources: ['create', 'read', 'update', 'delete']
  },
  student: {
    assignments: ['read'], // Only for enrolled classes
    submissions: ['create', 'read', 'update'], // Only own submissions
    resources: ['read'], // Only for enrolled classes
    forum: ['create', 'read', 'update'] // Own posts
  }
};

// Permission check helper
function hasPermission(
  userRole: string,
  resource: string,
  action: string
): boolean {
  return PERMISSIONS[userRole]?.[resource]?.includes(action) ?? false;
}

// Middleware permission check
export function requirePermission(resource: string, action: string) {
  return async (req: NextRequest) => {
    const session = await getSession(req);
    
    if (!hasPermission(session.user.role, resource, action)) {
      return new Response('Forbidden', { status: 403 });
    }
    
    return NextResponse.next();
  };
}
```

#### Row-Level Security (RLS) Examples
```sql
-- Students can only see their own submissions
CREATE POLICY "student_own_submissions" ON submissions
  FOR SELECT USING (
    student_id = auth.uid()
  );

-- Faculty can see submissions for their assignments
CREATE POLICY "faculty_assignment_submissions" ON submissions
  FOR SELECT USING (
    assignment_id IN (
      SELECT id FROM assignments WHERE faculty_id = auth.uid()
    )
  );

-- Students can only access resources for enrolled classes
CREATE POLICY "student_enrolled_resources" ON resources
  FOR SELECT USING (
    class_id IN (
      SELECT class_id FROM student_classes 
      WHERE student_id = auth.uid() 
      AND enrollment_status = 'active'
    )
    OR is_public = true
  );

-- Prevent students from updating graded submissions
CREATE POLICY "student_update_ungraded" ON submissions
  FOR UPDATE USING (
    student_id = auth.uid() AND graded_at IS NULL
  );
```

### Data Protection

#### Encryption Strategy
```typescript
// Sensitive fields encryption (at application level)
import { createCipheriv, createDecipheriv, randomBytes } from 'crypto';

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY; // 32 bytes
const ALGORITHM = 'aes-256-gcm';

function encrypt(text: string): string {
  const iv = randomBytes(16);
  const cipher = createCipheriv(ALGORITHM, ENCRYPTION_KEY, iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag();
  
  return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
}

function decrypt(encryptedText: string): string {
  const [ivHex, authTagHex, encrypted] = encryptedText.split(':');
  
  const iv = Buffer.from(ivHex, 'hex');
  const authTag = Buffer.from(authTagHex, 'hex');
  const decipher = createDecipheriv(ALGORITHM, ENCRYPTION_KEY, iv);
  
  decipher.setAuthTag(authTag);
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}

// Usage for sensitive data
const sensitiveData = {
  phone: encrypt(user.phone),
  address: encrypt(user.address)
};
```

#### PII Handling
```typescript
// PII fields identification
const PII_FIELDS = [
  'email',
  'phone',
  'address',
  'date_of_birth',
  'government_id'
];

// Data masking for logs
function maskPII(data: any): any {
  const masked = { ...data };
  
  for (const field of PII_FIELDS) {
    if (masked[field]) {
      masked[field] = '***REDACTED***';
    }
  }
  
  return masked;
}

// Audit logging with PII masking
async function logActivity(activity: ActivityLog) {
  const maskedMetadata = maskPII(activity.metadata);
  
  await supabase.from('activity_logs').insert({
    ...activity,
    metadata: maskedMetadata
  });
}
```

#### Secure File Storage
```typescript
// File upload with security checks
async function secureFileUpload(file: File, userId: string) {
  // 1. Validate file type
  const allowedTypes = [
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'image/jpeg',
    'image/png'
  ];
  
  if (!allowedTypes.includes(file.type)) {
    throw new Error('File type not allowed');
  }
  
  // 2. Validate file size (100MB max)
  if (file.size > 100 * 1024 * 1024) {
    throw new Error('File too large');
  }
  
  // 3. Scan for malware (if available)
  // await scanFile(file);
  
  // 4. Generate secure filename
  const ext = file.name.split('.').pop();
  const filename = `${userId}/${randomBytes(16).toString('hex')}.${ext}`;
  
  // 5. Upload to Supabase Storage
  const { data, error } = await supabase.storage
    .from('documents')
    .upload(filename, file, {
      cacheControl: '3600',
      upsert: false
    });
  
  if (error) throw error;
  
  // 6. Generate signed URL (expires in 1 hour)
  const { data: signedUrl } = await supabase.storage
    .from('documents')
    .createSignedUrl(filename, 3600);
  
  return {
    path: filename,
    url: signedUrl.signedUrl
  };
}
```

### API Security

#### Input Validation
```typescript
import { z } from 'zod';

// Schema validation with Zod
const CreateAssignmentSchema = z.object({
  class_id: z.string().uuid(),
  title: z.string().min(1).max(200),
  description: z.string().max(5000).optional(),
  due_date: z.string().datetime().refine(
    (date) => new Date(date) > new Date(),
    'Due date must be in the future'
  ),
  max_score: z.number().int().min(1).max(1000).default(100),
  file_attachments: z.array(z.string().url()).max(10).optional()
});

// API route with validation
export async function POST(req: Request) {
  try {
    const body = await req.json();
    
    // Validate input
    const validated = CreateAssignmentSchema.parse(body);
    
    // Process request
    const assignment = await createAssignment(validated);
    
    return Response.json(assignment);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return Response.json(
        { error: 'Validation error', details: error.errors },
        { status: 422 }
      );
    }
    throw error;
  }
}
```

#### SQL Injection Prevention
```typescript
// Supabase automatically prevents SQL injection through parameterized queries
// SAFE - Uses parameterized query
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userInput); // Automatically escaped

// UNSAFE - Never use raw SQL with user input
// const { data } = await supabase.rpc('raw_query', {
//   query: `SELECT * FROM users WHERE email = '${userInput}'`
// });
```

#### XSS Prevention
```typescript
// Content Security Policy headers
export function securityHeaders() {
  return {
    'Content-Security-Policy': [
      "default-src 'self'",
      "script-src 'self' 'unsafe-inline' 'unsafe-eval'", // Next.js requires unsafe-inline
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: https:",
      "font-src 'self' data:",
      "connect-src 'self' https://*.supabase.co",
      "frame-ancestors 'none'"
    ].join('; '),
    'X-Frame-Options': 'DENY',
    'X-Content-Type-Options': 'nosniff',
    'Referrer-Policy': 'strict-origin-when-cross-origin',
    'Permissions-Policy': 'camera=(), microphone=(), geolocation=()'
  };
}

// Sanitize user input before rendering
import DOMPurify from 'isomorphic-dompurify';

function sanitizeHTML(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href']
  });
}
```

#### CSRF Protection
```typescript
// Next.js automatically provides CSRF protection for API routes
// through SameSite cookies and origin checking

// Additional CSRF token for sensitive operations
import { randomBytes } from 'crypto';

async function generateCSRFToken(userId: string): Promise<string> {
  const token = randomBytes(32).toString('hex');
  
  // Store in database with expiration
  await supabase.from('csrf_tokens').insert({
    user_id: userId,
    token: token,
    expires_at: new Date(Date.now() + 3600000) // 1 hour
  });
  
  return token;
}

async function validateCSRFToken(userId: string, token: string): Promise<boolean> {
  const { data } = await supabase
    .from('csrf_tokens')
    .select('*')
    .eq('user_id', userId)
    .eq('token', token)
    .gt('expires_at', new Date().toISOString())
    .single();
  
  if (data) {
    // Delete token after use (one-time use)
    await supabase.from('csrf_tokens').delete().eq('id', data.id);
    return true;
  }
  
  return false;
}
```

### Secrets Management

```typescript
// Environment variables structure
// .env.local (never commit)
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJxxx... // Safe for frontend

// Server-only secrets
SUPABASE_SERVICE_ROLE_KEY=eyJxxx... // Never expose to frontend
GEMINI_API_KEY=AIzxxx...
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx
N8N_WEBHOOK_URL=https://xxx.n8n.cloud/webhook/xxx
N8N_API_KEY=xxx
ENCRYPTION_KEY=xxx // 32 bytes for AES-256

// Database connection (Supabase handles this)
DATABASE_URL=postgresql://xxx

// Accessing secrets safely
// Frontend (public)
const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY;

// Backend only (server components, API routes, edge functions)
const serviceRoleKey = process.env.SUPABASE_SERVICE_ROLE_KEY;
const geminiKey = process.env.GEMINI_API_KEY;

// Validation on startup
function validateEnvironment() {
  const required = [
    'NEXT_PUBLIC_SUPABASE_URL',
    'NEXT_PUBLIC_SUPABASE_ANON_KEY',
    'SUPABASE_SERVICE_ROLE_KEY',
    'GEMINI_API_KEY'
  ];
  
  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}
```

### Security Monitoring & Incident Response

```typescript
// Security event logging
interface SecurityEvent {
  type: 'failed_login' | 'unauthorized_access' | 'suspicious_activity' | 'data_breach';
  severity: 'low' | 'medium' | 'high' | 'critical';
  user_id?: string;
  ip_address: string;
  details: Record<string, any>;
}

async function logSecurityEvent(event: SecurityEvent) {
  // Log to database
  await supabase.from('security_events').insert(event);
  
  // Alert on critical events
  if (event.severity === 'critical') {
    await sendAlertToAdmins(event);
  }
  
  // Trigger automated response
  if (event.type === 'failed_login') {
    await handleFailedLogin(event);
  }
}

// Failed login handling
async function handleFailedLogin(event: SecurityEvent) {
  const { count } = await supabase
    .from('security_events')
    .select('*', { count: 'exact', head: true })
    .eq('type', 'failed_login')
    .eq('ip_address', event.ip_address)
    .gte('created_at', new Date(Date.now() - 3600000).toISOString());
  
  // Lock account after 5 failed attempts
  if (count >= 5) {
    await supabase
      .from('profiles')
      .update({ is_active: false, locked_reason: 'Too many failed login attempts' })
      .eq('email', event.details.email);
    
    await sendAccountLockedEmail(event.details.email);
  }
}
```

---


## Scalability Considerations

### Horizontal Scaling Strategy

```
Current (MVP): Single Region
┌─────────────────────────────────┐
│  Vercel (Auto-scaling)          │
│  └─ Next.js Instances (N)       │
└─────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│  Supabase (Single Instance)     │
│  └─ PostgreSQL + Storage        │
└─────────────────────────────────┘

Future (Scale): Multi-Region
┌──────────────────┐  ┌──────────────────┐
│  Vercel US-East  │  │  Vercel US-West  │
│  └─ Instances    │  │  └─ Instances    │
└────────┬─────────┘  └────────┬─────────┘
         │                     │
         └──────────┬──────────┘
                    ▼
         ┌─────────────────────┐
         │  Supabase Primary   │
         │  └─ Read Replicas   │
         └─────────────────────┘
```

### Database Scaling

#### Connection Pooling
```typescript
// Supabase connection pooling configuration
const supabaseConfig = {
  db: {
    pooler_mode: 'transaction', // or 'session'
    pool_size: 15, // Default for free tier
    max_client_conn: 100,
    default_pool_size: 20
  }
};

// Use connection pooler for serverless functions
const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY,
  {
    db: {
      schema: 'public'
    },
    global: {
      headers: {
        'x-connection-pooler': 'true'
      }
    }
  }
);
```

#### Query Optimization
```sql
-- Indexes for common queries
CREATE INDEX CONCURRENTLY idx_assignments_class_due 
  ON assignments(class_id, due_date) 
  WHERE status = 'published';

CREATE INDEX CONCURRENTLY idx_submissions_student_assignment 
  ON submissions(student_id, assignment_id);

CREATE INDEX CONCURRENTLY idx_resources_class_created 
  ON resources(class_id, created_at DESC);

-- Partial indexes for active records
CREATE INDEX CONCURRENTLY idx_profiles_active 
  ON profiles(role, institution_id) 
  WHERE is_active = true;

-- Composite indexes for complex queries
CREATE INDEX CONCURRENTLY idx_student_classes_lookup 
  ON student_classes(student_id, class_id, enrollment_status);

-- Full-text search indexes
CREATE INDEX CONCURRENTLY idx_resources_fts 
  ON resources USING GIN(to_tsvector('english', title || ' ' || COALESCE(description, '')));

-- JSONB indexes for metadata queries
CREATE INDEX CONCURRENTLY idx_profiles_metadata 
  ON profiles USING GIN(metadata);
```

#### Database Partitioning (Future)
```sql
-- Partition activity_logs by month for better performance
CREATE TABLE activity_logs (
  id UUID DEFAULT gen_random_uuid(),
  user_id UUID,
  action TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  ...
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE activity_logs_2026_02 
  PARTITION OF activity_logs
  FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');

CREATE TABLE activity_logs_2026_03 
  PARTITION OF activity_logs
  FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');

-- Automatic partition creation via cron job
CREATE OR REPLACE FUNCTION create_monthly_partition()
RETURNS void AS $$
DECLARE
  partition_date DATE;
  partition_name TEXT;
  start_date TEXT;
  end_date TEXT;
BEGIN
  partition_date := DATE_TRUNC('month', CURRENT_DATE + INTERVAL '1 month');
  partition_name := 'activity_logs_' || TO_CHAR(partition_date, 'YYYY_MM');
  start_date := partition_date::TEXT;
  end_date := (partition_date + INTERVAL '1 month')::TEXT;
  
  EXECUTE format(
    'CREATE TABLE IF NOT EXISTS %I PARTITION OF activity_logs FOR VALUES FROM (%L) TO (%L)',
    partition_name, start_date, end_date
  );
END;
$$ LANGUAGE plpgsql;
```

### Caching Strategy

#### Multi-Level Caching
```typescript
// 1. Browser Cache (Static Assets)
// Configured in next.config.js
const nextConfig = {
  async headers() {
    return [
      {
        source: '/static/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable'
          }
        ]
      }
    ];
  }
};

// 2. CDN Cache (Vercel Edge Network)
// Automatic for static pages and API routes with cache headers
export async function GET(req: Request) {
  return Response.json(data, {
    headers: {
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400'
    }
  });
}

// 3. Application Cache (AI Responses)
class CacheManager {
  async get(key: string): Promise<any> {
    // Check database cache
    const { data } = await supabase
      .from('ai_cache')
      .select('response_data')
      .eq('cache_key', key)
      .gt('expires_at', new Date().toISOString())
      .single();
    
    if (data) {
      // Update hit count and last accessed
      await supabase
        .from('ai_cache')
        .update({
          hit_count: supabase.sql`hit_count + 1`,
          last_accessed_at: new Date().toISOString()
        })
        .eq('cache_key', key);
      
      return data.response_data;
    }
    
    return null;
  }
  
  async set(key: string, value: any, ttl: number): Promise<void> {
    const expiresAt = new Date(Date.now() + ttl * 1000);
    
    await supabase.from('ai_cache').upsert({
      cache_key: key,
      response_data: value,
      expires_at: expiresAt.toISOString()
    });
  }
  
  async invalidate(pattern: string): Promise<void> {
    // Invalidate cache entries matching pattern
    await supabase
      .from('ai_cache')
      .delete()
      .like('cache_key', `${pattern}%`);
  }
}

// 4. Query Result Cache (React Query)
// Configured in app layout
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      refetchOnWindowFocus: false,
      retry: 1
    }
  }
});
```

#### Cache Invalidation Strategy
```typescript
// Event-based cache invalidation
async function onAssignmentCreated(assignment: Assignment) {
  // Invalidate relevant caches
  await Promise.all([
    cache.invalidate(`assignments:class:${assignment.class_id}`),
    cache.invalidate(`dashboard:faculty:${assignment.faculty_id}`),
    // Invalidate for all enrolled students
    invalidateStudentCaches(assignment.class_id)
  ]);
}

async function invalidateStudentCaches(classId: string) {
  const { data: students } = await supabase
    .from('student_classes')
    .select('student_id')
    .eq('class_id', classId)
    .eq('enrollment_status', 'active');
  
  await Promise.all(
    students.map(s => cache.invalidate(`dashboard:student:${s.student_id}`))
  );
}

// Time-based cache cleanup (scheduled job)
async function cleanupExpiredCache() {
  await supabase
    .from('ai_cache')
    .delete()
    .lt('expires_at', new Date().toISOString());
}
```

### Load Balancing & Auto-Scaling

#### Vercel Auto-Scaling
```typescript
// Vercel automatically scales based on:
// - Request volume
// - Response time
// - Error rate

// Configuration in vercel.json
{
  "functions": {
    "app/**/*.ts": {
      "maxDuration": 10,
      "memory": 1024
    },
    "api/**/*.ts": {
      "maxDuration": 30,
      "memory": 3008
    }
  },
  "regions": ["iad1"], // US East (MVP)
  // Future: ["iad1", "sfo1", "lhr1"] for multi-region
}
```

#### Rate Limiting & Throttling
```typescript
// Implement rate limiting at multiple levels
class RateLimiter {
  // 1. Per-user rate limiting
  async checkUserLimit(userId: string, action: string): Promise<boolean> {
    const limit = this.getLimitForAction(action);
    const key = `ratelimit:user:${userId}:${action}`;
    
    const { count } = await supabase
      .from('rate_limit_tracking')
      .select('*', { count: 'exact', head: true })
      .eq('key', key)
      .gte('created_at', new Date(Date.now() - limit.window));
    
    return count < limit.max;
  }
  
  // 2. Per-IP rate limiting (DDoS protection)
  async checkIPLimit(ipAddress: string): Promise<boolean> {
    const key = `ratelimit:ip:${ipAddress}`;
    const limit = { max: 100, window: 60000 }; // 100 req/min
    
    // Similar implementation
    return true;
  }
  
  // 3. Global rate limiting (system protection)
  async checkGlobalLimit(): Promise<boolean> {
    // Monitor system-wide request rate
    // Implement circuit breaker if needed
    return true;
  }
}
```

### Storage Scaling

#### File Storage Strategy
```typescript
// Supabase Storage with CDN
const storageConfig = {
  buckets: {
    documents: {
      public: false,
      fileSizeLimit: 100 * 1024 * 1024, // 100MB
      allowedMimeTypes: [
        'application/pdf',
        'application/msword',
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
        'image/jpeg',
        'image/png'
      ]
    },
    submissions: {
      public: false,
      fileSizeLimit: 50 * 1024 * 1024 // 50MB
    },
    media: {
      public: true,
      fileSizeLimit: 10 * 1024 * 1024 // 10MB
    }
  }
};

// Implement file compression for large uploads
async function uploadWithCompression(file: File) {
  let processedFile = file;
  
  // Compress images
  if (file.type.startsWith('image/')) {
    processedFile = await compressImage(file, {
      maxWidth: 1920,
      maxHeight: 1080,
      quality: 0.8
    });
  }
  
  // Compress PDFs (if needed)
  if (file.type === 'application/pdf' && file.size > 10 * 1024 * 1024) {
    processedFile = await compressPDF(file);
  }
  
  return uploadFile(processedFile);
}

// Implement chunked uploads for large files
async function chunkedUpload(file: File, chunkSize = 5 * 1024 * 1024) {
  const chunks = Math.ceil(file.size / chunkSize);
  const uploadId = generateUploadId();
  
  for (let i = 0; i < chunks; i++) {
    const start = i * chunkSize;
    const end = Math.min(start + chunkSize, file.size);
    const chunk = file.slice(start, end);
    
    await uploadChunk(uploadId, i, chunk);
  }
  
  return finalizeUpload(uploadId);
}
```

#### Storage Cleanup Strategy
```typescript
// Scheduled cleanup of orphaned files
async function cleanupOrphanedFiles() {
  // 1. Find files in storage not referenced in database
  const { data: storageFiles } = await supabase.storage
    .from('documents')
    .list();
  
  const { data: dbFiles } = await supabase
    .from('resources')
    .select('file_path');
  
  const dbFilePaths = new Set(dbFiles.map(f => f.file_path));
  const orphanedFiles = storageFiles.filter(f => !dbFilePaths.has(f.name));
  
  // 2. Delete orphaned files older than 7 days
  const sevenDaysAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
  
  for (const file of orphanedFiles) {
    if (new Date(file.created_at) < sevenDaysAgo) {
      await supabase.storage.from('documents').remove([file.name]);
    }
  }
}

// Archive old submissions
async function archiveOldSubmissions() {
  // Move submissions older than 1 year to archive bucket
  const oneYearAgo = new Date();
  oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);
  
  const { data: oldSubmissions } = await supabase
    .from('submissions')
    .select('id, file_attachments')
    .lt('submitted_at', oneYearAgo.toISOString());
  
  for (const submission of oldSubmissions) {
    for (const filePath of submission.file_attachments) {
      await moveToArchive(filePath);
    }
  }
}
```

### Performance Optimization

#### Database Query Optimization
```typescript
// Use select() to fetch only needed columns
// BAD: Fetches all columns
const { data } = await supabase.from('profiles').select('*');

// GOOD: Fetches only needed columns
const { data } = await supabase
  .from('profiles')
  .select('id, full_name, email, role');

// Use pagination for large result sets
async function getPaginatedAssignments(page: number, pageSize: number = 20) {
  const from = page * pageSize;
  const to = from + pageSize - 1;
  
  const { data, count } = await supabase
    .from('assignments')
    .select('*', { count: 'exact' })
    .range(from, to)
    .order('created_at', { ascending: false });
  
  return {
    data,
    total: count,
    page,
    pageSize,
    totalPages: Math.ceil(count / pageSize)
  };
}

// Use database functions for complex operations
CREATE OR REPLACE FUNCTION get_student_dashboard(student_id UUID)
RETURNS JSON AS $$
DECLARE
  result JSON;
BEGIN
  SELECT json_build_object(
    'assignments', (
      SELECT json_agg(a.*)
      FROM assignments a
      JOIN student_classes sc ON sc.class_id = a.class_id
      WHERE sc.student_id = student_id
      AND a.status = 'published'
      ORDER BY a.due_date
      LIMIT 5
    ),
    'recent_resources', (
      SELECT json_agg(r.*)
      FROM resources r
      JOIN student_classes sc ON sc.class_id = r.class_id
      WHERE sc.student_id = student_id
      ORDER BY r.created_at DESC
      LIMIT 5
    ),
    'notifications', (
      SELECT json_agg(n.*)
      FROM notifications n
      WHERE n.user_id = student_id
      AND n.is_read = false
      ORDER BY n.created_at DESC
      LIMIT 10
    )
  ) INTO result;
  
  RETURN result;
END;
$$ LANGUAGE plpgsql;

// Call from application
const { data } = await supabase.rpc('get_student_dashboard', {
  student_id: userId
});
```

#### Frontend Performance
```typescript
// Code splitting and lazy loading
import dynamic from 'next/dynamic';

const AIAssistant = dynamic(() => import('@/components/AIAssistant'), {
  loading: () => <Skeleton />,
  ssr: false // Don't render on server
});

// Image optimization
import Image from 'next/image';

<Image
  src="/profile.jpg"
  alt="Profile"
  width={200}
  height={200}
  loading="lazy"
  placeholder="blur"
/>

// Prefetch critical data
export async function generateMetadata({ params }) {
  // This runs on the server and prefetches data
  const assignment = await getAssignment(params.id);
  return { title: assignment.title };
}

// Use React Server Components for data fetching
export default async function AssignmentPage({ params }) {
  // Fetch data on server
  const assignment = await getAssignment(params.id);
  
  return <AssignmentView assignment={assignment} />;
}
```

### Monitoring & Observability

```typescript
// Performance monitoring
class PerformanceMonitor {
  async trackQuery(queryName: string, duration: number) {
    if (duration > 1000) { // Slow query threshold
      await supabase.from('slow_queries').insert({
        query_name: queryName,
        duration_ms: duration,
        timestamp: new Date().toISOString()
      });
    }
  }
  
  async trackAPICall(endpoint: string, duration: number, status: number) {
    await supabase.from('api_metrics').insert({
      endpoint,
      duration_ms: duration,
      status_code: status,
      timestamp: new Date().toISOString()
    });
  }
}

// Usage
const start = Date.now();
const result = await supabase.from('assignments').select('*');
const duration = Date.now() - start;
await monitor.trackQuery('get_assignments', duration);
```

---


## Deployment Architecture

### Environment Strategy

```
┌─────────────────────────────────────────────────────────┐
│                    Development                           │
│  • Local Next.js dev server                             │
│  • Supabase local instance (optional)                   │
│  • Mock n8n workflows                                   │
│  • Mock AI responses                                    │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                     Staging                              │
│  • Vercel preview deployment                            │
│  • Supabase staging project                             │
│  • n8n staging workflows                                │
│  • Limited AI quota                                     │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    Production                            │
│  • Vercel production deployment                         │
│  • Supabase production project                          │
│  • n8n production workflows                             │
│  • Full AI quota                                        │
└─────────────────────────────────────────────────────────┘
```

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy SparkLink

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run type check
        run: npm run type-check
      
      - name: Run tests
        run: npm run test
        env:
          NEXT_PUBLIC_SUPABASE_URL: ${{ secrets.SUPABASE_URL_TEST }}
          NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY_TEST }}
      
      - name: Build application
        run: npm run build
  
  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel Staging
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          scope: ${{ secrets.VERCEL_ORG_ID }}
  
  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel Production
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
          scope: ${{ secrets.VERCEL_ORG_ID }}
      
      - name: Run database migrations
        run: npx supabase db push
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
          SUPABASE_DB_PASSWORD: ${{ secrets.SUPABASE_DB_PASSWORD }}
      
      - name: Notify deployment
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK_URL }} \
            -H 'Content-Type: application/json' \
            -d '{"text":"SparkLink deployed to production successfully!"}'
```

### Database Migration Strategy

```bash
# Supabase CLI for migrations
# Install: npm install -g supabase

# Initialize Supabase project
supabase init

# Create new migration
supabase migration new create_initial_schema

# Apply migrations locally
supabase db reset

# Push migrations to remote
supabase db push

# Generate TypeScript types from database
supabase gen types typescript --local > src/types/database.ts
```

**Migration File Example**:
```sql
-- supabase/migrations/20260214000001_create_initial_schema.sql

-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Create custom types
CREATE TYPE user_role AS ENUM ('admin', 'faculty', 'student');
CREATE TYPE enrollment_status AS ENUM ('active', 'dropped', 'completed');

-- Create tables
CREATE TABLE institutions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  code TEXT UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ... (rest of schema)

-- Enable RLS
ALTER TABLE institutions ENABLE ROW LEVEL SECURITY;
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "admin_all_access" ON institutions
  FOR ALL USING (
    (SELECT role FROM profiles WHERE id = auth.uid()) = 'admin'
  );

-- Create indexes
CREATE INDEX idx_profiles_role ON profiles(role);
CREATE INDEX idx_profiles_institution ON profiles(institution_id);

-- Create functions
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create triggers
CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### Deployment Configuration

#### Vercel Configuration
```json
// vercel.json
{
  "version": 2,
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1"],
  "env": {
    "NEXT_PUBLIC_SUPABASE_URL": "@supabase-url",
    "NEXT_PUBLIC_SUPABASE_ANON_KEY": "@supabase-anon-key"
  },
  "build": {
    "env": {
      "SUPABASE_SERVICE_ROLE_KEY": "@supabase-service-role-key",
      "GEMINI_API_KEY": "@gemini-api-key",
      "N8N_WEBHOOK_URL": "@n8n-webhook-url"
    }
  },
  "functions": {
    "app/**/*.ts": {
      "maxDuration": 10,
      "memory": 1024
    }
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        }
      ]
    }
  ],
  "redirects": [
    {
      "source": "/",
      "destination": "/login",
      "permanent": false
    }
  ]
}
```

#### Next.js Configuration
```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  
  images: {
    domains: [
      'supabase.co',
      'your-project.supabase.co'
    ],
    formats: ['image/avif', 'image/webp']
  },
  
  experimental: {
    serverActions: true,
    serverComponentsExternalPackages: ['@supabase/supabase-js']
  },
  
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on'
          },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload'
          }
        ]
      }
    ];
  },
  
  async redirects() {
    return [
      {
        source: '/admin',
        destination: '/admin/dashboard',
        permanent: true
      },
      {
        source: '/faculty',
        destination: '/faculty/dashboard',
        permanent: true
      },
      {
        source: '/student',
        destination: '/student/dashboard',
        permanent: true
      }
    ];
  }
};

module.exports = nextConfig;
```

#### Supabase Configuration
```toml
# supabase/config.toml
[api]
enabled = true
port = 54321
schemas = ["public", "storage"]
extra_search_path = ["public"]
max_rows = 1000

[db]
port = 54322
major_version = 15

[studio]
enabled = true
port = 54323

[auth]
enabled = true
site_url = "http://localhost:3000"
additional_redirect_urls = ["https://sparklink.vercel.app"]
jwt_expiry = 86400
enable_signup = false  # Admin-only signup

[auth.email]
enable_signup = false
double_confirm_changes = true
enable_confirmations = true

[storage]
enabled = true
file_size_limit = "100MB"

[functions]
enabled = true
```

### Monitoring & Logging

#### Application Monitoring
```typescript
// lib/monitoring.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  
  beforeSend(event, hint) {
    // Filter sensitive data
    if (event.request) {
      delete event.request.cookies;
      delete event.request.headers?.Authorization;
    }
    return event;
  }
});

// Custom error tracking
export function trackError(error: Error, context?: Record<string, any>) {
  Sentry.captureException(error, {
    extra: context
  });
}

// Performance tracking
export function trackPerformance(name: string, duration: number) {
  Sentry.captureMessage(`Performance: ${name}`, {
    level: 'info',
    extra: { duration }
  });
}
```

#### Logging Strategy
```typescript
// lib/logger.ts
enum LogLevel {
  DEBUG = 'debug',
  INFO = 'info',
  WARN = 'warn',
  ERROR = 'error'
}

class Logger {
  private context: string;
  
  constructor(context: string) {
    this.context = context;
  }
  
  private log(level: LogLevel, message: string, meta?: any) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      context: this.context,
      message,
      meta: this.sanitizeMeta(meta)
    };
    
    // Console logging
    console[level](JSON.stringify(logEntry));
    
    // Send to logging service (production only)
    if (process.env.NODE_ENV === 'production') {
      this.sendToLoggingService(logEntry);
    }
  }
  
  private sanitizeMeta(meta: any): any {
    if (!meta) return meta;
    
    // Remove sensitive fields
    const sanitized = { ...meta };
    const sensitiveFields = ['password', 'token', 'apiKey', 'secret'];
    
    for (const field of sensitiveFields) {
      if (sanitized[field]) {
        sanitized[field] = '***REDACTED***';
      }
    }
    
    return sanitized;
  }
  
  private async sendToLoggingService(entry: any) {
    // Send to Supabase or external logging service
    await supabase.from('application_logs').insert(entry);
  }
  
  debug(message: string, meta?: any) {
    this.log(LogLevel.DEBUG, message, meta);
  }
  
  info(message: string, meta?: any) {
    this.log(LogLevel.INFO, message, meta);
  }
  
  warn(message: string, meta?: any) {
    this.log(LogLevel.WARN, message, meta);
  }
  
  error(message: string, error?: Error, meta?: any) {
    this.log(LogLevel.ERROR, message, {
      ...meta,
      error: {
        message: error?.message,
        stack: error?.stack
      }
    });
  }
}

// Usage
const logger = new Logger('AssignmentService');
logger.info('Assignment created', { assignmentId: '123', classId: '456' });
logger.error('Failed to create assignment', error, { userId: '789' });
```

### Health Checks & Status Page

```typescript
// app/api/health/route.ts
export async function GET() {
  const checks = await Promise.allSettled([
    checkDatabase(),
    checkStorage(),
    checkAIService(),
    checkN8N()
  ]);
  
  const status = {
    status: checks.every(c => c.status === 'fulfilled') ? 'healthy' : 'degraded',
    timestamp: new Date().toISOString(),
    services: {
      database: checks[0].status === 'fulfilled' ? 'up' : 'down',
      storage: checks[1].status === 'fulfilled' ? 'up' : 'down',
      ai: checks[2].status === 'fulfilled' ? 'up' : 'down',
      automation: checks[3].status === 'fulfilled' ? 'up' : 'down'
    }
  };
  
  return Response.json(status, {
    status: status.status === 'healthy' ? 200 : 503
  });
}

async function checkDatabase() {
  const { error } = await supabase.from('profiles').select('id').limit(1);
  if (error) throw error;
}

async function checkStorage() {
  const { data, error } = await supabase.storage.from('documents').list('', {
    limit: 1
  });
  if (error) throw error;
}

async function checkAIService() {
  // Ping AI service or check recent usage
  const { data, error } = await supabase
    .from('ai_usage_logs')
    .select('id')
    .gte('created_at', new Date(Date.now() - 300000).toISOString())
    .limit(1);
  
  // If no recent usage, do a test call
  if (!data || data.length === 0) {
    // Perform lightweight AI test
  }
}

async function checkN8N() {
  const response = await fetch(process.env.N8N_HEALTH_URL);
  if (!response.ok) throw new Error('n8n unhealthy');
}
```

### Backup & Disaster Recovery

```typescript
// Automated backup strategy
// Supabase provides automatic daily backups

// Additional backup script for critical data
async function backupCriticalData() {
  const timestamp = new Date().toISOString().split('T')[0];
  
  // 1. Export user data
  const { data: users } = await supabase
    .from('profiles')
    .select('*');
  
  await saveToBackupStorage(`users_${timestamp}.json`, users);
  
  // 2. Export assignments and submissions
  const { data: assignments } = await supabase
    .from('assignments')
    .select('*, submissions(*)');
  
  await saveToBackupStorage(`assignments_${timestamp}.json`, assignments);
  
  // 3. Export analytics snapshots
  const { data: analytics } = await supabase
    .from('analytics_snapshots')
    .select('*')
    .gte('snapshot_date', new Date(Date.now() - 90 * 24 * 60 * 60 * 1000));
  
  await saveToBackupStorage(`analytics_${timestamp}.json`, analytics);
}

async function saveToBackupStorage(filename: string, data: any) {
  // Save to separate backup bucket or external storage
  await supabase.storage
    .from('backups')
    .upload(filename, JSON.stringify(data), {
      contentType: 'application/json'
    });
}

// Disaster recovery procedure
async function restoreFromBackup(backupDate: string) {
  // 1. Verify backup integrity
  const backup = await loadBackup(backupDate);
  
  // 2. Create restore point
  await createRestorePoint();
  
  // 3. Restore data
  await restoreUsers(backup.users);
  await restoreAssignments(backup.assignments);
  await restoreAnalytics(backup.analytics);
  
  // 4. Verify restoration
  await verifyDataIntegrity();
  
  // 5. Notify admins
  await notifyAdmins('Backup restored successfully');
}
```

---


## Future Extensibility

### Modular Architecture for Growth

```
Current Architecture (MVP)
┌─────────────────────────────────────┐
│         Monolithic Next.js          │
│  • All portals in one app           │
│  • Shared components                │
│  • Single deployment                │
└─────────────────────────────────────┘

Future Architecture (Scale)
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Admin Portal │  │Faculty Portal│  │Student Portal│
│  (Next.js)   │  │  (Next.js)   │  │  (Next.js)   │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                  │
       └─────────────────┼──────────────────┘
                         │
                         ▼
              ┌──────────────────┐
              │   API Gateway    │
              │  (GraphQL/REST)  │
              └──────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │  User    │   │ Content  │   │   AI     │
  │ Service  │   │ Service  │   │ Service  │
  └──────────┘   └──────────┘   └──────────┘
```

### Phase 1: Current MVP Architecture

**Characteristics**:
- Monolithic Next.js application
- All features in single codebase
- Direct Supabase integration
- Simple deployment pipeline
- Suitable for 0-5K users

**Advantages**:
- Fast development
- Easy debugging
- Simple deployment
- Low operational overhead

**Limitations**:
- Scaling challenges beyond 10K users
- Difficult to have independent team ownership
- All features deployed together

---

### Phase 2: Modular Monolith (6-12 months)

**Architecture Evolution**:
```typescript
// Organize code by domain modules
src/
├── modules/
│   ├── auth/
│   │   ├── components/
│   │   ├── services/
│   │   ├── hooks/
│   │   └── types/
│   ├── assignments/
│   │   ├── components/
│   │   ├── services/
│   │   ├── hooks/
│   │   └── types/
│   ├── ai-assistant/
│   │   ├── components/
│   │   ├── services/
│   │   ├── hooks/
│   │   └── types/
│   └── analytics/
│       ├── components/
│       ├── services/
│       ├── hooks/
│       └── types/
└── shared/
    ├── components/
    ├── utils/
    └── types/
```

**Module Interface Pattern**:
```typescript
// modules/assignments/index.ts
export interface AssignmentModule {
  // Components
  AssignmentList: React.ComponentType<AssignmentListProps>;
  AssignmentForm: React.ComponentType<AssignmentFormProps>;
  
  // Services
  assignmentService: {
    create: (data: CreateAssignmentDTO) => Promise<Assignment>;
    update: (id: string, data: UpdateAssignmentDTO) => Promise<Assignment>;
    delete: (id: string) => Promise<void>;
    list: (filters: AssignmentFilters) => Promise<Assignment[]>;
  };
  
  // Hooks
  useAssignments: (classId: string) => UseAssignmentsResult;
  useAssignment: (id: string) => UseAssignmentResult;
}

// Clear module boundaries
export const assignmentModule: AssignmentModule = {
  AssignmentList,
  AssignmentForm,
  assignmentService,
  useAssignments,
  useAssignment
};
```

**Benefits**:
- Clear module boundaries
- Independent testing
- Easier team collaboration
- Preparation for microservices

---

### Phase 3: Microservices Architecture (12-24 months)

**Service Decomposition**:

#### 1. User Service
```typescript
// Responsibilities
- User authentication and authorization
- Profile management
- Role and permission management
- Session management

// API Endpoints
POST   /api/users
GET    /api/users/:id
PUT    /api/users/:id
DELETE /api/users/:id
POST   /api/users/bulk-import
GET    /api/users/:id/permissions

// Database
- profiles
- institutions
- departments
- user_sessions
```

#### 2. Content Service
```typescript
// Responsibilities
- Assignment management
- Resource management
- File storage and retrieval
- Content versioning

// API Endpoints
POST   /api/assignments
GET    /api/assignments/:id
PUT    /api/assignments/:id
DELETE /api/assignments/:id
POST   /api/resources
GET    /api/resources/:id

// Database
- assignments
- resources
- submissions
- file_metadata
```

#### 3. AI Service
```typescript
// Responsibilities
- AI model integration
- Response caching
- Usage tracking
- Rate limiting

// API Endpoints
POST   /api/ai/chat
POST   /api/ai/quiz-generate
POST   /api/ai/summarize
POST   /api/ai/study-plan
GET    /api/ai/usage

// Database
- ai_cache
- ai_usage_logs
- ai_conversations
```

#### 4. Notification Service
```typescript
// Responsibilities
- Email notifications
- In-app notifications
- Push notifications
- Notification preferences

// API Endpoints
POST   /api/notifications
GET    /api/notifications
PUT    /api/notifications/:id/read
POST   /api/notifications/preferences

// Database
- notifications
- notification_preferences
- notification_logs
```

#### 5. Analytics Service
```typescript
// Responsibilities
- Usage analytics
- Performance metrics
- Report generation
- Data aggregation

// API Endpoints
GET    /api/analytics/dashboard
GET    /api/analytics/users
GET    /api/analytics/engagement
POST   /api/analytics/reports

// Database
- analytics_snapshots
- activity_logs
- performance_metrics
```

**Inter-Service Communication**:
```typescript
// Event-driven architecture using message queue
interface ServiceEvent {
  eventType: string;
  timestamp: string;
  source: string;
  data: any;
}

// Example: Assignment created event
const event: ServiceEvent = {
  eventType: 'assignment.created',
  timestamp: new Date().toISOString(),
  source: 'content-service',
  data: {
    assignmentId: '123',
    classId: '456',
    dueDate: '2026-03-01'
  }
};

// Notification service listens and sends notifications
// Analytics service listens and updates metrics
```

---

### Phase 4: Advanced Features

#### 1. Real-Time Collaboration
```typescript
// WebSocket-based real-time features
interface CollaborationFeatures {
  // Live document editing
  collaborativeEditor: {
    connect: (documentId: string) => void;
    onUserJoined: (callback: (user: User) => void) => void;
    onContentChanged: (callback: (changes: Delta) => void) => void;
    sendChanges: (changes: Delta) => void;
  };
  
  // Live class sessions
  liveSession: {
    startSession: (classId: string) => void;
    raiseHand: () => void;
    sendMessage: (message: string) => void;
    shareScreen: () => void;
  };
  
  // Real-time presence
  presence: {
    updateStatus: (status: 'online' | 'away' | 'busy') => void;
    getOnlineUsers: (classId: string) => Promise<User[]>;
  };
}
```

#### 2. Advanced AI Features
```typescript
// Multi-modal AI capabilities
interface AdvancedAI {
  // Voice interaction
  voiceAssistant: {
    startListening: () => void;
    stopListening: () => void;
    onTranscript: (callback: (text: string) => void) => void;
    speak: (text: string) => void;
  };
  
  // Image recognition
  imageAnalysis: {
    analyzeHandwriting: (image: File) => Promise<string>;
    extractTextFromImage: (image: File) => Promise<string>;
    analyzeDiagram: (image: File) => Promise<DiagramAnalysis>;
  };
  
  // Personalized learning
  adaptiveLearning: {
    generatePersonalizedContent: (studentId: string) => Promise<Content[]>;
    recommendNextTopics: (studentId: string) => Promise<Topic[]>;
    predictPerformance: (studentId: string) => Promise<PerformancePrediction>;
  };
}
```

#### 3. Mobile Applications
```typescript
// React Native mobile app structure
mobile/
├── src/
│   ├── screens/
│   │   ├── auth/
│   │   ├── dashboard/
│   │   ├── assignments/
│   │   └── ai-assistant/
│   ├── components/
│   ├── services/
│   │   └── api/ // Shared with web
│   ├── navigation/
│   └── store/
├── ios/
├── android/
└── package.json

// Shared API client
// packages/api-client/
export class SparkLinkAPI {
  constructor(baseURL: string, authToken: string) {
    this.client = axios.create({
      baseURL,
      headers: { Authorization: `Bearer ${authToken}` }
    });
  }
  
  // Shared methods used by web and mobile
  async getAssignments(classId: string) { }
  async submitAssignment(data: SubmissionData) { }
  async chatWithAI(query: string) { }
}
```

#### 4. Plugin Architecture
```typescript
// Extensible plugin system
interface Plugin {
  id: string;
  name: string;
  version: string;
  author: string;
  
  // Lifecycle hooks
  onInstall?: () => Promise<void>;
  onUninstall?: () => Promise<void>;
  onEnable?: () => Promise<void>;
  onDisable?: () => Promise<void>;
  
  // Extension points
  routes?: PluginRoute[];
  components?: PluginComponent[];
  hooks?: PluginHook[];
  settings?: PluginSettings;
}

// Example plugin: Plagiarism Checker
const plagiarismPlugin: Plugin = {
  id: 'plagiarism-checker',
  name: 'Plagiarism Checker',
  version: '1.0.0',
  author: 'SparkLink Team',
  
  routes: [
    {
      path: '/api/plagiarism/check',
      handler: async (req, res) => {
        const { content } = req.body;
        const result = await checkPlagiarism(content);
        res.json(result);
      }
    }
  ],
  
  components: [
    {
      name: 'PlagiarismReport',
      location: 'submission-view',
      component: PlagiarismReportComponent
    }
  ],
  
  hooks: [
    {
      event: 'submission.created',
      handler: async (submission) => {
        await queuePlagiarismCheck(submission.id);
      }
    }
  ]
};
```

#### 5. Multi-Tenant Architecture
```typescript
// Support multiple institutions
interface TenantConfig {
  tenantId: string;
  subdomain: string;
  customDomain?: string;
  branding: {
    logo: string;
    primaryColor: string;
    secondaryColor: string;
  };
  features: {
    aiAssistant: boolean;
    advancedAnalytics: boolean;
    customIntegrations: boolean;
  };
  limits: {
    maxUsers: number;
    maxStorage: number;
    aiRequestsPerMonth: number;
  };
}

// Tenant isolation at database level
CREATE SCHEMA tenant_abc123;
CREATE SCHEMA tenant_xyz789;

// Row-level security with tenant_id
CREATE POLICY "tenant_isolation" ON profiles
  FOR ALL USING (
    institution_id = current_setting('app.current_tenant_id')::UUID
  );

// Middleware for tenant resolution
export async function tenantMiddleware(req: NextRequest) {
  // Resolve tenant from subdomain or custom domain
  const host = req.headers.get('host');
  const tenant = await resolveTenant(host);
  
  if (!tenant) {
    return NextResponse.redirect('/tenant-not-found');
  }
  
  // Set tenant context
  req.headers.set('x-tenant-id', tenant.id);
  
  return NextResponse.next();
}
```

---

### Technology Upgrade Paths

#### Database Scaling
```
Current: Single PostgreSQL instance
↓
Phase 1: Connection pooling + Query optimization
↓
Phase 2: Read replicas for analytics
↓
Phase 3: Sharding by institution_id
↓
Phase 4: Distributed database (CockroachDB/YugabyteDB)
```

#### Caching Evolution
```
Current: PostgreSQL-based cache
↓
Phase 1: Redis for hot data
↓
Phase 2: Multi-tier caching (Redis + CDN)
↓
Phase 3: Distributed cache (Redis Cluster)
```

#### AI Service Evolution
```
Current: Direct Gemini API calls
↓
Phase 1: Response caching + rate limiting
↓
Phase 2: Model fine-tuning on institutional data
↓
Phase 3: Self-hosted models for sensitive data
↓
Phase 4: Hybrid approach (cloud + on-premise)
```

#### Search Enhancement
```
Current: PostgreSQL full-text search
↓
Phase 1: Optimized indexes + tsvector
↓
Phase 2: Elasticsearch for advanced search
↓
Phase 3: Vector search for semantic similarity
↓
Phase 4: AI-powered search with context understanding
```

---

### API Evolution Strategy

#### GraphQL Migration
```typescript
// Current: REST API
GET /api/assignments?class_id=123
GET /api/submissions?assignment_id=456
GET /api/resources?class_id=123

// Future: GraphQL API
query GetClassData($classId: ID!) {
  class(id: $classId) {
    id
    name
    assignments {
      id
      title
      dueDate
      submissions {
        id
        student {
          id
          name
        }
        score
      }
    }
    resources {
      id
      title
      fileUrl
    }
  }
}

// Benefits:
// - Single request for related data
// - Client specifies exact data needed
// - Strongly typed schema
// - Real-time subscriptions
```

#### API Versioning
```typescript
// URL-based versioning
/api/v1/assignments
/api/v2/assignments

// Header-based versioning
GET /api/assignments
Headers: { 'API-Version': '2.0' }

// Deprecation strategy
{
  "version": "1.0",
  "deprecated": true,
  "sunset_date": "2027-01-01",
  "migration_guide": "https://docs.sparklink.com/api/v1-to-v2"
}
```

---

### Monitoring & Observability Evolution

```typescript
// Current: Basic logging
console.log('Assignment created', assignmentId);

// Phase 1: Structured logging
logger.info('Assignment created', {
  assignmentId,
  classId,
  facultyId,
  timestamp: new Date()
});

// Phase 2: Distributed tracing
import { trace } from '@opentelemetry/api';

const span = trace.getTracer('sparklink').startSpan('create-assignment');
span.setAttribute('assignment.id', assignmentId);
span.setAttribute('class.id', classId);
// ... operation
span.end();

// Phase 3: Full observability stack
// - Metrics: Prometheus
// - Logs: Loki
// - Traces: Jaeger
// - Dashboards: Grafana
```

---

### Documentation & Developer Experience

```typescript
// API documentation with OpenAPI/Swagger
/**
 * @swagger
 * /api/assignments:
 *   post:
 *     summary: Create a new assignment
 *     tags: [Assignments]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/CreateAssignment'
 *     responses:
 *       201:
 *         description: Assignment created successfully
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Assignment'
 */

// Interactive API playground
// https://api.sparklink.com/playground

// SDK generation for multiple languages
// - TypeScript/JavaScript
// - Python
// - Java
// - Go
```

---

## Conclusion

This system design document provides a comprehensive blueprint for building SparkLink from MVP to enterprise scale. The architecture prioritizes:

1. **Rapid Development**: Monolithic Next.js for fast MVP delivery
2. **Security First**: Multi-layer security with RLS and RBAC
3. **Scalability**: Clear upgrade paths for each component
4. **Maintainability**: Modular design with clear boundaries
5. **Extensibility**: Plugin architecture for future features
6. **Cost Efficiency**: Free-tier friendly with linear scaling costs

The design balances immediate needs (hackathon MVP) with long-term vision (enterprise platform), ensuring SparkLink can grow from a college pilot to a multi-institution deployment without major architectural rewrites.

---

**Document Version**: 1.0  
**Last Updated**: February 14, 2026  
**Next Review**: May 14, 2026  
**Maintained By**: Architecture Team

=======
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
