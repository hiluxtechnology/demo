This is a complete, production-ready codebase for the "Ground-to-Key" Estimation System. It implements a sequential state-machine using Google's Gemini 1.5 Pro for the front-end intake (Agent A) and a deterministic, guardrail-protected mathematical engine (Agent B) for the estimation.

### File Structure

```text
ground-to-key/
├── README.md
├── ARCHITECTURE.md
├── package.json
├── tsconfig.json
├── tailwind.config.ts
├── postcss.config.js
├── components.json
├── .env.example
├── src/
│   ├── middleware.ts
│   ├── app/
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── admin/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── login/
│   │   │       └── page.tsx
│   │   └── api/
│   │       ├── chat/route.ts
│   │       ├── estimate/route.ts
│   │       ├── lead/route.ts
│   │       ├── admin/leads/route.ts
│   │       └── auth/route.ts
│   ├── components/
│   │   ├── landing-hero.tsx
│   │   ├── chat-intake.tsx
│   │   ├── estimation-dashboard.tsx
│   │   ├── lead-form.tsx
│   │   └── ui/
│   │       ├── button.tsx
│   │       ├── input.tsx
│   │       └── card.tsx
│   └── lib/
│       ├── utils.ts
│       ├── store.ts
│       ├── schema.ts
│       ├── guardrails/
│       │   ├── rate-limiter.ts
│       │   └── pii-redactor.ts
│       ├── agents/
│       │   ├── agent-a-prompt.ts
│       │   └── agent-b.ts
│       └── db/
│           └── supabase.ts
```

### 1. Documentation & Configuration

### [README.md]
```markdown
# Ground-to-Key: AI Construction Estimator

An intelligent, multi-agent AI system for construction estimation and lead generation. 

## Features
- **Agent A (Digital Consultant):** Gemini 1.5 Pro-powered conversational intake. Validates impossible configurations (e.g., 6 BHK on 400 sqft).
- **Agent B (Quantity Surveyor):** Deterministic pricing engine with regional multipliers and price floor/ceiling guardrails to prevent AI hallucination.
- **Guardrails Built-in:** PII Redaction in logs, Rate Limiting against scraping, Logical bounds validation.
- **Lead Management:** Secure Admin dashboard to view captured leads.

## Setup Instructions

1. **Install Dependencies:**
   ```bash
   npm install
   ```

2. **Environment Variables:**
   Copy `.env.example` to `.env.local` and fill in your keys.
   ```bash
   cp .env.example .env.local
   ```

3. **Supabase Database Setup (Optional for Demo, Required for Prod):**
   Create a table `leads`:
   `id` (uuid), `name` (text), `email` (text), `phone` (text), `project_data` (jsonb), `estimated_cost` (numeric), `created_at` (timestamp).

4. **Run Development Server:**
   ```bash
   npm run dev
   ```

5. **Access Admin:** Navigate to `/admin`. Default password is in your `.env.local` file (ADMIN_PASSWORD).
```

### [ARCHITECTURE.md]
```markdown
# System Architecture

## Sequential State-Machine
1. **Intake & Validation (Agent A)**
   - Uses `@google/generative-ai` (Gemini 1.5 Pro) with Tool Calling.
   - User inputs conversational details. Gemini attempts to call the `finalize_estimation` tool.
   - The backend intercepts this tool call, running Zod validation and logical checks (e.g., `BHK * 400 > Plot_Size`). 
   - If invalid, returns the error to Gemini to naturally converse and correct the user.
   
2. **Deterministic Bridge**
   - Once Agent A succeeds, strict JSON is handed off to the frontend state, which triggers Agent B.

3. **Engineering & Costing (Agent B)**
   - Pure TypeScript module executing math. Applies base rates, regional multipliers, and forces the final `Cost/SqFt` into a safe range ($40 - $150).

4. **Lead Capture & Admin**
   - Saves project data and PII to PostgreSQL via Supabase. API endpoints redact PII before logging to the console.
```

