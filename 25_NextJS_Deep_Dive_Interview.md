# NEXT.JS DEEP DIVE - INTERVIEW GUIDE

## 🚀 NEXT.JS CORE CONCEPTS

### 1. Rendering Strategies
Next.js cung cấp 4 rendering methods chính:

#### **Static Site Generation (SSG)**
```javascript
// pages/posts/[id].js
export async function getStaticProps({ params }) {
  const post = await fetchPost(params.id);
  
  return {
    props: { post },
    revalidate: 60 // ISR - regenerate every 60s
  };
}

export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  const paths = posts.map(post => ({
    params: { id: post.id.toString() }
  }));

  return {
    paths,
    fallback: 'blocking' // 'true' | 'false' | 'blocking'
  };
}
```

#### **Server-Side Rendering (SSR)**
```javascript
// pages/dashboard.js
export async function getServerSideProps({ req, query }) {
  const token = req.cookies.token;
  
  if (!token) {
    return {
      redirect: {
        destination: '/login',
        permanent: false
      }
    };
  }

  const userData = await fetchUserData(token);
  
  return {
    props: { userData }
  };
}
```

#### **Client-Side Rendering (CSR)**
```javascript
// components/Profile.js
import { useState, useEffect } from 'react';

export default function Profile() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(setUser);
  }, []);

  if (!user) return <div>Loading...</div>;
  return <div>Welcome {user.name}</div>;
}
```

#### **Incremental Static Regeneration (ISR)**
```javascript
export async function getStaticProps() {
  const posts = await fetchPosts();
  
  return {
    props: { posts },
    revalidate: 10, // Regenerate page every 10 seconds
  };
}
```

---

## 🛣️ ROUTING SYSTEM

### App Router (Next.js 13+) vs Pages Router

#### **App Router Structure**
```
app/
├── layout.tsx          # Root layout
├── page.tsx           # Home page
├── about/page.tsx     # /about
├── blog/
│   ├── layout.tsx     # Blog layout
│   ├── page.tsx       # /blog
│   └── [slug]/page.tsx # /blog/[slug]
└── api/
    └── users/route.ts # API route
```

#### **Pages Router Structure**
```
pages/
├── _app.tsx           # Global app wrapper
├── _document.tsx      # HTML document
├── index.tsx          # Home page
├── about.tsx          # /about
├── blog/
│   ├── index.tsx      # /blog
│   └── [slug].tsx     # /blog/[slug]
└── api/
    └── users.ts       # API route
```

### Dynamic Routes Examples

#### **Catch-all Routes**
```javascript
// pages/shop/[...slug].js
// Matches: /shop/a, /shop/a/b, /shop/a/b/c
export default function Shop({ params }) {
  const { slug } = params; // ['a', 'b', 'c']
  return <div>Shop: {slug.join('/')}</div>;
}
```

#### **Optional Catch-all Routes**
```javascript
// pages/shop/[[...slug]].js  
// Matches: /shop, /shop/a, /shop/a/b
export default function Shop({ params }) {
  const slug = params?.slug || [];
  return <div>Shop: {slug.join('/')}</div>;
}
```

---

## 🛡️ MIDDLEWARE & AUTHENTICATION

### Middleware (Next.js 12+)
```javascript
// middleware.js
import { NextResponse } from 'next/server';
import { jwtVerify } from 'jose';

export async function middleware(request) {
  const token = request.cookies.get('token')?.value;
  
  // Protect admin routes
  if (request.nextUrl.pathname.startsWith('/admin')) {
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
    
    try {
      await jwtVerify(token, new TextEncoder().encode(process.env.JWT_SECRET));
    } catch {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }
  
  // Add custom headers
  const response = NextResponse.next();
  response.headers.set('X-Version', '1.0');
  return response;
}

export const config = {
  matcher: ['/admin/:path*', '/dashboard/:path*']
};
```

