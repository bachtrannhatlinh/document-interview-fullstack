# TRẢ LỜI CHI TIẾT - FRONTEND (REACT / NEXT.JS)

## 🔹 FRONTEND (REACT / NEXT.JS)

---

### **1. Giải thích cơ chế reconciliation trong React. Khi nào component re-render?**

**A: RECONCILIATION TRONG REACT**

**Reconciliation** là quá trình React so sánh Virtual DOM mới với Virtual DOM cũ để xác định những thay đổi tối thiểu cần thực hiện trên Real DOM.

**Cơ chế hoạt động:**
```javascript
// Khi state thay đổi
function MyComponent() {
    const [count, setCount] = useState(0);
    
    // Mỗi lần setCount được gọi:
    // 1. React tạo Virtual DOM tree mới
    // 2. So sánh (diff) với Virtual DOM cũ
    // 3. Tính toán minimal changes
    // 4. Cập nhật Real DOM
    
    return <div>{count}</div>;
}
```

**Diffing Algorithm:**
- **Same Type**: So sánh props và state
- **Different Type**: Unmount cũ, mount mới
- **Keys**: Xác định elements có thay đổi vị trí

**Khi nào component re-render:**
1. **State thay đổi** (useState, useReducer)
2. **Props thay đổi** từ parent component
3. **Parent re-render** (nếu không có optimization)
4. **Context value thay đổi**
5. **Force update** (useReducer với dummy dispatch)

**Ví dụ tối ưu:**
```javascript
// Tránh re-render không cần thiết
const OptimizedChild = React.memo(({ data }) => {
    return <div>{data.name}</div>;
});

// Parent component
function Parent() {
    const [count, setCount] = useState(0);
    const data = useMemo(() => ({ name: 'John' }), []); // Stable reference
    
    return (
        <div>
            <button onClick={() => setCount(count + 1)}>Count: {count}</button>
            <OptimizedChild data={data} /> {/* Không re-render khi count thay đổi */}
        </div>
    );
}
```

---

### **2. Phân biệt useMemo và useCallback. Khi nào nên dùng, khi nào không nên?**

**A: USEMEMO VS USECALLBACK**

| **useMemo** | **useCallback** |
|-------------|-----------------|
| Memoize **giá trị** | Memoize **function** |
| Tránh tính toán lại expensive operations | Tránh tạo function reference mới |
| Return value của function | Return function itself |

**useMemo - Value Memoization:**
```javascript
function ExpensiveComponent({ items }) {
    // ✅ Tốt: Expensive calculation
    const expensiveValue = useMemo(() => {
        return items.reduce((sum, item) => {
            return sum + heavyCalculation(item); // Tính toán phức tạp
        }, 0);
    }, [items]);

    // ❌ Không cần thiết: Simple calculation
    const simpleValue = useMemo(() => {
        return items.length * 2; // Quá đơn giản
    }, [items]);

    return <div>Total: {expensiveValue}</div>;
}
```

**useCallback - Function Memoization:**
```javascript
function Parent({ items }) {
    const [filter, setFilter] = useState('');
    
    // ✅ Tốt: Prevent child re-render
    const handleItemClick = useCallback((id) => {
        console.log('Clicked item:', id);
        // Xử lý logic phức tạp
    }, []); // Stable reference

    // ❌ Không cần thiết: Function không được pass down
    const localHandler = useCallback(() => {
        console.log('Local action');
    }, []);

    return (
        <div>
            {items.map(item => (
                <ExpensiveChild 
                    key={item.id} 
                    item={item}
                    onClick={handleItemClick} // Stable reference ngăn re-render
                />
            ))}
        </div>
    );
}
```

**Khi nào NÊN dùng:**
- **useMemo**: Expensive calculations, complex object/array creation
- **useCallback**: Functions passed to child components, dependency arrays

**Khi nào KHÔNG NÊN dùng:**
- **useMemo**: Simple calculations, primitive values
- **useCallback**: Functions không được pass down hoặc không có expensive children

---

### **3. React key dùng để làm gì? Nếu dùng index làm key có vấn đề gì?**

**A: REACT KEYS**

**Mục đích của keys:**
- Giúp React identify elements trong lists
- Optimize reconciliation process
- Preserve component state và DOM nodes