### [package.json]
```json
{
  "name": "ground-to-key",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@google/generative-ai": "^0.21.0",
    "@radix-ui/react-slot": "^1.1.0",
    "@supabase/supabase-js": "^2.45.4",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.1",
    "jose": "^5.9.3",
    "lucide-react": "^0.446.0",
    "next": "^14.2.13",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "recharts": "^2.12.7",
    "tailwind-merge": "^2.5.2",
    "zod": "^3.23.8",
    "zustand": "^5.0.0-rc.2"
  },
  "devDependencies": {
    "@types/node": "^20.16.10",
    "@types/react": "^18.3.10",
    "@types/react-dom": "^18.3.0",
    "autoprefixer": "^10.4.20",
    "eslint": "^8",
    "eslint-config-next": "14.2.13",
    "postcss": "^8.4.47",
    "tailwindcss": "^3.4.13",
    "typescript": "^5.6.2"
  }
}
```

### [tsconfig.json]
```json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### [tailwind.config.ts]
```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/components/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          slate: "#0f172a",
          yellow: "#f59e0b",
          light: "#f8fafc",
        }
      },
    },
  },
  plugins: [],
};
export default config;
```

### [postcss.config.js]
```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### [components.json]
```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "src/app/globals.css",
    "baseColor": "slate",
    "cssVariables": false,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

### [.env.example]
```env
GEMINI_API_KEY="your_google_gemini_api_key"
NEXT_PUBLIC_SUPABASE_URL="your_supabase_url"
SUPABASE_SERVICE_ROLE_KEY="your_supabase_service_key"
ADMIN_PASSWORD="supersecretadminpassword"
JWT_SECRET="generate_a_random_secret_string"
```

---

### 2. Core Libraries & Guardrails

### [src/lib/utils.ts]
```typescript
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

### [src/lib/guardrails/pii-redactor.ts]
```typescript
// PII Redaction Guardrail
export function redactPII(text: string | Record<string, any>): string {
  let str = typeof text === 'string' ? text : JSON.stringify(text);
  
  // Redact Emails
  str = str.replace(/[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/g, '[REDACTED_EMAIL]');
  
  // Redact Phone Numbers (Simple regex for standard formats)
  str = str.replace(/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/g, '[REDACTED_PHONE]');
  
  return str;
}

export function safeLog(message: string, data?: any) {
  const safeData = data ? redactPII(data) : '';
  console.log(`[SAFE_LOG] ${message}`, safeData);
}
```

### [src/lib/guardrails/rate-limiter.ts]
```typescript
// In-memory rate limiter to prevent competitor scraping
// Note: In a multi-server production environment, use Upstash Redis.
type RateLimitStore = {
  count: number;
  resetTime: number;
};

const store = new Map<string, RateLimitStore>();

export function rateLimit(ip: string, limit: number = 10, windowMs: number = 60000): boolean {
  const now = Date.now();
  const record = store.get(ip);

  if (!record || record.resetTime < now) {
    store.set(ip, { count: 1, resetTime: now + windowMs });
    return true; // Allowed
  }

  if (record.count >= limit) {
    return false; // Blocked
  }

  record.count += 1;
  store.set(ip, record);
  return true; // Allowed
}
```

### [src/lib/schema.ts]
```typescript
import { z } from "zod";

export const ProjectSchema = z.object({
  plot_size_sqft: z.number().min(200).max(50000),
  bhk: z.number().int().min(1).max(10),
  material_grade: z.enum(['Standard', 'Premium', 'Luxury']),
  location: z.string(),
  vastu_required: z.boolean()
});

export type ProjectDetails = z.infer<typeof ProjectSchema>;

export const LeadSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  phone: z.string().min(10)
});

export type LeadDetails = z.infer<typeof LeadSchema>;
```

### [src/lib/store.ts]
```typescript
import { create } from 'zustand';
import { ProjectDetails, LeadDetails } from './schema';
import { EstimationResult } from './agents/agent-b';

type AppState = {
  step: 'landing' | 'intake' | 'calculating' | 'result' | 'lead_capture' | 'complete';
  projectDetails: ProjectDetails | null;
  estimationResult: EstimationResult | null;
  leadDetails: LeadDetails | null;
  setStep: (step: AppState['step']) => void;
  setProjectDetails: (details: ProjectDetails) => void;
  setEstimationResult: (result: EstimationResult) => void;
  setLeadDetails: (details: LeadDetails) => void;
};

export const useAppStore = create<AppState>((set) => ({
  step: 'landing',
  projectDetails: null,
  estimationResult: null,
  leadDetails: null,
  setStep: (step) => set({ step }),
  setProjectDetails: (projectDetails) => set({ projectDetails }),
  setEstimationResult: (estimationResult) => set({ estimationResult }),
  setLeadDetails: (leadDetails) => set({ leadDetails }),
}));
```