### Authentication Pattern
```javascript
// lib/auth.js
import jwt from 'jsonwebtoken';

export function generateToken(payload) {
  return jwt.sign(payload, process.env.JWT_SECRET, { expiresIn: '7d' });
}

export function verifyToken(token) {
  try {
    return jwt.verify(token, process.env.JWT_SECRET);
  } catch {
    return null;
  }
}

// pages/api/auth/login.js
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }
  
  const { email, password } = req.body;
  const user = await authenticateUser(email, password);
  
  if (!user) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }
  
  const token = generateToken({ userId: user.id });
  
  // Set httpOnly cookie
  res.setHeader('Set-Cookie', `token=${token}; HttpOnly; Path=/; Max-Age=604800`);
  res.status(200).json({ user: { id: user.id, email: user.email } });
}
```

---

## 📡 API ROUTES

### RESTful API Design
```javascript
// pages/api/users/index.js
export default async function handler(req, res) {
  switch (req.method) {
    case 'GET':
      const users = await prisma.user.findMany({
        select: { id: true, email: true, name: true }
      });
      return res.status(200).json(users);
      
    case 'POST':
      const { name, email } = req.body;
      
      if (!name || !email) {
        return res.status(400).json({ error: 'Name and email required' });
      }
      
      try {
        const user = await prisma.user.create({
          data: { name, email }
        });
        return res.status(201).json(user);
      } catch (error) {
        if (error.code === 'P2002') {
          return res.status(409).json({ error: 'Email already exists' });
        }
        return res.status(500).json({ error: 'Internal server error' });
      }
      
    default:
      res.setHeader('Allow', ['GET', 'POST']);
      return res.status(405).json({ error: 'Method not allowed' });
  }
}

// pages/api/users/[id].js
export default async function handler(req, res) {
  const { id } = req.query;
  
  switch (req.method) {
    case 'GET':
      const user = await prisma.user.findUnique({ where: { id: parseInt(id) } });
      if (!user) return res.status(404).json({ error: 'User not found' });
      return res.status(200).json(user);
      
    case 'PUT':
      // Update user logic
      break;
      
    case 'DELETE':
      await prisma.user.delete({ where: { id: parseInt(id) } });
      return res.status(204).end();
      
    default:
      res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
      return res.status(405).json({ error: 'Method not allowed' });
  }
}
```

### App Router API Routes (Next.js 13+)
```javascript
// app/api/users/route.js
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request) {
  const { searchParams } = new URL(request.url);
  const page = searchParams.get('page') || '1';
  
  const users = await getUsers({ page: parseInt(page) });
  return NextResponse.json(users);
}

export async function POST(request) {
  const body = await request.json();
  const user = await createUser(body);
  return NextResponse.json(user, { status: 201 });
}

// app/api/users/[id]/route.js
export async function GET(request, { params }) {
  const user = await getUserById(params.id);
  if (!user) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }
  return NextResponse.json(user);
}
```

---

## 🎨 STYLING & OPTIMIZATION

### CSS-in-JS với Styled Components
```javascript
// components/Button.js
import styled from 'styled-components';

const StyledButton = styled.button`
  background: ${props => props.primary ? 'blue' : 'white'};
  color: ${props => props.primary ? 'white' : 'blue'};
  padding: 1rem 2rem;
  border: 2px solid blue;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    opacity: 0.8;
  }
`;

export default function Button({ children, primary, ...props }) {
  return (
    <StyledButton primary={primary} {...props}>
      {children}
    </StyledButton>
  );
}

// pages/_app.js
import { ThemeProvider } from 'styled-components';

const theme = {
  colors: { primary: '#0070f3' }
};

export default function App({ Component, pageProps }) {
  return (
    <ThemeProvider theme={theme}>
      <Component {...pageProps} />
    </ThemeProvider>
  );
}
```

### Image Optimization
```javascript
import Image from 'next/image';

// Static import
import profilePic from '../public/profile.jpg';

export default function Profile() {
  return (
    <div>
      {/* Static image */}
      <Image
        src={profilePic}
        alt="Profile picture"
        width={500}
        height={500}
        placeholder="blur"
        priority // Load immediately
      />
      
      {/* Dynamic image */}
      <Image
        src="/dynamic-image.jpg"
        alt="Dynamic image" 
        fill
        sizes="(max-width: 768px) 100vw, 50vw"
        style={{ objectFit: 'cover' }}
      />
    </div>
  );
}
```

---

## 🎯 INTERVIEW QUESTIONS & ANSWERS

