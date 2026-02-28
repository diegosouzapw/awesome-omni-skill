---
name: seo-metadata
description: Optimize portfolio websites for search engines with Next.js Metadata API, structured data, Open Graph, and performance SEO.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# SEO & Metadata Optimization for Portfolio Sites

## Core Philosophy

SEO for portfolios isn't about ranking #1 for "web developer"—it's about **discoverability** when someone Googles your name, sharing beautiful previews on social media, and ensuring your projects appear professionally in search results.

## Next.js Metadata API

### 1. Root Metadata (Global Defaults)

```tsx
// app/layout.tsx
import type { Metadata, Viewport } from "next";

export const viewport: Viewport = {
  themeColor: [
    { media: "(prefers-color-scheme: light)", color: "#ffffff" },
    { media: "(prefers-color-scheme: dark)", color: "#0a0a0a" },
  ],
  width: "device-width",
  initialScale: 1,
  maximumScale: 5,
};

export const metadata: Metadata = {
  metadataBase: new URL("https://yourportfolio.com"),

  title: {
    default: "John Doe | Full-Stack Developer",
    template: "%s | John Doe", // For child pages: "Projects | John Doe"
  },

  description:
    "Full-stack developer specializing in React, Next.js, and TypeScript. Building beautiful, performant web experiences.",

  keywords: [
    "John Doe",
    "Web Developer",
    "Full-Stack Developer",
    "React Developer",
    "Next.js",
    "TypeScript",
    "Frontend Developer",
    "Portfolio",
  ],

  authors: [{ name: "John Doe", url: "https://yourportfolio.com" }],
  creator: "John Doe",
  publisher: "John Doe",

  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      "max-video-preview": -1,
      "max-image-preview": "large",
      "max-snippet": -1,
    },
  },

  // Open Graph (Facebook, LinkedIn, Discord)
  openGraph: {
    type: "website",
    locale: "en_US",
    url: "https://yourportfolio.com",
    siteName: "John Doe Portfolio",
    title: "John Doe | Full-Stack Developer",
    description:
      "Full-stack developer specializing in React, Next.js, and TypeScript.",
    images: [
      {
        url: "/og-image.png",
        width: 1200,
        height: 630,
        alt: "John Doe - Full-Stack Developer",
      },
    ],
  },

  // Twitter Card
  twitter: {
    card: "summary_large_image",
    site: "@johndoe",
    creator: "@johndoe",
    title: "John Doe | Full-Stack Developer",
    description:
      "Full-stack developer specializing in React, Next.js, and TypeScript.",
    images: ["/og-image.png"],
  },

  // Verification
  verification: {
    google: "your-google-verification-code",
    yandex: "your-yandex-verification-code",
  },

  // Alternate languages (if applicable)
  alternates: {
    canonical: "https://yourportfolio.com",
    languages: {
      "en-US": "https://yourportfolio.com",
      "es-ES": "https://yourportfolio.com/es",
    },
  },

  // App links
  other: {
    "msapplication-TileColor": "#0a0a0a",
  },
};
```

### 2. Page-Specific Metadata

```tsx
// app/projects/page.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Projects", // Becomes "Projects | John Doe" via template
  description:
    "Explore my portfolio of web applications, open source projects, and client work.",
  openGraph: {
    title: "Projects | John Doe",
    description: "Explore my portfolio of web applications and client work.",
    images: ["/og-projects.png"],
  },
};

export default function ProjectsPage() {
  return <main>...</main>;
}
```

### 3. Dynamic Metadata (Project Pages)