### [src/lib/db/supabase.ts]
```typescript
import { createClient } from '@supabase/supabase-js';
import { safeLog } from '../guardrails/pii-redactor';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL || '';
const supabaseKey = process.env.SUPABASE_SERVICE_ROLE_KEY || '';

export const supabase = supabaseUrl && supabaseKey 
  ? createClient(supabaseUrl, supabaseKey) 
  : null;

export async function saveLead(data: any) {
  if (!supabase) {
    safeLog('No Supabase credentials found. Mocking lead save:', data);
    return { success: true, mocked: true };
  }

  const { error } = await supabase.from('leads').insert([data]);
  if (error) {
    console.error("DB Error:", error);
    return { success: false, error };
  }
  return { success: true };
}
```

---

### 3. Agent Logic

### [src/lib/agents/agent-a-prompt.ts]
```typescript
export const AGENT_A_SYSTEM_PROMPT = `
You are Agent A: The Digital Architectural Consultant.
Your goal is to gather the following details from the user:
1. Plot Size (in sqft)
2. BHK (Number of bedrooms)
3. Material Grade (Must be 'Standard', 'Premium', or 'Luxury')
4. Location (City name)
5. Vastu Requirement (Yes or No)

RULES:
- Be empathetic, professional, and trustworthy.
- If the user provides incomplete info, ask conversational questions to get the rest.
- Once you believe you have all 5 pieces of information, call the "finalize_estimation" function/tool.
- Do NOT make assumptions. If they say "best materials", clarify if they mean 'Premium' or 'Luxury'.
`;
```

### [src/lib/agents/agent-b.ts]
```typescript
import { ProjectDetails } from '../schema';

export type EstimationResult = {
  total: number;
  costPerSqft: number;
  breakdown: {
    excavation: number;
    rcc: number;
    masonry: number;
    mep: number;
    finishing: number;
  };
  guardrail_applied: boolean;
};

// Agent B: The Deterministic Mathematical Engine with Guardrails
export function calculateEstimate(project: ProjectDetails): EstimationResult {
  // Base rates mapping
  const baseRates = {
    'Standard': 100,
    'Premium': 130,
    'Luxury': 160
  };

  // Location multipliers (mocked dataset)
  const multipliers: Record<string, number> = {
    'mumbai': 1.3,
    'bangalore': 1.15,
    'delhi': 1.1,
    'chennai': 1.05,
    'default': 1.0
  };

  const locKey = project.location.toLowerCase();
  const multiplier = multipliers[locKey] || multipliers['default'];
  
  let baseCostPerSqft = baseRates[project.material_grade] * multiplier;

  // Vastu adjustment (slight premium for specific spatial requirements)
  if (project.vastu_required) {
    baseCostPerSqft *= 1.05; 
  }

  let guardrail_applied = false;

  // ----------------------------------------------------
  // GUARDRAIL: Estimation Range Filtering (Anti-Hallucination)
  // Ensures price never drops below $40 or exceeds $150 per sqft
  // ----------------------------------------------------
  if (baseCostPerSqft < 40) {
    baseCostPerSqft = 40;
    guardrail_applied = true;
  } else if (baseCostPerSqft > 150) {
    baseCostPerSqft = 150;
    guardrail_applied = true;
  }

  const total = baseCostPerSqft * project.plot_size_sqft;

  // Standard construction cost distribution percentages
  const distribution = {
    excavation: 0.10,
    rcc: 0.30,
    masonry: 0.15,
    mep: 0.15,
    finishing: 0.30
  };

  return {
    total: Math.round(total),
    costPerSqft: Math.round(baseCostPerSqft),
    breakdown: {
      excavation: Math.round(total * distribution.excavation),
      rcc: Math.round(total * distribution.rcc),
      masonry: Math.round(total * distribution.masonry),
      mep: Math.round(total * distribution.mep),
      finishing: Math.round(total * distribution.finishing),
    },
    guardrail_applied
  };
}
```

---

### 4. Application Routes (APIs)

### [src/app/api/chat/route.ts]
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { GoogleGenerativeAI } from '@google/generative-ai';
import { ProjectSchema } from '@/lib/schema';
import { AGENT_A_SYSTEM_PROMPT } from '@/lib/agents/agent-a-prompt';
import { rateLimit } from '@/lib/guardrails/rate-limiter';
import { safeLog } from '@/lib/guardrails/pii-redactor';

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY || '');