**Vấn đề khi dùng index làm key:**
```javascript
// ❌ Vấn đề với index keys
function TodoList({ todos }) {
    return (
        <ul>
            {todos.map((todo, index) => (
                <li key={index}> {/* Index không stable khi list thay đổi */}
                    <input type="text" defaultValue={todo.text} />
                    <button onClick={() => deleteTodo(index)}>Delete</button>
                </li>
            ))}
        </ul>
    );
}

// Vấn đề: Khi xóa item đầu tiên
// - Input values bị trộn lẫn
// - Component state bị mất
// - Performance kém do unnecessary re-creates
```

**Giải pháp tốt:**
```javascript
// ✅ Dùng unique, stable keys
function TodoList({ todos }) {
    return (
        <ul>
            {todos.map((todo) => (
                <li key={todo.id}> {/* Unique và stable */}
                    <TodoItem todo={todo} />
                </li>
            ))}
        </ul>
    );
}

// ✅ Nếu không có ID, tạo composite key
function CommentList({ comments }) {
    return (
        <div>
            {comments.map((comment) => (
                <div key={`${comment.userId}-${comment.timestamp}`}>
                    {comment.text}
                </div>
            ))}
        </div>
    );
}
```

**Best Practices:**
- Dùng unique, stable identifiers
- Tránh dùng index trừ khi list hoàn toàn static
- Không dùng random values (Math.random())

---

### **4. Trình bày cách tối ưu performance khi có list lớn (virtualization, React Window)**

**A: OPTIMIZATION CHO LARGE LISTS**

**1. React Window (react-window):**
```javascript
import { FixedSizeList as List } from 'react-window';

// Component cho mỗi item
const Row = ({ index, style }) => (
    <div style={style}>
        Item {index}
    </div>
);

// Virtualized list
function VirtualizedList({ items }) {
    return (
        <List
            height={400}        // Container height
            itemCount={items.length}
            itemSize={50}       // Mỗi item cao 50px
            width="100%"
        >
            {Row}
        </List>
    );
}
```

**2. Variable Size List:**
```javascript
import { VariableSizeList as List } from 'react-window';

// Tính toán dynamic height
const getItemSize = (index) => {
    // Logic tính height based on content
    return index % 2 === 0 ? 60 : 100;
};

function VariableSizeExample({ items }) {
    return (
        <List
            height={400}
            itemCount={items.length}
            itemSize={getItemSize}
        >
            {({ index, style }) => (
                <div style={style}>
                    <ComplexItem data={items[index]} />
                </div>
            )}
        </List>
    );
}
```

**3. React Virtuoso (Alternative):**
```javascript
import { Virtuoso } from 'react-virtuoso';

function VirtuosoExample({ items }) {
    return (
        <Virtuoso
            style={{ height: '400px' }}
            totalCount={items.length}
            itemContent={(index) => (
                <div>
                    <ItemComponent data={items[index]} />
                </div>
            )}
            endReached={() => {
                // Load more items
                loadMoreItems();
            }}
        />
    );
}
```

**4. Custom Implementation Concepts:**
```javascript
// Basic concept for custom virtualization
function useVirtualization({ items, containerHeight, itemHeight }) {
    const [scrollTop, setScrollTop] = useState(0);
    
    const visibleRange = useMemo(() => {
        const startIndex = Math.floor(scrollTop / itemHeight);
        const endIndex = Math.min(
            startIndex + Math.ceil(containerHeight / itemHeight) + 1,
            items.length - 1
        );
        
        return { startIndex, endIndex };
    }, [scrollTop, itemHeight, containerHeight, items.length]);
    
    const visibleItems = useMemo(() => {
        return items.slice(visibleRange.startIndex, visibleRange.endIndex + 1);
    }, [items, visibleRange]);
    
    return {
        visibleItems,
        visibleRange,
        totalHeight: items.length * itemHeight,
        offsetY: visibleRange.startIndex * itemHeight
    };
}
```

**Performance Benefits:**
- Chỉ render visible items
- Constant memory usage
- Smooth scrolling với thousands of items
- Improved initial load time

---

### **5. So sánh client-side routing (React Router) với server-side routing (Next.js)**

**A: CLIENT-SIDE VS SERVER-SIDE ROUTING**

| **Client-Side Routing** | **Server-Side Routing** |
|-------------------------|-------------------------|
| **React Router** | **Next.js** |
| Navigation không reload page | Full page reload (traditional) |
| Single bundle initially | Code splitting per route |
| State preserved across routes | State reset on navigation |
| SEO challenges | SEO friendly |