```tsx
// app/projects/[slug]/page.tsx
import type { Metadata, ResolvingMetadata } from "next";
import { getProjectBySlug, getAllProjects } from "@/lib/projects";
import { notFound } from "next/navigation";

interface Props {
  params: Promise<{ slug: string }>;
}

export async function generateMetadata(
  { params }: Props,
  parent: ResolvingMetadata,
): Promise<Metadata> {
  const { slug } = await params;
  const project = await getProjectBySlug(slug);

  if (!project) {
    return { title: "Project Not Found" };
  }

  // Optionally access parent metadata
  const previousImages = (await parent).openGraph?.images || [];

  return {
    title: project.title,
    description: project.description,

    openGraph: {
      title: `${project.title} | John Doe`,
      description: project.description,
      type: "article",
      publishedTime: project.date,
      authors: ["John Doe"],
      images: [
        {
          url: project.ogImage || project.image,
          width: 1200,
          height: 630,
          alt: project.title,
        },
        ...previousImages,
      ],
    },

    twitter: {
      card: "summary_large_image",
      title: project.title,
      description: project.description,
      images: [project.ogImage || project.image],
    },
  };
}

// Generate static paths for all projects
export async function generateStaticParams() {
  const projects = await getAllProjects();
  return projects.map((project) => ({
    slug: project.slug,
  }));
}

export default async function ProjectPage({ params }: Props) {
  const { slug } = await params;
  const project = await getProjectBySlug(slug);

  if (!project) notFound();

  return <article>...</article>;
}
```

### 4. Blog Post Metadata

```tsx
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPostBySlug(slug);

  if (!post) return { title: "Post Not Found" };

  return {
    title: post.title,
    description: post.excerpt,

    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: "article",
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      tags: post.tags,
      images: [
        {
          url: post.coverImage,
          width: 1200,
          height: 630,
          alt: post.title,
        },
      ],
    },

    // Article-specific
    other: {
      "article:published_time": post.publishedAt,
      "article:author": post.author.name,
      "article:section": post.category,
      "article:tag": post.tags.join(", "),
    },
  };
}
```

## Structured Data (JSON-LD)

### 1. Person Schema (About Page)

```tsx
// app/about/page.tsx
import Script from "next/script";

const personJsonLd = {
  "@context": "https://schema.org",
  "@type": "Person",
  name: "John Doe",
  url: "https://yourportfolio.com",
  image: "https://yourportfolio.com/john-doe.jpg",
  sameAs: [
    "https://twitter.com/johndoe",
    "https://linkedin.com/in/johndoe",
    "https://github.com/johndoe",
  ],
  jobTitle: "Full-Stack Developer",
  worksFor: {
    "@type": "Organization",
    name: "Freelance",
  },
  knowsAbout: ["React", "Next.js", "TypeScript", "Node.js", "PostgreSQL"],
};

export default function AboutPage() {
  return (
    <>
      <Script
        id="person-schema"
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(personJsonLd) }}
      />
      <main>...</main>
    </>
  );
}
```

### 2. Website Schema (Global)

```tsx
// app/layout.tsx
import Script from "next/script";

const websiteJsonLd = {
  "@context": "https://schema.org",
  "@type": "WebSite",
  name: "John Doe Portfolio",
  url: "https://yourportfolio.com",
  description:
    "Full-stack developer portfolio showcasing web applications and projects.",
  author: {
    "@type": "Person",
    name: "John Doe",
  },
  potentialAction: {
    "@type": "SearchAction",
    target: "https://yourportfolio.com/search?q={search_term_string}",
    "query-input": "required name=search_term_string",
  },
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Script
          id="website-schema"
          type="application/ld+json"
          dangerouslySetInnerHTML={{ __html: JSON.stringify(websiteJsonLd) }}
        />
        {children}
      </body>
    </html>
  );
}
```

### 3. Project/Creative Work Schema

```tsx
// components/project-schema.tsx
import Script from "next/script";

interface ProjectSchemaProps {
  project: {
    title: string;
    description: string;
    image: string;
    url: string;
    dateCreated: string;
    technologies: string[];
  };
}

export function ProjectSchema({ project }: ProjectSchemaProps) {
  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "CreativeWork",
    name: project.title,
    description: project.description,
    image: project.image,
    url: project.url,
    dateCreated: project.dateCreated,
    author: {
      "@type": "Person",
      name: "John Doe",
      url: "https://yourportfolio.com",
    },
    keywords: project.technologies.join(", "),
  };

  return (
    <Script
      id={`project-schema-${project.title}`}
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
    />
  );
}
```

### 4. Blog Article Schema