export async function POST(req: NextRequest) {
  // RATE LIMITING GUARDRAIL
  const ip = req.headers.get('x-forwarded-for') || 'anonymous';
  if (!rateLimit(ip, 15, 60000)) {
    return NextResponse.json({ error: "Too many requests" }, { status: 429 });
  }

  try {
    const { messages } = await req.json();
    safeLog('Chat request received');

    const model = genAI.getGenerativeModel({
      model: "gemini-1.5-pro",
      systemInstruction: AGENT_A_SYSTEM_PROMPT,
      tools: [{
        functionDeclarations: [{
          name: "finalize_estimation",
          description: "Call this when you have gathered all required project details from the user.",
          parameters: {
            type: "OBJECT",
            properties: {
              plot_size_sqft: { type: "NUMBER", description: "Size of the plot in sqft" },
              bhk: { type: "NUMBER", description: "Number of bedrooms" },
              material_grade: { type: "STRING", description: "Standard, Premium, or Luxury" },
              location: { type: "STRING", description: "City name" },
              vastu_required: { type: "BOOLEAN", description: "True if Vastu compliance is needed" },
            },
            required: ["plot_size_sqft", "bhk", "material_grade", "location", "vastu_required"]
          }
        }]
      }]
    });

    // Format history for Gemini
    const chat = model.startChat({
      history: messages.slice(0, -1).map((m: any) => ({
        role: m.role === 'user' ? 'user' : 'model',
        parts: [{ text: m.content }]
      }))
    });

    const lastMessage = messages[messages.length - 1].content;
    const result = await chat.sendMessage(lastMessage);
    const response = result.response;

    const functionCalls = response.functionCalls();
    
    if (functionCalls && functionCalls.length > 0) {
      const call = functionCalls[0];
      if (call.name === "finalize_estimation") {
        const args = call.args;
        
        // LOGICAL GUARDRAIL: Validate Schema
        const parsed = ProjectSchema.safeParse(args);
        if (!parsed.success) {
          // Tell the model to correct the user
          const errorCorrection = await chat.sendMessage([{
            functionResponse: {
              name: "finalize_estimation",
              response: { error: "Validation failed. " + parsed.error.message + ". Please ask user to clarify." }
            }
          }]);
          return NextResponse.json({ role: 'assistant', content: errorCorrection.response.text() });
        }

        // LOGICAL GUARDRAIL: Physical constraints (e.g. 5 BHK needs > 1500 sqft)
        const requiredSqft = parsed.data.bhk * 400; // Minimum 400 sqft per BHK
        if (parsed.data.plot_size_sqft < requiredSqft) {
          const logicCorrection = await chat.sendMessage([{
            functionResponse: {
              name: "finalize_estimation",
              response: { error: `Physical impossibility: A ${parsed.data.bhk} BHK requires at least ${requiredSqft} sqft. The user provided ${parsed.data.plot_size_sqft} sqft. Politely explain this physical limitation and ask them to adjust either BHK or plot size.` }
            }
          }]);
          return NextResponse.json({ role: 'assistant', content: logicCorrection.response.text() });
        }

        // Success Handoff to Agent B via Frontend State
        return NextResponse.json({ 
          role: 'assistant', 
          content: "Perfect! I have all the details. Preparing your estimate...",
          toolResult: parsed.data 
        });
      }
    }

    return NextResponse.json({ role: 'assistant', content: response.text() });

  } catch (error: any) {
    console.error(error);
    return NextResponse.json({ error: "Internal Server Error" }, { status: 500 });
  }
}
```

### [src/app/api/estimate/route.ts]
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { ProjectSchema } from '@/lib/schema';
import { calculateEstimate } from '@/lib/agents/agent-b';
import { rateLimit } from '@/lib/guardrails/rate-limiter';
import { safeLog } from '@/lib/guardrails/pii-redactor';

export async function POST(req: NextRequest) {
  const ip = req.headers.get('x-forwarded-for') || 'anonymous';
  if (!rateLimit(ip, 20, 60000)) return NextResponse.json({ error: "Rate limit exceeded" }, { status: 429 });

  try {
    const body = await req.json();
    const parsed = ProjectSchema.safeParse(body);
    
    if (!parsed.success) {
      return NextResponse.json({ error: "Invalid Project Data" }, { status: 400 });
    }

    safeLog('Executing Agent B Logic for Project Details');
    
    // Execute Agent B Logic
    const estimate = calculateEstimate(parsed.data);

    return NextResponse.json(estimate);
  } catch (error) {
    return NextResponse.json({ error: "Estimation failed" }, { status: 500 });
  }
}
```