**React Router (Client-Side):**
```javascript
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
    return (
        <BrowserRouter>
            {/* Navigation không reload page */}
            <nav>
                <Link to="/">Home</Link>
                <Link to="/about">About</Link>
            </nav>
            
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
                <Route path="/users/:id" element={<UserProfile />} />
            </Routes>
        </BrowserRouter>
    );
}

// Protected routes
function ProtectedRoute({ children }) {
    const isAuthenticated = useAuth();
    return isAuthenticated ? children : <Navigate to="/login" />;
}
```

**Next.js (Server-Side/Hybrid):**
```javascript
// File-based routing: pages/about.js
export default function About() {
    return <div>About Page</div>;
}

// Dynamic routes: pages/users/[id].js
export default function UserProfile({ user }) {
    return <div>User: {user.name}</div>;
}

// Pre-fetch data server-side
export async function getServerSideProps({ params }) {
    const user = await fetchUser(params.id);
    return { props: { user } };
}

// Link component optimizations
import Link from 'next/link';

function Navigation() {
    return (
        <nav>
            <Link href="/" prefetch={false}>Home</Link>
            <Link href="/about">About</Link> {/* Auto prefetch */}
        </nav>
    );
}
```

**Performance Comparison:**
```javascript
// React Router - Code splitting
const About = lazy(() => import('./pages/About'));
const Users = lazy(() => import('./pages/Users'));

function App() {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <Routes>
                <Route path="/about" element={<About />} />
                <Route path="/users" element={<Users />} />
            </Routes>
        </Suspense>
    );
}

// Next.js - Automatic code splitting
// Mỗi page tự động được split thành separate bundle
```

**Use Cases:**
- **React Router**: SPAs, dashboard applications, admin panels
- **Next.js**: Marketing websites, blogs, e-commerce, SEO-critical apps

---

### **6. Trình bày cách xử lý authentication trong Next.js (protected routes, middleware)**

**A: AUTHENTICATION TRONG NEXT.JS**

**1. JWT Authentication Setup:**
```javascript
// lib/auth.js
import jwt from 'jsonwebtoken';

export function generateToken(payload) {
    return jwt.sign(payload, process.env.JWT_SECRET, { expiresIn: '24h' });
}

export function verifyToken(token) {
    try {
        return jwt.verify(token, process.env.JWT_SECRET);
    } catch (error) {
        return null;
    }
}
```

**2. API Routes Authentication:**
```javascript
// pages/api/auth/login.js
export default async function handler(req, res) {
    if (req.method === 'POST') {
        const { email, password } = req.body;
        
        // Verify credentials
        const user = await authenticateUser(email, password);
        
        if (user) {
            const token = generateToken({ userId: user.id, email: user.email });
            
            // Set HTTP-only cookie
            res.setHeader('Set-Cookie', `token=${token}; HttpOnly; Path=/; Max-Age=86400; SameSite=Strict`);
            
            res.status(200).json({ success: true, user: { id: user.id, email: user.email } });
        } else {
            res.status(401).json({ error: 'Invalid credentials' });
        }
    }
}

// pages/api/auth/logout.js
export default function handler(req, res) {
    res.setHeader('Set-Cookie', 'token=; HttpOnly; Path=/; Max-Age=0');
    res.status(200).json({ success: true });
}
```

**3. Middleware for Route Protection (Next.js 12+):**
```javascript
// middleware.js
import { NextResponse } from 'next/server';
import { verifyToken } from './lib/auth';

export function middleware(request) {
    const token = request.cookies.get('token')?.value;
    const isAuthPage = request.nextUrl.pathname.startsWith('/auth');
    const isProtectedPage = request.nextUrl.pathname.startsWith('/dashboard');
    
    // Redirect authenticated users away from auth pages
    if (isAuthPage && token && verifyToken(token)) {
        return NextResponse.redirect(new URL('/dashboard', request.url));
    }
    
    // Protect dashboard routes
    if (isProtectedPage && (!token || !verifyToken(token))) {
        return NextResponse.redirect(new URL('/auth/login', request.url));
    }
    
    return NextResponse.next();
}

export const config = {
    matcher: ['/dashboard/:path*', '/auth/:path*']
};
```

