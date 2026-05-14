# ReadFlow Technical Architecture

## 1. Database Schema (Supabase/PostgreSQL)

### Users
- id: uuid (PK)
- email: string
- display_name: string
- avatar_url: string
- daily_goal_minutes: integer (default: 30)
- streak_count: integer
- last_read_at: timestamp

### Books
- id: uuid (PK)
- user_id: uuid (FK)
- title: string
- author: string
- cover_url: string
- total_pages: integer
- total_chapters: integer
- raw_content: text (extracted from PDF)
- metadata: jsonb (ISBN, publisher, etc.)
- status: enum ('reading', 'completed', 'want_to_read')

### ReadingProgress
- id: uuid (PK)
- user_id: uuid (FK)
- book_id: uuid (FK)
- current_chapter: integer
- current_position_char: integer (char index for exact resume)
- percent_completed: float
- time_spent_ms: bigint
- last_accessed: timestamp

### Notes & Highlights
- id: uuid (PK)
- user_id: uuid (FK)
- book_id: uuid (FK)
- content: text
- color_hex: string
- chapter_index: integer
- type: enum ('highlight', 'note')
- created_at: timestamp

---

## 2. Backend Architecture & API Flow

### PDF Parsing Pipeline
1. **Upload**: User uploads PDF to Supabase Storage.
2. **Extraction**: Edge Function triggers `PDF.js` to extract text and structure.
3. **Structuring**: GPT-4 parses the raw text to identify chapter breaks and headings.
4. **Storage**: Structured text and metadata saved to the Books table.

### AI Reading Assistant (OpenAI Integration)
- **Summarization**: Request summaries per chapter on-demand.
- **Q&A**: Vectorize book content (using pgvector) for semantic search and RAG (Retrieval-Augmented Generation) to answer user questions.
- **Simplification**: "Explain like simple English" mode using a specific LLM prompt template.

### Sync Strategy
- Real-time subscriptions via Supabase for progress syncing across devices.
- LocalStorage caching for offline reading capability.