### 1. **"SSG vs SSR vs ISR - Khi nào dùng từng loại?"**

```
SSG (Static Site Generation):
✅ Content ít thay đổi (blog, documentation)
✅ SEO quan trọng
✅ Performance tối ưu
❌ Không phù hợp với user-specific data

SSR (Server-Side Rendering):
✅ Content thay đổi theo user/request
✅ Cần fresh data mỗi lần
✅ Authentication pages
❌ Slower than static

ISR (Incremental Static Regeneration):
✅ Best of both: static + fresh data
✅ E-commerce product pages
✅ High traffic với content updates
❌ Complex caching logic
```

### 2. **"Next.js App Router vs Pages Router?"**

```
App Router (Next.js 13+):
- File-based routing trong app/ folder
- React Server Components by default
- Improved performance
- Co-located layouts
- Streaming và Suspense support

Pages Router (Legacy):
- File-based routing trong pages/ folder  
- Client Components by default
- Mature, stable
- Many tutorials/examples available

Migration: Có thể dùng cả hai cùng lúc
```

### 3. **"Làm thế nào để optimize Next.js app?"**

```
Performance:
- Image optimization (next/image)
- Font optimization (next/font)
- Code splitting tự động
- Bundle analyzer
- Service Workers với workbox

SEO:
- Meta tags trong Head
- Structured data (JSON-LD)
- Sitemap generation
- robots.txt

Loading:
- Prefetching với Link component
- Dynamic imports
- Streaming với Suspense
- Loading states
```

### 4. **"Next.js Middleware hoạt động như thế nào?"**

```
- Chạy trước khi request đến page/API
- Edge runtime (faster startup)
- Use cases:
  * Authentication
  * Redirects
  * Header modification
  * Geolocation
  * A/B testing
  * Rate limiting

Limitations:
- No Node.js APIs
- Limited bundle size
- Chỉ chạy trên Edge runtime
```

### 5. **"Dynamic routing với catch-all routes?"**

```javascript
// [slug].js - Single dynamic segment
// [id].js matches: /posts/123

// [...slug].js - Catch-all routes  
// Matches: /shop/a, /shop/a/b, /shop/a/b/c
// params.slug = ['a', 'b', 'c']

// [[...slug]].js - Optional catch-all
// Matches: /shop, /shop/a, /shop/a/b  
// params.slug = [] | ['a'] | ['a', 'b']

Priority:
1. Static routes: /about
2. Dynamic routes: /posts/[id]
3. Catch-all: /posts/[...slug]
```

---

## 💻 PRACTICAL EXAMPLES

### E-commerce Product Page
```javascript
// pages/products/[slug].js
import { useState } from 'react';
import Image from 'next/image';
import { useRouter } from 'next/router';

export default function ProductPage({ product }) {
  const router = useRouter();
  const [selectedSize, setSelectedSize] = useState('');
  const [loading, setLoading] = useState(false);

  const addToCart = async () => {
    setLoading(true);
    try {
      await fetch('/api/cart', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          productId: product.id,
          size: selectedSize
        })
      });
      router.push('/cart');
    } catch (error) {
      console.error('Add to cart failed:', error);
    } finally {
      setLoading(false);
    }
  };

  if (router.isFallback) {
    return <div>Loading...</div>;
  }

  return (
    <div className="product-page">
      <div className="product-images">
        <Image
          src={product.image}
          alt={product.name}
          width={600}
          height={600}
          priority
        />
      </div>
      
      <div className="product-info">
        <h1>{product.name}</h1>
        <p className="price">${product.price}</p>
        
        <div className="size-selector">
          {product.sizes.map(size => (
            <button
              key={size}
              className={selectedSize === size ? 'selected' : ''}
              onClick={() => setSelectedSize(size)}
            >
              {size}
            </button>
          ))}
        </div>
        
        <button 
          onClick={addToCart}
          disabled={!selectedSize || loading}
          className="add-to-cart"
        >
          {loading ? 'Adding...' : 'Add to Cart'}
        </button>
      </div>
    </div>
  );
}

export async function getStaticPaths() {
  const products = await prisma.product.findMany({
    select: { slug: true }
  });
  
  const paths = products.map(product => ({
    params: { slug: product.slug }
  }));

  return {
    paths,
    fallback: 'blocking'
  };
}

export async function getStaticProps({ params }) {
  const product = await prisma.product.findUnique({
    where: { slug: params.slug },
    include: { 
      images: true,
      reviews: {
        take: 10,
        orderBy: { createdAt: 'desc' }
      }
    }
  });

  if (!product) {
    return { notFound: true };
  }

  return {
    props: { product },
    revalidate: 300 // 5 minutes
  };
}
```