**4. Auth Context Provider:**
```javascript
// contexts/AuthContext.js
import { createContext, useContext, useEffect, useState } from 'react';
import { useRouter } from 'next/router';

const AuthContext = createContext();

export function useAuth() {
    return useContext(AuthContext);
}

export function AuthProvider({ children }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    const router = useRouter();
    
    useEffect(() => {
        checkAuth();
    }, []);
    
    const checkAuth = async () => {
        try {
            const response = await fetch('/api/auth/me');
            if (response.ok) {
                const userData = await response.json();
                setUser(userData);
            }
        } catch (error) {
            console.error('Auth check failed:', error);
        } finally {
            setLoading(false);
        }
    };
    
    const login = async (email, password) => {
        const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password }),
        });
        
        if (response.ok) {
            const data = await response.json();
            setUser(data.user);
            router.push('/dashboard');
            return { success: true };
        }
        
        return { success: false, error: 'Login failed' };
    };
    
    const logout = async () => {
        await fetch('/api/auth/logout', { method: 'POST' });
        setUser(null);
        router.push('/');
    };
    
    return (
        <AuthContext.Provider value={{ user, login, logout, loading }}>
            {children}
        </AuthContext.Provider>
    );
}
```

**5. Protected Route Component:**
```javascript
// components/ProtectedRoute.js
import { useAuth } from '../contexts/AuthContext';
import { useRouter } from 'next/router';
import { useEffect } from 'react';

export default function ProtectedRoute({ children }) {
    const { user, loading } = useAuth();
    const router = useRouter();
    
    useEffect(() => {
        if (!loading && !user) {
            router.push('/auth/login');
        }
    }, [user, loading, router]);
    
    if (loading) {
        return <div>Loading...</div>;
    }
    
    if (!user) {
        return null;
    }
    
    return children;
}

// Usage in pages
export default function Dashboard() {
    return (
        <ProtectedRoute>
            <div>Dashboard content</div>
        </ProtectedRoute>
    );
}
```

**6. Server-Side Auth Check:**
```javascript
// utils/serverAuth.js
import { verifyToken } from '../lib/auth';

export function requireAuth(gssp) {
    return async (context) => {
        const token = context.req.cookies.token;
        
        if (!token || !verifyToken(token)) {
            return {
                redirect: {
                    destination: '/auth/login',
                    permanent: false,
                },
            };
        }
        
        return await gssp(context);
    };
}

// Usage
export const getServerSideProps = requireAuth(async (context) => {
    // Protected page logic
    const data = await fetchProtectedData();
    return { props: { data } };
});
```

---

### **7. Cách preload dữ liệu trong Next.js với getStaticProps, getServerSideProps, getInitialProps**

**A: DATA FETCHING METHODS**

**1. getStaticProps (Static Generation):**
```javascript
// pages/blog/[slug].js
export default function BlogPost({ post }) {
    return (
        <article>
            <h1>{post.title}</h1>
            <div dangerouslySetInnerHTML={{ __html: post.content }} />
        </article>
    );
}

export async function getStaticProps({ params }) {
    // Fetch data at build time
    const post = await fetchPost(params.slug);
    
    if (!post) {
        return { notFound: true };
    }
    
    return {
        props: { post },
        revalidate: 60, // ISR - regenerate every 60 seconds
    };
}

export async function getStaticPaths() {
    // Pre-generate popular posts
    const posts = await fetchPopularPosts();
    
    const paths = posts.map((post) => ({
        params: { slug: post.slug },
    }));
    
    return {
        paths,
        fallback: 'blocking', // Generate other posts on-demand
    };
}
```

**2. getServerSideProps (Server-Side Rendering):**
```javascript
// pages/dashboard.js
export default function Dashboard({ user, stats }) {
    return (
        <div>
            <h1>Welcome, {user.name}</h1>
            <Stats data={stats} />
        </div>
    );
}

export async function getServerSideProps({ req, res }) {
    // Run on every request
    const token = req.cookies.token;
    
    if (!token) {
        return {
            redirect: {
                destination: '/login',
                permanent: false,
            },
        };
    }
    
    try {
        const [user, stats] = await Promise.all([
            fetchUser(token),
            fetchUserStats(token),
        ]);
        
        // Set cache headers
        res.setHeader(
            'Cache-Control',
            'public, s-maxage=10, stale-while-revalidate=59'
        );
        
        return {
            props: { user, stats },
        };
    } catch (error) {
        return {
            redirect: {
                destination: '/login',
                permanent: false,
            },
        };
    }
}
```

**3. getInitialProps (Legacy - không khuyến khích):**
```javascript
// pages/legacy.js (Tránh dùng trong projects mới)
function LegacyPage({ data }) {
    return <div>{data.message}</div>;
}

LegacyPage.getInitialProps = async (ctx) => {
    // Runs on both server and client
    const { req, res, query } = ctx;
    
    if (req) {
        // Server-side
        const data = await fetchServerData();
        return { data };
    } else {
        // Client-side
        const data = await fetchClientData();
        return { data };
    }
};

export default LegacyPage;
```