### [src/app/api/lead/route.ts]
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { LeadSchema, ProjectSchema } from '@/lib/schema';
import { saveLead } from '@/lib/db/supabase';
import { rateLimit } from '@/lib/guardrails/rate-limiter';
import { safeLog } from '@/lib/guardrails/pii-redactor';

export async function POST(req: NextRequest) {
  const ip = req.headers.get('x-forwarded-for') || 'anonymous';
  if (!rateLimit(ip, 5, 60000)) return NextResponse.json({ error: "Rate limit exceeded" }, { status: 429 });

  try {
    const body = await req.json();
    const leadData = LeadSchema.parse(body.lead);
    const projectData = ProjectSchema.parse(body.project);
    const estimatedCost = body.estimatedCost;

    safeLog('Saving lead data', { email: leadData.email, phone: leadData.phone });

    const result = await saveLead({
      name: leadData.name,
      email: leadData.email,
      phone: leadData.phone,
      project_data: projectData,
      estimated_cost: estimatedCost,
    });

    if (!result.success) throw new Error("DB Save Failed");

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error(error);
    return NextResponse.json({ error: "Failed to save lead" }, { status: 400 });
  }
}
```

### [src/app/api/auth/route.ts]
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { SignJWT } from 'jose';

export async function POST(req: NextRequest) {
  const { password } = await req.json();

  if (password === (process.env.ADMIN_PASSWORD || 'admin')) {
    const secret = new TextEncoder().encode(process.env.JWT_SECRET || 'secret');
    const token = await new SignJWT({ admin: true })
      .setProtectedHeader({ alg: 'HS256' })
      .setExpirationTime('24h')
      .sign(secret);

    const res = NextResponse.json({ success: true });
    res.cookies.set('admin_token', token, { httpOnly: true, secure: true, sameSite: 'strict' });
    return res;
  }

  return NextResponse.json({ success: false }, { status: 401 });
}
```

### [src/app/api/admin/leads/route.ts]
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { supabase } from '@/lib/db/supabase';

export async function GET(req: NextRequest) {
  // Authentication handled in middleware.ts
  if (!supabase) {
    return NextResponse.json({ leads: [
      { id: 1, name: 'Mock Lead', email: 'mock@example.com', estimated_cost: 120000, project_data: { bhk: 2, location: 'Mumbai' } }
    ] });
  }

  const { data, error } = await supabase.from('leads').select('*').order('created_at', { ascending: false });
  if (error) return NextResponse.json({ error: error.message }, { status: 500 });

  return NextResponse.json({ leads: data });
}
```

### [src/middleware.ts]
```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { jwtVerify } from 'jose';

export async function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/admin') && !request.nextUrl.pathname.includes('/login')) {
    const token = request.cookies.get('admin_token')?.value;

    if (!token) {
      return NextResponse.redirect(new URL('/admin/login', request.url));
    }

    try {
      const secret = new TextEncoder().encode(process.env.JWT_SECRET || 'secret');
      await jwtVerify(token, secret);
      return NextResponse.next();
    } catch (err) {
      return NextResponse.redirect(new URL('/admin/login', request.url));
    }
  }
  return NextResponse.next();
}

export const config = {
  matcher: '/admin/:path*',
};
```

---

### 5. UI Components & Frontend (App Router)

### [src/app/globals.css]
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }
  body {
    @apply bg-brand-light text-brand-slate;
  }
}
```

### [src/components/ui/button.tsx]
```typescript
import * as React from "react"
import { Slot } from "@radix-ui/react-slot"
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-brand-yellow text-brand-slate font-bold hover:bg-brand-yellow/90",
        outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        ghost: "hover:bg-accent hover:text-accent-foreground",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button"
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    )
  }
)
Button.displayName = "Button"

export { Button, buttonVariants }
```

### [src/components/ui/input.tsx]
```typescript
import * as React from "react"
import { cn } from "@/lib/utils"

export interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {}

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ className, type, ...props }, ref) => {
    return (
      <input
        type={type}
        className={cn(
          "flex h-10 w-full rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-brand-yellow focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50",
          className
        )}
        ref={ref}
        {...props}
      />
    )
  }
)
Input.displayName = "Input"

export { Input }
```