### Real-time Chat with WebSocket
```javascript
// pages/chat.js
import { useState, useEffect, useRef } from 'react';
import io from 'socket.io-client';

export default function Chat() {
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState('');
  const [connected, setConnected] = useState(false);
  const socketRef = useRef();

  useEffect(() => {
    // Initialize socket connection
    socketRef.current = io('/api/socket');
    
    socketRef.current.on('connect', () => {
      setConnected(true);
    });

    socketRef.current.on('message', (message) => {
      setMessages(prev => [...prev, message]);
    });

    socketRef.current.on('disconnect', () => {
      setConnected(false);
    });

    return () => socketRef.current.close();
  }, []);

  const sendMessage = (e) => {
    e.preventDefault();
    if (newMessage.trim() && connected) {
      socketRef.current.emit('message', {
        text: newMessage,
        timestamp: Date.now()
      });
      setNewMessage('');
    }
  };

  return (
    <div className="chat-container">
      <div className="connection-status">
        Status: {connected ? '🟢 Connected' : '🔴 Disconnected'}
      </div>
      
      <div className="messages">
        {messages.map((msg, index) => (
          <div key={index} className="message">
            <span className="timestamp">
              {new Date(msg.timestamp).toLocaleTimeString()}
            </span>
            <span className="text">{msg.text}</span>
          </div>
        ))}
      </div>
      
      <form onSubmit={sendMessage} className="message-form">
        <input
          type="text"
          value={newMessage}
          onChange={(e) => setNewMessage(e.target.value)}
          placeholder="Type a message..."
          disabled={!connected}
        />
        <button type="submit" disabled={!connected || !newMessage.trim()}>
          Send
        </button>
      </form>
    </div>
  );
}

// pages/api/socket.js
import { Server } from 'socket.io';

export default function handler(req, res) {
  if (!res.socket.server.io) {
    const io = new Server(res.socket.server);
    
    io.on('connection', (socket) => {
      console.log('Client connected:', socket.id);
      
      socket.on('message', (message) => {
        // Broadcast to all connected clients
        io.emit('message', {
          ...message,
          id: socket.id
        });
      });

      socket.on('disconnect', () => {
        console.log('Client disconnected:', socket.id);
      });
    });
    
    res.socket.server.io = io;
  }
  
  res.end();
}
```

---

## 🏗️ SYSTEM DESIGN với Next.js

### Full-stack E-commerce Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   NEXT.JS APP   │    │   API ROUTES    │    │    DATABASE     │
│                 │    │                 │    │                 │
│ - SSG Products  │◄──►│ - Auth APIs     │◄──►│ - PostgreSQL    │
│ - SSR Cart      │    │ - Product APIs  │    │ - Redis Cache   │
│ - CSR Dashboard │    │ - Order APIs    │    │ - File Storage  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌─────────────────┐
│   EXTERNAL      │    │   SERVICES      │
│                 │    │                 │
│ - Payment API   │    │ - Email Service │
│ - Shipping API  │    │ - Analytics     │
│ - CDN           │    │ - Monitoring    │
└─────────────────┘    └─────────────────┘
```

---

## ✅ CHECKLIST CHUẨN BỊ PHỎNG VẤN

### **Core Concepts:**
- [ ] SSG vs SSR vs ISR vs CSR
- [ ] App Router vs Pages Router
- [ ] Dynamic routing patterns
- [ ] Middleware functionality
- [ ] API Routes design

### **Performance:**
- [ ] Image optimization
- [ ] Code splitting
- [ ] Bundle analysis
- [ ] Caching strategies
- [ ] Core Web Vitals

### **Advanced Topics:**
- [ ] Authentication patterns
- [ ] Database integration
- [ ] Real-time features
- [ ] SEO optimization
- [ ] Deployment strategies

### **Practical Skills:**
- [ ] Build complete CRUD app
- [ ] Implement authentication
- [ ] Create responsive layouts
- [ ] Handle form submissions
- [ ] Error handling và loading states

---

## 🔥 NEXT.JS 13+ & REACT SERVER COMPONENTS

### React Server Components (RSC)
```javascript
// app/posts/page.tsx - Server Component (default)
import { Suspense } from 'react';
import PostsList from './posts-list';
import { getPosts } from '@/lib/db';