**4. Client-Side Data Fetching (SWR/React Query):**
```javascript
// pages/posts.js
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then((res) => res.json());

export default function Posts() {
    const { data: posts, error, isLoading } = useSWR('/api/posts', fetcher, {
        refreshInterval: 30000, // Refresh every 30 seconds
        revalidateOnFocus: false,
    });
    
    if (error) return <div>Failed to load posts</div>;
    if (isLoading) return <div>Loading...</div>;
    
    return (
        <div>
            {posts.map((post) => (
                <PostCard key={post.id} post={post} />
            ))}
        </div>
    );
}
```

**5. Hybrid Approach:**
```javascript
// pages/products.js
export default function Products({ initialProducts }) {
    // Use initial data, then switch to client-side updates
    const { data: products = initialProducts } = useSWR('/api/products', fetcher, {
        fallbackData: initialProducts,
    });
    
    return (
        <div>
            {products.map((product) => (
                <ProductCard key={product.id} product={product} />
            ))}
        </div>
    );
}

// Pre-render with initial data
export async function getStaticProps() {
    const initialProducts = await fetchProducts();
    
    return {
        props: { initialProducts },
        revalidate: 300, // 5 minutes
    };
}
```

**Use Cases Summary:**
- **getStaticProps**: Blogs, documentation, marketing pages
- **getServerSideProps**: User dashboards, personalized content
- **Client-side**: Interactive features, real-time updates
- **Hybrid**: Best of both worlds

---

### **8. Bạn sẽ làm gì để tối ưu SEO trên ứng dụng Next.js?**

**A: SEO OPTIMIZATION TRONG NEXT.JS**

**1. Meta Tags và Head Management:**
```javascript
// components/SEOHead.js
import Head from 'next/head';

export default function SEOHead({ 
    title, 
    description, 
    keywords, 
    image,
    url,
    type = 'website' 
}) {
    const siteTitle = 'My Awesome Site';
    const fullTitle = title ? `${title} | ${siteTitle}` : siteTitle;
    
    return (
        <Head>
            {/* Basic Meta Tags */}
            <title>{fullTitle}</title>
            <meta name="description" content={description} />
            <meta name="keywords" content={keywords} />
            <meta name="author" content="Your Name" />
            <meta name="viewport" content="width=device-width, initial-scale=1" />
            
            {/* Open Graph */}
            <meta property="og:title" content={fullTitle} />
            <meta property="og:description" content={description} />
            <meta property="og:image" content={image} />
            <meta property="og:url" content={url} />
            <meta property="og:type" content={type} />
            <meta property="og:site_name" content={siteTitle} />
            
            {/* Twitter Cards */}
            <meta name="twitter:card" content="summary_large_image" />
            <meta name="twitter:title" content={fullTitle} />
            <meta name="twitter:description" content={description} />
            <meta name="twitter:image" content={image} />
            
            {/* Canonical URL */}
            <link rel="canonical" href={url} />
            
            {/* Favicon */}
            <link rel="icon" href="/favicon.ico" />
            <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
        </Head>
    );
}

// Usage in pages
export default function BlogPost({ post }) {
    return (
        <>
            <SEOHead
                title={post.title}
                description={post.excerpt}
                image={post.featuredImage}
                url={`https://mysite.com/blog/${post.slug}`}
                type="article"
            />
            <article>
                <h1>{post.title}</h1>
                <div>{post.content}</div>
            </article>
        </>
    );
}
```

**2. Structured Data (JSON-LD):**
```javascript
// components/StructuredData.js
export function ArticleStructuredData({ article }) {
    const structuredData = {
        "@context": "https://schema.org",
        "@type": "Article",
        "headline": article.title,
        "description": article.excerpt,
        "image": article.featuredImage,
        "author": {
            "@type": "Person",
            "name": article.author.name
        },
        "publisher": {
            "@type": "Organization",
            "name": "My Site",
            "logo": {
                "@type": "ImageObject",
                "url": "https://mysite.com/logo.png"
            }
        },
        "datePublished": article.publishedAt,
        "dateModified": article.updatedAt
    };
    
    return (
        <script
            type="application/ld+json"
            dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
        />
    );
}