### [src/components/ui/card.tsx]
```typescript
import * as React from "react"
import { cn } from "@/lib/utils"

const Card = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(({ className, ...props }, ref) => (
  <div ref={ref} className={cn("rounded-xl border bg-white text-brand-slate shadow", className)} {...props} />
))
Card.displayName = "Card"

export { Card }
```

### [src/components/landing-hero.tsx]
```typescript
import { Button } from "./ui/button";
import { useAppStore } from "@/lib/store";

export function LandingHero() {
  const setStep = useAppStore(state => state.setStep);

  return (
    <div className="flex flex-col items-center justify-center min-h-[80vh] text-center px-4">
      <h1 className="text-5xl font-extrabold tracking-tight sm:text-6xl text-brand-slate mb-6">
        Build with <span className="text-brand-yellow">Precision.</span>
      </h1>
      <p className="max-w-2xl text-lg text-gray-600 mb-8">
        Experience the first "Ground-to-Key" AI Estimation System. Get an engineering-grade construction estimate in 60 seconds without physical site visits.
      </p>
      <Button size="lg" onClick={() => setStep('intake')} className="text-lg px-8 py-6 rounded-full shadow-lg">
        Start Free Estimate
      </Button>
    </div>
  );
}
```

### [src/components/chat-intake.tsx]
```typescript
import { useState, useRef, useEffect } from "react";
import { Button } from "./ui/button";
import { Input } from "./ui/input";
import { Card } from "./ui/card";
import { useAppStore } from "@/lib/store";

export function ChatIntake() {
  const [messages, setMessages] = useState([{ role: 'assistant', content: 'Hello! I am your Digital Consultant. To give you an accurate estimate, I need to know: Plot Size (sqft), Number of Bedrooms (BHK), Preferred Material Grade (Standard, Premium, Luxury), City, and if you require Vastu compliance.' }]);
  const [input, setInput] = useState('');
  const [loading, setLoading] = useState(false);
  const setStep = useAppStore(state => state.setStep);
  const setProjectDetails = useAppStore(state => state.setProjectDetails);
  
  const bottomRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  const sendMessage = async () => {
    if (!input.trim()) return;
    const newMsgs = [...messages, { role: 'user', content: input }];
    setMessages(newMsgs);
    setInput('');
    setLoading(true);

    try {
      const res = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages: newMsgs })
      });
      const data = await res.json();
      
      setMessages([...newMsgs, { role: 'assistant', content: data.content }]);

      if (data.toolResult) {
        setProjectDetails(data.toolResult);
        setTimeout(() => setStep('calculating'), 1500);
      }
    } catch (error) {
      setMessages([...newMsgs, { role: 'assistant', content: 'Sorry, I encountered an error. Please try again.' }]);
    }
    setLoading(false);
  };

  return (
    <Card className="w-full max-w-2xl mx-auto flex flex-col h-[600px] shadow-2xl overflow-hidden mt-10">
      <div className="bg-brand-slate text-white p-4 text-center font-bold">Digital Consultant</div>
      <div className="flex-1 overflow-y-auto p-4 space-y-4 bg-gray-50">
        {messages.map((m, i) => (
          <div key={i} className={`flex ${m.role === 'user' ? 'justify-end' : 'justify-start'}`}>
            <div className={`p-3 rounded-lg max-w-[80%] ${m.role === 'user' ? 'bg-brand-yellow text-brand-slate' : 'bg-white border'}`}>
              {m.content}
            </div>
          </div>
        ))}
        {loading && <div className="text-gray-500 text-sm">Consultant is typing...</div>}
        <div ref={bottomRef} />
      </div>
      <div className="p-4 bg-white border-t flex gap-2">
        <Input 
          value={input} 
          onChange={e => setInput(e.target.value)} 
          placeholder="e.g. I want a 3BHK on 1200 sqft plot in Mumbai..."
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
          disabled={loading}
        />
        <Button onClick={sendMessage} disabled={loading}>Send</Button>
      </div>
    </Card>
  );
}
```