export default async function PostsPage() {
  // This runs on server, not sent to client
  const posts = await getPosts();
  
  return (
    <div>
      <h1>Blog Posts</h1>
      <Suspense fallback={<PostsSkeleton />}>
        <PostsList posts={posts} />
      </Suspense>
    </div>
  );
}

// app/components/search.tsx - Client Component
'use client';
import { useState } from 'react';

export default function Search() {
  const [query, setQuery] = useState('');
  
  return (
    <input
      type="text"
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search posts..."
    />
  );
}
```

### App Router File Conventions
```
app/
├── layout.tsx         # Root layout (required)
├── page.tsx          # Home page
├── loading.tsx       # Loading UI
├── error.tsx         # Error UI
├── not-found.tsx     # 404 page
├── global-error.tsx  # Global error boundary
└── blog/
    ├── layout.tsx    # Blog section layout
    ├── page.tsx      # /blog
    ├── loading.tsx   # Blog loading state
    ├── [slug]/
    │   ├── page.tsx  # /blog/[slug]
    │   └── loading.tsx
    └── @sidebar/     # Parallel route
        └── page.tsx
```

### Data Fetching với Cache
```javascript
// app/posts/page.tsx
async function getPosts() {
  // Cached by default
  const res = await fetch('https://api.example.com/posts');
  return res.json();
}

// Force fresh data
async function getFreshPosts() {
  const res = await fetch('https://api.example.com/posts', {
    cache: 'no-store' // or next: { revalidate: 0 }
  });
  return res.json();
}

// Cache with time-based revalidation
async function getPostsWithRevalidation() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 60 } // Revalidate every 60 seconds
  });
  return res.json();
}

// Cache with tags for granular invalidation
async function getPostsWithTags() {
  const res = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }
  });
  return res.json();
}

// Invalidate tagged cache
import { revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  // Update posts
  await updatePosts();
  
  // Invalidate all requests tagged with 'posts'
  revalidateTag('posts');
  
  return Response.json({ success: true });
}
```

### Server Actions (Next.js 14)
```javascript
// app/actions.ts
'use server';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  
  // Validate and save to database
  const post = await db.post.create({
    data: { title, content }
  });
  
  // Revalidate the posts page
  revalidatePath('/posts');
  
  // Redirect to the new post
  redirect(`/posts/${post.id}`);
}

// app/new-post/page.tsx
import { createPost } from '../actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Post title" />
      <textarea name="content" placeholder="Post content" />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

### Metadata API
```javascript
// app/posts/[id]/page.tsx
import { Metadata } from 'next';

type Props = {
  params: { id: string }
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.id);
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.image],
      type: 'article',
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    }
  };
}
```

### Parallel & Intercepting Routes
```javascript
// app/dashboard/@analytics/page.tsx
export default function Analytics() {
  return <div>Analytics Dashboard</div>;
}

// app/dashboard/@team/page.tsx  
export default function Team() {
  return <div>Team Overview</div>;
}

// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
}) {
  return (
    <div>
      {children}
      <div style={{ display: 'flex' }}>
        <div>{analytics}</div>
        <div>{team}</div>
      </div>
    </div>
  );
}

// Intercepting Routes - app/photos/(..)modal/page.tsx
// Intercepts /modal when navigating from /photos
export default function PhotoModal() {
  return <div>Photo Modal</div>;
}
```

---

## ⚡ EDGE RUNTIME & PERFORMANCE