// Usage
export default function BlogPost({ post }) {
    return (
        <>
            <ArticleStructuredData article={post} />
            <SEOHead {...post} />
            <article>{/* content */}</article>
        </>
    );
}
```

**3. Image Optimization:**
```javascript
import Image from 'next/image';

export default function OptimizedImages() {
    return (
        <div>
            {/* Automatic optimization */}
            <Image
                src="/hero-image.jpg"
                alt="Hero image description"
                width={800}
                height={400}
                priority // Load immediately for above-fold images
                placeholder="blur"
                blurDataURL="data:image/jpeg;base64,..."
            />
            
            {/* Responsive images */}
            <Image
                src="/responsive-image.jpg"
                alt="Responsive image"
                fill
                sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
                style={{ objectFit: 'cover' }}
            />
        </div>
    );
}
```

**4. Sitemap Generation:**
```javascript
// scripts/generate-sitemap.js
const fs = require('fs');
const globby = require('globby');

async function generateSitemap() {
    const pages = await globby([
        'pages/**/*.js',
        '!pages/_*.js',
        '!pages/api',
    ]);
    
    // Fetch dynamic routes
    const posts = await fetchAllPosts();
    const products = await fetchAllProducts();
    
    const sitemap = `
        <?xml version="1.0" encoding="UTF-8"?>
        <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
            ${pages.map((page) => {
                const path = page
                    .replace('pages', '')
                    .replace('.js', '')
                    .replace('/index', '');
                const route = path === '/index' ? '' : path;
                
                return `
                    <url>
                        <loc>https://mysite.com${route}</loc>
                        <lastmod>${new Date().toISOString()}</lastmod>
                        <changefreq>weekly</changefreq>
                        <priority>0.8</priority>
                    </url>
                `;
            }).join('')}
            
            ${posts.map((post) => `
                <url>
                    <loc>https://mysite.com/blog/${post.slug}</loc>
                    <lastmod>${post.updatedAt}</lastmod>
                    <changefreq>monthly</changefreq>
                    <priority>0.6</priority>
                </url>
            `).join('')}
        </urlset>
    `;
    
    fs.writeFileSync('public/sitemap.xml', sitemap);
}

generateSitemap();
```

**5. robots.txt:**
```javascript
// pages/api/robots.js
export default function handler(req, res) {
    res.setHeader('Content-Type', 'text/plain');
    res.write(`
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /api/

Sitemap: https://mysite.com/sitemap.xml
    `);
    res.end();
}
```

**6. Core Web Vitals Optimization:**
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
    // Enable experimental features for better performance
    experimental: {
        optimizeCss: true,
        optimizePackageImports: ['@mui/material', '@mui/icons-material'],
    },
    
    // Image optimization
    images: {
        formats: ['image/webp', 'image/avif'],
        deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
        imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    },
    
    // Compression
    compress: true,
    
    // Headers for caching
    async headers() {
        return [
            {
                source: '/images/:path*',
                headers: [
                    {
                        key: 'Cache-Control',
                        value: 'public, max-age=31536000, immutable',
                    },
                ],
            },
        ];
    },
};

module.exports = nextConfig;
```

**7. Performance Monitoring:**
```javascript
// lib/analytics.js
export function reportWebVitals(metric) {
    const { id, name, value, label } = metric;
    
    // Send to analytics service
    gtag('event', name, {
        event_category: label === 'web-vital' ? 'Web Vitals' : 'Next.js custom metric',
        value: Math.round(name === 'CLS' ? value * 1000 : value),
        event_label: id,
        non_interaction: true,
    });
}

// pages/_app.js
export { reportWebVitals } from '../lib/analytics';
```

**SEO Checklist:**
- ✅ Proper meta tags và Open Graph
- ✅ Structured data (JSON-LD)
- ✅ Image optimization với alt tags
- ✅ Sitemap.xml tự động generate
- ✅ robots.txt configuration
- ✅ Core Web Vitals optimization
- ✅ Mobile-first responsive design
- ✅ Semantic HTML structure
- ✅ Internal linking strategy
- ✅ Page loading performance

---

**🎯 TÓM TẮT FRONTEND BEST PRACTICES**

1. **Performance First**: Virtualization, memoization, code splitting
2. **SEO Optimization**: Meta tags, structured data, Core Web Vitals
3. **User Experience**: Smooth navigation, loading states, error boundaries
4. **Code Quality**: Proper keys, stable references, clean architecture
5. **Security**: Protected routes, input validation, XSS prevention