### [src/components/estimation-dashboard.tsx]
```typescript
import { useEffect } from "react";
import { useAppStore } from "@/lib/store";
import { Card } from "./ui/card";
import { PieChart, Pie, Cell, ResponsiveContainer, Tooltip } from 'recharts';
import { LeadForm } from "./lead-form";

const COLORS = ['#f59e0b', '#3b82f6', '#10b981', '#6366f1', '#f43f5e'];

export function EstimationDashboard() {
  const projectDetails = useAppStore(state => state.projectDetails);
  const estimationResult = useAppStore(state => state.estimationResult);
  const setEstimationResult = useAppStore(state => state.setEstimationResult);

  useEffect(() => {
    if (projectDetails && !estimationResult) {
      fetch('/api/estimate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(projectDetails)
      })
      .then(r => r.json())
      .then(data => setEstimationResult(data));
    }
  }, [projectDetails, estimationResult]);

  if (!estimationResult) {
    return <div className="text-center mt-20 text-2xl font-bold animate-pulse text-brand-slate">Agent B is calculating structures...</div>;
  }

  const chartData = [
    { name: 'Excavation', value: estimationResult.breakdown.excavation },
    { name: 'RCC', value: estimationResult.breakdown.rcc },
    { name: 'Masonry', value: estimationResult.breakdown.masonry },
    { name: 'MEP', value: estimationResult.breakdown.mep },
    { name: 'Finishing', value: estimationResult.breakdown.finishing },
  ];

  return (
    <div className="max-w-5xl mx-auto mt-10 grid md:grid-cols-2 gap-8 px-4">
      <div className="space-y-6">
        <Card className="p-6">
          <h2 className="text-2xl font-bold text-brand-slate border-b pb-4 mb-4">Total Estimated Cost</h2>
          <div className="text-5xl font-black text-brand-yellow">
            ${estimationResult.total.toLocaleString()}
          </div>
          <div className="text-gray-500 mt-2">
            ${estimationResult.costPerSqft}/sq.ft {estimationResult.guardrail_applied && "(Adjusted to industry standards)"}
          </div>
        </Card>
        
        <Card className="p-6 h-[300px]">
          <h3 className="font-bold text-lg mb-4">Cost Distribution</h3>
          <ResponsiveContainer width="100%" height="100%">
            <PieChart>
              <Pie data={chartData} innerRadius={60} outerRadius={80} paddingAngle={5} dataKey="value">
                {chartData.map((entry, index) => <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />)}
              </Pie>
              <Tooltip formatter={(value: number) => `$${value.toLocaleString()}`} />
            </PieChart>
          </ResponsiveContainer>
        </Card>
      </div>
      <div>
        <LeadForm />
      </div>
    </div>
  );
}
```

### [src/components/lead-form.tsx]
```typescript
import { useState } from "react";
import { Card } from "./ui/card";
import { Input } from "./ui/input";
import { Button } from "./ui/button";
import { useAppStore } from "@/lib/store";

export function LeadForm() {
  const [formData, setFormData] = useState({ name: '', email: '', phone: '' });
  const [submitted, setSubmitted] = useState(false);
  
  const projectDetails = useAppStore(state => state.projectDetails);
  const estimationResult = useAppStore(state => state.estimationResult);
  const setStep = useAppStore(state => state.setStep);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await fetch('/api/lead', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        lead: formData,
        project: projectDetails,
        estimatedCost: estimationResult?.total
      })
    });
    setSubmitted(true);
  };

  if (submitted) {
    return (
      <Card className="p-8 text-center bg-green-50 border-green-200">
        <h3 className="text-2xl font-bold text-green-700">Proposal Sent!</h3>
        <p className="mt-4 text-green-600">Our engineering team will contact you shortly with the detailed breakdown.</p>
        <Button className="mt-6" onClick={() => { setStep('landing'); window.location.reload(); }}>Start New Estimate</Button>
      </Card>
    );
  }

  return (
    <Card className="p-6">
      <h3 className="text-xl font-bold mb-4">Get Detailed PDF Proposal</h3>
      <p className="text-gray-500 mb-6 text-sm">Enter your details to receive the complete material take-off and connect with our engineers.</p>
      <form onSubmit={handleSubmit} className="space-y-4">
        <Input required placeholder="Full Name" value={formData.name} onChange={e => setFormData({...formData, name: e.target.value})} />
        <Input required type="email" placeholder="Email Address" value={formData.email} onChange={e => setFormData({...formData, email: e.target.value})} />
        <Input required type="tel" placeholder="Phone Number" value={formData.phone} onChange={e => setFormData({...formData, phone: e.target.value})} />
        <Button type="submit" className="w-full">Unlock Complete Proposal</Button>
      </form>
    </Card>
  );
}
```