### Edge Runtime vs Node.js Runtime
```javascript
// app/api/edge/route.ts
export const runtime = 'edge';

export async function GET(request: Request) {
  // Runs on Edge Runtime (V8 isolates)
  // - Faster cold start (<1ms)
  // - Limited APIs (no fs, child_process)
  // - Smaller memory footprint
  
  const country = request.headers.get('cf-ipcountry');
  
  return Response.json({
    message: `Hello from ${country}`,
    timestamp: Date.now()
  });
}

// app/api/node/route.ts (default)
export async function GET() {
  // Runs on Node.js Runtime
  // - Full Node.js API access
  // - Slower cold start (~100ms)
  // - More memory usage
  
  const fs = require('fs');
  const data = fs.readFileSync('./data.json', 'utf8');
  
  return Response.json({ data });
}
```

### Advanced next.config.js
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Webpack 5 vs Turbopack
  experimental: {
    turbo: {
      rules: {
        '*.svg': {
          loaders: ['@svgr/webpack'],
          as: '*.js',
        },
      },
    },
  },
  
  // Headers for security
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: "default-src 'self'; script-src 'self' 'unsafe-eval';"
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          }
        ]
      }
    ];
  },
  
  // Rewrites for API proxying
  async rewrites() {
    return [
      {
        source: '/api/external/:path*',
        destination: 'https://external-api.com/:path*'
      }
    ];
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true
      }
    ];
  },
  
  // Image optimization
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
        port: '',
        pathname: '/images/**',
      }
    ],
    formats: ['image/webp', 'image/avif'],
    minimumCacheTTL: 60,
  },
  
  // Environment variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  
  // Bundle analyzer
  webpack: (config, { isServer, dev }) => {
    if (!dev && !isServer) {
      const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
      config.plugins.push(
        new BundleAnalyzerPlugin({
          analyzerMode: 'static',
          openAnalyzer: false,
        })
      );
    }
    return config;
  }
};

module.exports = nextConfig;
```

### Font & Asset Optimization
```javascript
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google';
import localFont from 'next/font/local';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
});