```tsx
// components/article-schema.tsx
export function ArticleSchema({ post }: { post: BlogPost }) {
  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "Article",
    headline: post.title,
    description: post.excerpt,
    image: post.coverImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt || post.publishedAt,
    author: {
      "@type": "Person",
      name: post.author.name,
      url: "https://yourportfolio.com",
    },
    publisher: {
      "@type": "Person",
      name: "John Doe",
      logo: {
        "@type": "ImageObject",
        url: "https://yourportfolio.com/logo.png",
      },
    },
    mainEntityOfPage: {
      "@type": "WebPage",
      "@id": `https://yourportfolio.com/blog/${post.slug}`,
    },
  };

  return (
    <Script
      id={`article-schema-${post.slug}`}
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
    />
  );
}
```

### 5. Breadcrumb Schema

```tsx
// components/breadcrumb-schema.tsx
interface BreadcrumbItem {
  name: string;
  url: string;
}

export function BreadcrumbSchema({ items }: { items: BreadcrumbItem[] }) {
  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    itemListElement: items.map((item, index) => ({
      "@type": "ListItem",
      position: index + 1,
      name: item.name,
      item: item.url,
    })),
  };

  return (
    <Script
      id="breadcrumb-schema"
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
    />
  );
}

// Usage
<BreadcrumbSchema
  items={[
    { name: "Home", url: "https://yourportfolio.com" },
    { name: "Projects", url: "https://yourportfolio.com/projects" },
    {
      name: project.title,
      url: `https://yourportfolio.com/projects/${project.slug}`,
    },
  ]}
/>;
```

## OG Image Generation

### Static OG Images

Place in `/public/og-image.png` (1200x630px)

### Dynamic OG Images with ImageResponse

```tsx
// app/api/og/route.tsx
import { ImageResponse } from "next/og";
import { NextRequest } from "next/server";