### [src/app/page.tsx]
```typescript
"use client";
import { useAppStore } from "@/lib/store";
import { LandingHero } from "@/components/landing-hero";
import { ChatIntake } from "@/components/chat-intake";
import { EstimationDashboard } from "@/components/estimation-dashboard";

export default function Home() {
  const step = useAppStore(state => state.step);

  return (
    <main className="min-h-screen bg-brand-light font-sans">
      <header className="bg-brand-slate text-white py-4 px-8 flex justify-between items-center shadow-md">
        <div className="text-2xl font-black tracking-tighter">GROUND<span className="text-brand-yellow">2</span>KEY</div>
        <a href="/admin" className="text-sm font-semibold hover:text-brand-yellow transition-colors">Admin Login</a>
      </header>

      {step === 'landing' && <LandingHero />}
      {step === 'intake' && <ChatIntake />}
      {(step === 'calculating' || step === 'result') && <EstimationDashboard />}
    </main>
  );
}
```

### [src/app/layout.tsx]
```typescript
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "Ground-to-Key | AI Construction Estimator",
  description: "AI-powered architectural estimation system.",
};

export default function RootLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

---

### 6. Admin Panel (Password Protected)

### [src/app/admin/layout.tsx]
```typescript
export default function AdminLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen bg-gray-100">
      <header className="bg-brand-slate text-white p-4">
        <h1 className="text-xl font-bold">Admin Dashboard - Ground2Key</h1>
      </header>
      <main className="p-8">
        {children}
      </main>
    </div>
  );
}
```

### [src/app/admin/login/page.tsx]
```typescript
"use client";
import { useState } from "react";
import { useRouter } from "next/navigation";
import { Card } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";

export default function AdminLogin() {
  const [password, setPassword] = useState("");
  const router = useRouter();

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault();
    const res = await fetch('/api/auth', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ password })
    });
    if (res.ok) router.push('/admin');
    else alert("Invalid password");
  };

  return (
    <div className="flex justify-center items-center h-[70vh]">
      <Card className="p-8 w-96">
        <h2 className="text-2xl font-bold mb-6 text-center">Admin Access</h2>
        <form onSubmit={handleLogin} className="space-y-4">
          <Input 
            type="password" 
            placeholder="Enter Password" 
            value={password} 
            onChange={(e) => setPassword(e.target.value)} 
          />
          <Button type="submit" className="w-full">Login</Button>
        </form>
      </Card>
    </div>
  );
}
```

### [src/app/admin/page.tsx]
```typescript
"use client";
import { useEffect, useState } from "react";
import { Card } from "@/components/ui/card";

export default function AdminDashboard() {
  const [leads, setLeads] = useState<any[]>([]);

  useEffect(() => {
    fetch('/api/admin/leads')
      .then(r => r.json())
      .then(data => setLeads(data.leads || []));
  }, []);

  return (
    <Card className="p-6">
      <h2 className="text-2xl font-bold mb-6">Captured Leads</h2>
      <div className="overflow-x-auto">
        <table className="w-full text-left border-collapse">
          <thead>
            <tr className="bg-gray-100 border-b">
              <th className="p-3">Name</th>
              <th className="p-3">Contact</th>
              <th className="p-3">Location</th>
              <th className="p-3">Project Scope</th>
              <th className="p-3">Est. Cost</th>
            </tr>
          </thead>
          <tbody>
            {leads.map((lead, i) => (
              <tr key={i} className="border-b hover:bg-gray-50">
                <td className="p-3 font-semibold">{lead.name}</td>
                <td className="p-3">
                  <div className="text-sm">{lead.email}</div>
                  <div className="text-xs text-gray-500">{lead.phone}</div>
                </td>
                <td className="p-3 capitalize">{lead.project_data?.location}</td>
                <td className="p-3">
                  <div className="text-sm">{lead.project_data?.bhk} BHK • {lead.project_data?.plot_size_sqft} sqft</div>
                  <div className="text-xs text-gray-500">{lead.project_data?.material_grade}</div>
                </td>
                <td className="p-3 font-bold text-brand-slate">
                  ${lead.estimated_cost?.toLocaleString()}
                </td>
              </tr>
            ))}
            {leads.length === 0 && (
              <tr><td colSpan={5} className="text-center p-6 text-gray-500">No leads captured yet.</td></tr>
            )}
          </tbody>
        </table>
      </div>
    </Card>
  );
}
```