const customFont = localFont({
  src: './fonts/custom-font.woff2',
  display: 'swap',
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

---

## 🌍 INTERNATIONALIZATION & DOMAIN ROUTING

### i18n Configuration
```javascript
// next.config.js
const nextConfig = {
  i18n: {
    locales: ['en', 'vi', 'ja'],
    defaultLocale: 'en',
    domains: [
      {
        domain: 'example.com',
        defaultLocale: 'en',
      },
      {
        domain: 'example.vn',
        defaultLocale: 'vi',
      }
    ],
    localeDetection: false, // Disable automatic detection
  },
};

// app/[locale]/layout.tsx (App Router)
export async function generateStaticParams() {
  return [{ locale: 'en' }, { locale: 'vi' }, { locale: 'ja' }];
}

export default function LocaleLayout({
  children,
  params: { locale }
}: {
  children: React.ReactNode;
  params: { locale: string };
}) {
  return (
    <html lang={locale}>
      <body>{children}</body>
    </html>
  );
}
```

### Locale-specific Data Fetching
```javascript
// lib/i18n.ts
export async function getDictionary(locale: string) {
  const dictionaries = {
    en: () => import('./dictionaries/en.json').then(m => m.default),
    vi: () => import('./dictionaries/vi.json').then(m => m.default),
  };
  
  return dictionaries[locale as keyof typeof dictionaries]();
}

// app/[locale]/page.tsx
import { getDictionary } from '@/lib/i18n';

export default async function HomePage({
  params: { locale }
}: {
  params: { locale: string };
}) {
  const dict = await getDictionary(locale);
  
  return (
    <div>
      <h1>{dict.welcome}</h1>
      <p>{dict.description}</p>
    </div>
  );
}
```

---

## 🧪 TESTING STRATEGIES

### Unit Testing với Jest & RTL
```javascript
// __tests__/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import Button from '@/components/Button';

describe('Button Component', () => {
  it('renders button with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });
  
  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});

// __tests__/pages/api/users.test.ts
import { createMocks } from 'node-mocks-http';
import handler from '@/pages/api/users';

describe('/api/users', () => {
  it('returns users for GET request', async () => {
    const { req, res } = createMocks({ method: 'GET' });
    
    await handler(req, res);
    
    expect(res._getStatusCode()).toBe(200);
    const data = JSON.parse(res._getData());
    expect(Array.isArray(data)).toBe(true);
  });
});
```

### E2E Testing với Playwright
```javascript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('should login successfully', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="welcome-message"]')).toBeVisible();
  });
  
  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('[data-testid="email"]', 'wrong@example.com');
    await page.fill('[data-testid="password"]', 'wrongpassword');
    await page.click('[data-testid="login-button"]');
    
    await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
  });
});
```

### Testing Middleware
```javascript
// __tests__/middleware.test.ts
import { createRequest, createResponse } from 'node-mocks-http';
import { middleware } from '@/middleware';

describe('Middleware', () => {
  it('redirects to login for protected routes without token', async () => {
    const req = createRequest({
      url: '/admin/dashboard',
      headers: { host: 'localhost:3000' }
    });
    
    const res = await middleware(req);
    
    expect(res.status).toBe(302);
    expect(res.headers.get('location')).toBe('/login');
  });
});
```

---

## 🚨 ERROR HANDLING & MONITORING

### Error Boundaries & Error Pages
```javascript
// app/error.tsx
'use client';
import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log to error reporting service
    console.error('Application error:', error);
  }, [error]);

  return (
    <div className="error-boundary">
      <h2>Something went wrong!</h2>
      <details>
        <summary>Error details</summary>
        <pre>{error.message}</pre>
      </details>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// app/global-error.tsx
'use client';
export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h2>Something went wrong globally!</h2>
        <button onClick={reset}>Try again</button>
      </body>
    </html>
  );
}

// app/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>Could not find requested resource</p>
    </div>
  );
}
```

### Error Tracking với Sentry
```javascript
// sentry.client.config.js
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  debug: process.env.NODE_ENV === 'development',
  tracesSampleRate: 1.0,
  beforeSend(event) {
    // Filter out development errors
    if (process.env.NODE_ENV === 'development') {
      return null;
    }
    return event;
  }
});

// pages/_app.tsx
import { reportWebVitals } from '@/lib/analytics';

export { reportWebVitals };

// lib/analytics.ts
export function reportWebVitals(metric: any) {
  // Send to analytics service
  console.log(metric);
  
  // Send to Sentry performance monitoring
  if (metric.name === 'CLS' || metric.name === 'FCP' || metric.name === 'LCP') {
    // Track Core Web Vitals
  }
}
```

---

## 🔐 SECURITY & BEST PRACTICES

### CSRF Protection
```javascript
// lib/csrf.ts
import { randomBytes } from 'crypto';

export function generateCSRFToken(): string {
  return randomBytes(32).toString('hex');
}

export function validateCSRFToken(token: string, sessionToken: string): boolean {
  return token === sessionToken;
}

// middleware.ts
import { NextResponse } from 'next/server';
import { generateCSRFToken } from '@/lib/csrf';

export async function middleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // Add CSRF token to safe methods
  if (['GET', 'HEAD', 'OPTIONS'].includes(request.method)) {
    const csrfToken = generateCSRFToken();
    response.cookies.set('csrf-token', csrfToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict'
    });
  }
  
  return response;
}
```

### Rate Limiting
```javascript
// lib/rate-limit.ts
import { NextRequest } from 'next/server';

const rateLimitMap = new Map();

export function rateLimit(ip: string, limit: number = 10, window: number = 60000) {
  const now = Date.now();
  const windowStart = now - window;
  
  const requests = rateLimitMap.get(ip) || [];
  const recentRequests = requests.filter((time: number) => time > windowStart);
  
  if (recentRequests.length >= limit) {
    return false;
  }
  
  recentRequests.push(now);
  rateLimitMap.set(ip, recentRequests);
  
  return true;
}

// pages/api/auth/login.ts
import { rateLimit } from '@/lib/rate-limit';

export default async function handler(req: NextRequest, res: NextResponse) {
  const ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
  
  if (!rateLimit(ip as string, 5, 900000)) { // 5 attempts per 15 minutes
    return res.status(429).json({ error: 'Too many attempts' });
  }
  
  // Continue with login logic
}
```

---

## 📊 ADVANCED INTERVIEW QUESTIONS

### **6. "React Server Components vs getServerSideProps khác nhau gì?"**
```
React Server Components:
✅ Render trên server, không gửi JavaScript xuống client
✅ Có thể nested và compose với Client Components  
✅ Stream HTML chunks, không block toàn bộ page
✅ Direct database access, không cần API layer
❌ Không có state, event handlers

getServerSideProps:
✅ Pre-render HTML với fresh data
✅ Có access to request context (cookies, headers)
❌ Block toàn bộ page render
❌ Tất cả data serialize qua JSON
❌ Chỉ work với Page components
```

### **7. "Cách handle caching trong App Router?"**
```javascript
// Time-based revalidation
fetch('https://api.example.com', { 
  next: { revalidate: 60 } 
});

// Tag-based revalidation
fetch('https://api.example.com', {
  next: { tags: ['posts'] }
});

// Invalidate on-demand
import { revalidateTag } from 'next/cache';
revalidateTag('posts');

// Per-route cache control
export const revalidate = 3600; // 1 hour
export const dynamic = 'force-dynamic'; // Disable caching
```

### **8. "Streaming và Suspense hoạt động như thế nào?"**
```javascript
// Page với multiple data sources
export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<AnalyticsSkeleton />}>
        <Analytics /> {/* Slow data fetch */}
      </Suspense>
      <Suspense fallback={<ChartsSkeleton />}>
        <Charts /> {/* Another slow fetch */}
      </Suspense>
    </div>
  );
}

// HTML stream progression:
// 1. Initial shell với loading states
// 2. Analytics data arrives → replace skeleton
// 3. Charts data arrives → replace skeleton
```

### **9. "Edge Runtime có những giới hạn gì?"**
```
Không có:
❌ Node.js APIs (fs, path, crypto.createHash)
❌ Native modules / binary dependencies
❌ Evaluation of code strings (eval, Function)
❌ Environment variables at build time

Có hỗ trợ:
✅ Web APIs (fetch, Response, Headers)
✅ Crypto Web API
✅ Environment variables at runtime
✅ Import dynamic modules

Use cases phù hợp:
- Authentication middleware
- A/B testing
- Geo-based redirects
- Simple data transformations
```

### **10. "Server Actions vs API Routes khi nào dùng?"**
```
Server Actions (Next.js 14):
✅ Form submissions, mutations
✅ Progressive enhancement
✅ Type-safe end-to-end
✅ Automatic revalidation
❌ Chỉ support POST method
❌ Không có custom response headers

API Routes:
✅ RESTful APIs, multiple HTTP methods
✅ Third-party integrations
✅ Custom response formats/headers
✅ Rate limiting, middleware
❌ Requires client-side fetch code
❌ Manual cache invalidation
```

---

## ✅ ENHANCED CHECKLIST

### **App Router & RSC:**
- [ ] Server vs Client Components
- [ ] File conventions (layout, loading, error)
- [ ] Streaming với Suspense
- [ ] Metadata API & SEO
- [ ] Server Actions & form handling
- [ ] Parallel & Intercepting routes

### **Performance & Edge:**
- [ ] Edge Runtime vs Node.js limitations  
- [ ] Advanced caching strategies (tags, time-based)
- [ ] Bundle optimization với Turbopack
- [ ] Font & Image optimization strategies
- [ ] Core Web Vitals monitoring

### **Production & Security:**
- [ ] CSP headers & security hardening
- [ ] Rate limiting implementation
- [ ] Error boundaries & monitoring
- [ ] CSRF protection patterns
- [ ] i18n & domain routing

### **Testing & DevOps:**
- [ ] Unit testing API routes & components
- [ ] E2E testing với Playwright
- [ ] Middleware testing strategies
- [ ] CI/CD với Vercel Preview URLs
- [ ] Performance regression testing

---

## 🚀 NEXT STEPS

1. **Practice Projects:**
   - Blog với App Router + RSC
   - E-commerce với Server Actions
   - Dashboard với Edge middleware
   - Multi-tenant app với i18n

2. **Study Resources:**
   - Next.js 14 release notes
   - React Server Components RFC
   - Vercel Edge Functions docs
   - Next.js Conf 2023 talks

3. **Performance Testing:**
   - Lighthouse CI integration
   - Bundle analyzer automation
   - Core Web Vitals tracking
   - Edge function cold start monitoring