export const runtime = "edge";

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const title = searchParams.get("title") || "John Doe";
  const description = searchParams.get("description") || "Full-Stack Developer";

  return new ImageResponse(
    <div
      style={{
        height: "100%",
        width: "100%",
        display: "flex",
        flexDirection: "column",
        alignItems: "center",
        justifyContent: "center",
        backgroundColor: "#0a0a0a",
        backgroundImage:
          "radial-gradient(circle at 25% 25%, #1a1a2e 0%, transparent 50%)",
      }}
    >
      <div
        style={{
          display: "flex",
          flexDirection: "column",
          alignItems: "center",
          justifyContent: "center",
          textAlign: "center",
          padding: "40px 80px",
        }}
      >
        <h1
          style={{
            fontSize: 72,
            fontWeight: "bold",
            color: "white",
            lineHeight: 1.1,
            marginBottom: 20,
          }}
        >
          {title}
        </h1>
        <p
          style={{
            fontSize: 32,
            color: "#a1a1aa",
            marginTop: 0,
          }}
        >
          {description}
        </p>
      </div>

      {/* Footer branding */}
      <div
        style={{
          position: "absolute",
          bottom: 40,
          display: "flex",
          alignItems: "center",
          gap: 12,
        }}
      >
        <span style={{ fontSize: 24, color: "#71717a" }}>
          yourportfolio.com
        </span>
      </div>
    </div>,
    {
      width: 1200,
      height: 630,
    },
  );
}
```

### Using Dynamic OG in Metadata

```tsx
// app/projects/[slug]/page.tsx
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const project = await getProjectBySlug(slug);

  if (!project) return { title: "Not Found" };

  const ogUrl = new URL("/api/og", "https://yourportfolio.com");
  ogUrl.searchParams.set("title", project.title);
  ogUrl.searchParams.set("description", project.description);

  return {
    title: project.title,
    description: project.description,
    openGraph: {
      images: [
        {
          url: ogUrl.toString(),
          width: 1200,
          height: 630,
          alt: project.title,
        },
      ],
    },
  };
}
```

## Sitemap Generation

```tsx
// app/sitemap.ts
import { MetadataRoute } from "next";
import { getAllProjects } from "@/lib/projects";
import { getAllPosts } from "@/lib/blog";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = "https://yourportfolio.com";

  // Static pages
  const staticPages: MetadataRoute.Sitemap = [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: "monthly",
      priority: 1,
    },
    {
      url: `${baseUrl}/about`,
      lastModified: new Date(),
      changeFrequency: "monthly",
      priority: 0.8,
    },
    {
      url: `${baseUrl}/projects`,
      lastModified: new Date(),
      changeFrequency: "weekly",
      priority: 0.9,
    },
    {
      url: `${baseUrl}/blog`,
      lastModified: new Date(),
      changeFrequency: "weekly",
      priority: 0.8,
    },
    {
      url: `${baseUrl}/contact`,
      lastModified: new Date(),
      changeFrequency: "yearly",
      priority: 0.5,
    },
  ];

  // Dynamic project pages
  const projects = await getAllProjects();
  const projectPages: MetadataRoute.Sitemap = projects.map((project) => ({
    url: `${baseUrl}/projects/${project.slug}`,
    lastModified: new Date(project.updatedAt || project.date),
    changeFrequency: "monthly" as const,
    priority: 0.7,
  }));

  // Dynamic blog pages
  const posts = await getAllPosts();
  const blogPages: MetadataRoute.Sitemap = posts.map((post) => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: new Date(post.updatedAt || post.publishedAt),
    changeFrequency: "weekly" as const,
    priority: 0.6,
  }));

  return [...staticPages, ...projectPages, ...blogPages];
}
```

## Robots.txt

```tsx
// app/robots.ts
import { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: "*",
        allow: "/",
        disallow: ["/api/", "/admin/"],
      },
    ],
    sitemap: "https://yourportfolio.com/sitemap.xml",
  };
}
```

## Favicon & App Icons

```tsx
// app/manifest.ts
import { MetadataRoute } from "next";

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: "John Doe Portfolio",
    short_name: "John Doe",
    description: "Full-Stack Developer Portfolio",
    start_url: "/",
    display: "standalone",
    background_color: "#0a0a0a",
    theme_color: "#8b5cf6",
    icons: [
      {
        src: "/icon-192.png",
        sizes: "192x192",
        type: "image/png",
      },
      {
        src: "/icon-512.png",
        sizes: "512x512",
        type: "image/png",
      },
      {
        src: "/icon-512.png",
        sizes: "512x512",
        type: "image/png",
        purpose: "maskable",
      },
    ],
  };
}
```

### Icon Files Structure

```
app/
├── favicon.ico          # 32x32 favicon
├── icon.png            # App icon (auto-detected)
├── apple-icon.png      # Apple touch icon
├── opengraph-image.png # Default OG image
└── twitter-image.png   # Default Twitter image
```

## Performance SEO

### Core Web Vitals Optimization

```tsx
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    optimizeCss: true, // CSS optimization
  },
  images: {
    formats: ["image/avif", "image/webp"],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  // Compress responses
  compress: true,
};
```

### Preload Critical Resources

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <head>
        {/* Preload fonts */}
        <link
          rel="preload"
          href="/fonts/CalSans-SemiBold.woff2"
          as="font"
          type="font/woff2"
          crossOrigin="anonymous"
        />

        {/* Preconnect to external domains */}
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link
          rel="preconnect"
          href="https://fonts.gstatic.com"
          crossOrigin="anonymous"
        />

        {/* DNS prefetch for analytics */}
        <link rel="dns-prefetch" href="https://www.googletagmanager.com" />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

## SEO Checklist for Portfolio

### Technical SEO

- [ ] Unique title for every page
- [ ] Meta descriptions (150-160 chars)
- [ ] Canonical URLs set
- [ ] Sitemap.xml generated
- [ ] Robots.txt configured
- [ ] Mobile-friendly (responsive)
- [ ] HTTPS enabled
- [ ] Fast loading (<3s)

### Content SEO

- [ ] H1 on every page
- [ ] Semantic HTML structure
- [ ] Alt text on all images
- [ ] Internal linking between pages
- [ ] Clear URL structure

### Social SEO

- [ ] Open Graph tags
- [ ] Twitter Card tags
- [ ] OG images (1200x630)
- [ ] Social profile links

### Structured Data

- [ ] Person schema
- [ ] Website schema
- [ ] Article schema (blog)
- [ ] CreativeWork schema (projects)
- [ ] BreadcrumbList schema

## Testing Tools

1. **Google Search Console** - Monitor indexing
2. **Rich Results Test** - Validate structured data
3. **PageSpeed Insights** - Performance metrics
4. **Open Graph Debugger** - Facebook/LinkedIn previews
5. **Twitter Card Validator** - Twitter previews
6. **Schema Markup Validator** - JSON-LD validation
