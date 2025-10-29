# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

morethan-log is a Next.js static blog that uses Notion as a CMS. Posts are fetched from a Notion database and rendered using react-notion-x. The blog supports both Blog format posts and Page format for content like resumes.

## Build Commands

```bash
# Install dependencies
yarn install

# Development server (runs on port 3000)
yarn dev

# Production build
yarn build

# Start production server
yarn start

# Linting
yarn lint

# Generate sitemap (runs automatically after build via postbuild script)
yarn postbuild
```

## Docker Commands

```bash
# Setup with Docker (requires NOTION_PAGE_ID)
make setup NOTION_PAGE_ID=your_notion_page_id

# Run development server with Docker (accessible on port 8001)
make dev

# Run Docker container shell
make run
```

## Environment Variables

Required for deployment (typically set in Vercel):

- `NOTION_PAGE_ID` (Required): The Notion page ID from the shared Notion database URL
- `NEXT_PUBLIC_GOOGLE_MEASUREMENT_ID`: For Google Analytics
- `NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION`: For Google Search Console
- `NEXT_PUBLIC_NAVER_SITE_VERIFICATION`: For Naver Search Advisor
- `NEXT_PUBLIC_UTTERANCES_REPO`: For Utterances commenting system
- `VERCEL_ENV`: Automatically set by Vercel to distinguish environments

## Configuration

All site configuration is managed in `site.config.js` including:
- Profile information (name, image, role, bio, social links)
- Blog settings (title, description, color scheme)
- Plugin configurations (Google Analytics, Search Console, Utterances, Cusdis)
- `revalidateTime`: ISR revalidation interval (default: 151200 seconds)

## Architecture

### Data Flow

1. **Notion API Integration**: Posts are fetched from Notion using `notion-client` library (src/apis/notion-client/getPosts.ts)
2. **Data Processing**: Raw Notion data is transformed into typed `TPost` objects with properties extracted from Notion's schema
3. **Static Generation**: Next.js ISR generates pages at build time and revalidates based on `revalidateTime`
4. **React Query**: Client-side data fetching and caching managed via @tanstack/react-query with prefetched data from getStaticProps

### Page Types

The blog supports three content types (defined in src/types/index.ts):
- `Post`: Regular blog posts
- `Paper`: Academic or research papers
- `Page`: Static pages (e.g., resume, portfolio)

Posts have three status levels:
- `Private`: Not shown anywhere
- `Public`: Shown in feed and detail pages
- `PublicOnDetail`: Only accessible via direct URL, not shown in feed

### Key Architecture Patterns

**Pages (src/pages/):**
- `index.tsx`: Main feed page listing all posts
- `[slug].tsx`: Dynamic detail pages for individual posts/pages
- Uses Next.js ISR with prefetched React Query data via `dehydrate`

**Routes (src/routes/):**
- `Feed/`: Post listing with filters (category, tags, search)
- `Detail/`: Individual post rendering with PostDetail (blog posts) and PageDetail (static pages)
- Detail pages use `react-notion-x` to render Notion blocks

**Notion Integration (src/libs/utils/notion/):**
- `getAllPageIds`: Extract all page IDs from Notion collection
- `getPageProperties`: Parse Notion schema to extract post metadata
- `filterPosts`: Filter posts by type and status
- Custom image URL mapping for Notion images

**Styling:**
- Uses Emotion for CSS-in-JS styling
- Theme system with light/dark mode support (src/hooks/useScheme.ts)
- Radix UI colors for design tokens (src/styles/colors.ts)

### Special Features

**Mermaid Diagrams**: Dynamic rendering with theme support via useMermaidEffect hook (src/routes/Detail/hooks/useMermaidEffect.ts). Mermaid code blocks are detected and rendered client-side with theme-aware styling.

**SEO**:
- Dynamic OG image generation using og-image-korean service
- Automatic sitemap.xml generation via next-sitemap
- MetaConfig component handles all meta tags per page

**Comments**: Supports either Utterances (GitHub Issues) or Cusdis commenting systems, configured via site.config.js

## Working with Posts

Posts are filtered using `filterPosts` utility which accepts:
- `acceptStatus`: Which status types to include (default: ["Public"])
- `acceptType`: Which content types to include (default: ["Post", "Paper"])

Example from src/pages/[slug].tsx:
```javascript
const filter: FilterPostsOptions = {
  acceptStatus: ["Public", "PublicOnDetail"],
  acceptType: ["Paper", "Post", "Page"],
}
```

## Custom Assets

- Profile avatar: `public/avatar.svg`
- Favicon: `public/favicon.ico`
- Apple touch icon: `public/apple-touch-icon.png`
- Custom fonts: Pretendard and Roboto loaded from src/assets/fonts/