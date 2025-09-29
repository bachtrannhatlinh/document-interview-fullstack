# CÂU TRẢ LỜI CHI TIẾT - PHỎNG VẤN IGNISOFT FRONT-END DEVELOPER

## 🔥 TECHNICAL QUESTIONS & ANSWERS

### **1. REACT/NEXT.JS QUESTIONS**

#### **Q: "Giải thích sự khác biệt giữa SSG, SSR và CSR trong Next.js?"**

**A: TRẢ LỜI CHI TIẾT**

```
SSG (Static Site Generation):
✅ Tạo HTML tại build time
✅ Performance cao nhất - files tĩnh
✅ SEO tuyệt vời
✅ CDN friendly
❌ Data không real-time
❌ Rebuild cần thiết cho updates

Use cases: Blog, documentation, marketing pages

SSR (Server-Side Rendering):
✅ HTML tạo mỗi request
✅ Data luôn fresh
✅ SEO tốt
✅ Personalized content
❌ Slower than SSG
❌ Server load cao

Use cases: User dashboard, e-commerce product pages

CSR (Client-Side Rendering):
✅ Interactive, dynamic
✅ Rich user experience
✅ Reduced server load
❌ SEO challenges
❌ Slower initial load
❌ JavaScript required

Use cases: Admin panels, complex web apps

ISR (Incremental Static Regeneration):
✅ Best of SSG + fresh data
✅ Automatic background updates
✅ High performance
Use cases: E-commerce catalogs, news sites
```

**Code Example:**
```javascript
// SSG - getStaticProps
export async function getStaticProps() {
  const posts = await fetchPosts();
  return {
    props: { posts },
    revalidate: 60 // ISR - regenerate every 60s
  };
}

// SSR - getServerSideProps  
export async function getServerSideProps({ req }) {
  const user = await getCurrentUser(req.cookies.token);
  if (!user) {
    return { redirect: { destination: '/login' } };
  }
  return { props: { user } };
}

// CSR - useEffect
function Dashboard() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/dashboard')
      .then(res => res.json())
      .then(setData);
  }, []);
  
  if (!data) return <Loading />;
  return <DashboardContent data={data} />;
}
```

---

#### **Q: "Khi nào dùng useCallback vs useMemo?"**

**A: TRẢ LỜI CHI TIẾT**

```
useCallback - Function Memoization:
✅ Khi function được pass to child components
✅ Dependency trong useEffect
✅ Prevent unnecessary re-renders
❌ Don't overuse - có cost của comparison

useMemo - Value Memoization:
✅ Expensive calculations
✅ Complex object/array creation  
✅ Derived state computations
❌ Simple calculations không cần thiết
```

**Code Example:**
```javascript
function Parent({ items }) {
  const [filter, setFilter] = useState('');
  
  // ✅ useCallback - function passed to child
  const handleItemClick = useCallback((id) => {
    // Complex logic here
    updateItem(id);
  }, [updateItem]);
  
  // ✅ useMemo - expensive calculation
  const filteredItems = useMemo(() => {
    return items.filter(item => {
      // Complex filtering logic
      return expensiveFilter(item, filter);
    });
  }, [items, filter]);
  
  // ❌ Không cần thiết - simple calculation
  const itemCount = useMemo(() => items.length, [items]);
  
  return (
    <div>
      {filteredItems.map(item => (
        <ChildComponent 
          key={item.id}
          item={item}
          onClick={handleItemClick} // Stable reference
        />
      ))}
    </div>
  );
}

const ChildComponent = React.memo(({ item, onClick }) => {
  return (
    <div onClick={() => onClick(item.id)}>
      {item.name}
    </div>
  );
});
```

---

#### **Q: "Cách optimize large list rendering?"**

**A: TRẢ LỜI CHI TIẾT**

**1. Virtualization với React Window:**
```javascript
import { FixedSizeList as List } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <ItemComponent data={items[index]} />
    </div>
  );

  return (
    <List
      height={400}
      itemCount={items.length}
      itemSize={60}
      width="100%"
    >
      {Row}
    </List>
  );
}
```

**2. Pagination Strategy:**
```javascript
function PaginatedList({ items }) {
  const [currentPage, setCurrentPage] = useState(1);
  const itemsPerPage = 50;
  
  const paginatedItems = useMemo(() => {
    const start = (currentPage - 1) * itemsPerPage;
    const end = start + itemsPerPage;
    return items.slice(start, end);
  }, [items, currentPage, itemsPerPage]);
  
  return (
    <div>
      <ItemGrid items={paginatedItems} />
      <Pagination 
        current={currentPage}
        total={Math.ceil(items.length / itemsPerPage)}
        onChange={setCurrentPage}
      />
    </div>
  );
}
```

**3. Infinite Scroll với Intersection Observer:**
```javascript
function InfiniteList({ fetchMore, hasMore }) {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);
  const lastElementRef = useRef();

  const observer = useCallback(node => {
    if (loading) return;
    if (observer.current) observer.current.disconnect();
    
    observer.current = new IntersectionObserver(entries => {
      if (entries[0].isIntersecting && hasMore) {
        loadMore();
      }
    });
    
    if (node) observer.current.observe(node);
  }, [loading, hasMore]);

  const loadMore = async () => {
    setLoading(true);
    const newItems = await fetchMore();
    setItems(prev => [...prev, ...newItems]);
    setLoading(false);
  };

  return (
    <div>
      {items.map((item, index) => (
        <div
          key={item.id}
          ref={index === items.length - 1 ? observer : null}
        >
          <ItemComponent item={item} />
        </div>
      ))}
      {loading && <LoadingSpinner />}
    </div>
  );
}
```

---

### **2. API INTEGRATION QUESTIONS**

#### **Q: "Cách handle API errors trong React app?"**

**A: TRẢ LỜI CHI TIẾT**

**1. Error Boundary Pattern:**
```javascript
class APIErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error reporting service
    console.error('API Error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

**2. Custom Hook for API calls:**
```javascript
function useApi(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${getToken()}`,
          ...options.headers
        },
        ...options
      });

      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}));
        throw new APIError(
          errorData.message || 'Request failed',
          response.status,
          errorData
        );
      }

      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err);
      
      // Handle specific error types
      if (err.status === 401) {
        // Redirect to login
        window.location.href = '/login';
      } else if (err.status === 403) {
        // Show permission denied
        showNotification('Access denied', 'error');
      } else if (err.status >= 500) {
        // Server error
        showNotification('Server error, please try again', 'error');
      }
    } finally {
      setLoading(false);
    }
  }, [url, options]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}

class APIError extends Error {
  constructor(message, status, data) {
    super(message);
    this.status = status;
    this.data = data;
  }
}
```

**3. Global Error Handler:**
```javascript
// API client setup
const apiClient = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
  timeout: 10000,
});

apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const { response, config } = error;
    
    if (response?.status === 401 && !config._retry) {
      config._retry = true;
      
      try {
        const newToken = await refreshToken();
        config.headers.Authorization = `Bearer ${newToken}`;
        return apiClient.request(config);
      } catch (refreshError) {
        // Redirect to login
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    return Promise.reject(error);
  }
);
```

---

#### **Q: "JWT authentication workflow?"**

**A: TRẢ LỜI CHI TIẾT**

**Complete Authentication Flow:**

```javascript
// 1. Authentication Context
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // Initialize auth state
  useEffect(() => {
    const initializeAuth = async () => {
      const token = localStorage.getItem('token');
      if (token) {
        try {
          const userData = await verifyToken(token);
          setUser(userData);
        } catch {
          localStorage.removeItem('token');
        }
      }
      setLoading(false);
    };
    
    initializeAuth();
  }, []);

  const login = async (email, password) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });

      if (!response.ok) {
        throw new Error('Invalid credentials');
      }

      const { user, token } = await response.json();
      
      // Store token
      localStorage.setItem('token', token);
      setUser(user);
      
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

// 2. Protected Route Component
function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();
  
  if (loading) return <LoadingSpinner />;
  if (!user) return <Navigate to="/login" replace />;
  
  return children;
}

// 3. Login Component
function LoginForm() {
  const [credentials, setCredentials] = useState({ email: '', password: '' });
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const { login } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');

    const result = await login(credentials.email, credentials.password);
    
    if (result.success) {
      navigate('/dashboard');
    } else {
      setError(result.error);
    }
    
    setLoading(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <div className="error">{error}</div>}
      
      <input
        type="email"
        value={credentials.email}
        onChange={(e) => setCredentials(prev => ({
          ...prev,
          email: e.target.value
        }))}
        required
      />
      
      <input
        type="password"
        value={credentials.password}
        onChange={(e) => setCredentials(prev => ({
          ...prev,
          password: e.target.value
        }))}
        required
      />
      
      <button type="submit" disabled={loading}>
        {loading ? 'Signing in...' : 'Sign In'}
      </button>
    </form>
  );
}

// 4. Token Refresh Logic
async function setupTokenRefresh() {
  const token = localStorage.getItem('token');
  if (!token) return;

  try {
    const payload = JSON.parse(atob(token.split('.')[1]));
    const expirationTime = payload.exp * 1000;
    const currentTime = Date.now();
    const timeUntilExpiration = expirationTime - currentTime;

    // Refresh 5 minutes before expiration
    const refreshTime = timeUntilExpiration - 5 * 60 * 1000;

    if (refreshTime > 0) {
      setTimeout(async () => {
        try {
          const response = await fetch('/api/auth/refresh', {
            method: 'POST',
            headers: {
              'Authorization': `Bearer ${token}`
            }
          });

          if (response.ok) {
            const { token: newToken } = await response.json();
            localStorage.setItem('token', newToken);
            setupTokenRefresh(); // Setup next refresh
          } else {
            // Token refresh failed, logout user
            logout();
          }
        } catch (error) {
          logout();
        }
      }, refreshTime);
    }
  } catch (error) {
    // Invalid token format
    logout();
  }
}
```

---

### **3. PERFORMANCE QUESTIONS**

#### **Q: "Cách optimize Core Web Vitals?"**

**A: TRẢ LỜI CHI TIẾT**

**1. Largest Contentful Paint (LCP) - < 2.5s:**

```javascript
// Image optimization
import Image from 'next/image';

function HeroSection() {
  return (
    <div>
      <Image
        src="/hero-image.jpg"
        alt="Hero"
        width={1200}
        height={600}
        priority // Load immediately
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..."
      />
    </div>
  );
}

// Font optimization
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // Avoid invisible text during font load
});

// Preload critical resources
export default function Layout({ children }) {
  return (
    <Head>
      <link
        rel="preload"
        href="/critical-resource.css"
        as="style"
      />
      <link
        rel="preconnect"
        href="https://api.example.com"
      />
    </Head>
  );
}
```

**2. First Input Delay (FID) - < 100ms:**

```javascript
// Code splitting để giảm JavaScript bundle
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <div>
      <Header />
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}

// Debounce user inputs
function SearchInput() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const debouncedSearch = useMemo(
    () => debounce(async (searchTerm) => {
      if (searchTerm) {
        const data = await searchAPI(searchTerm);
        setResults(data);
      }
    }, 300),
    []
  );

  useEffect(() => {
    debouncedSearch(query);
  }, [query, debouncedSearch]);

  return (
    <input
      type="text"
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

**3. Cumulative Layout Shift (CLS) - < 0.1:**

```javascript
// Reserve space for dynamic content
function DynamicContent() {
  const [content, setContent] = useState(null);

  return (
    <div style={{ minHeight: '200px' }}> {/* Reserve space */}
      {content ? (
        <div>{content}</div>
      ) : (
        <div className="skeleton-loader" style={{ height: '200px' }} />
      )}
    </div>
  );
}

// Use aspect ratio for responsive images
.responsive-image {
  aspect-ratio: 16 / 9;
  width: 100%;
  height: auto;
}

// Avoid inserting content above existing content
function NewsletterSignup() {
  const [isVisible, setIsVisible] = useState(false);

  return (
    <div>
      <main>
        {/* Main content */}
      </main>
      
      {/* Newsletter appears at bottom, not top */}
      {isVisible && (
        <div className="newsletter-popup">
          Newsletter signup form
        </div>
      )}
    </div>
  );
}
```

---

## 🏗️ SYSTEM DESIGN QUESTIONS & ANSWERS

### **Q: "Thiết kế kiến trúc cho E-commerce website với Next.js?"**

**A: SYSTEM DESIGN CHI TIẾT**

```
┌─────────────────────────────────────────────────────────────┐
│                        CDN Layer                             │
│  ┌───────────┐  ┌───────────┐  ┌─────────────┐             │
│  │  Static   │  │   Images  │  │    Fonts    │             │
│  │   Assets  │  │           │  │             │             │
│  └───────────┘  └───────────┘  └─────────────┘             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Next.js Frontend                         │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐     │
│  │     SSG     │  │     SSR     │  │       CSR       │     │
│  │             │  │             │  │                 │     │
│  │ - Product   │  │ - User      │  │ - Shopping Cart │     │
│  │   Catalog   │  │   Dashboard │  │ - Checkout Flow │     │
│  │ - SEO Pages │  │ - Profile   │  │ - Real-time     │     │
│  │ - Blog      │  │ - Orders    │  │   Features      │     │
│  └─────────────┘  └─────────────┘  └─────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway                            │
│                   (Rate Limiting, Auth)                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Microservices                            │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐     │
│  │   Product   │  │    User     │  │     Payment     │     │
│  │   Service   │  │   Service   │  │    Service      │     │
│  └─────────────┘  └─────────────┘  └─────────────────┘     │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐     │
│  │   Order     │  │ Inventory   │  │ Notification    │     │
│  │   Service   │  │  Service    │  │   Service       │     │
│  └─────────────┘  └─────────────┘  └─────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Data Layer                              │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐     │
│  │ PostgreSQL  │  │    Redis    │  │   Elasticsearch │     │
│  │ (Products,  │  │   (Cache,   │  │    (Search)     │     │
│  │ Users,      │  │   Sessions) │  │                 │     │
│  │ Orders)     │  │             │  │                 │     │
│  └─────────────┘  └─────────────┘  └─────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

**Frontend Architecture Implementation:**

```javascript
// 1. Product Catalog (SSG)
// pages/products/[category]/[slug].js
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.slug);
  
  if (!product) {
    return { notFound: true };
  }

  return {
    props: { product },
    revalidate: 3600, // ISR - update hourly
  };
}

export async function getStaticPaths() {
  const products = await fetchPopularProducts();
  
  const paths = products.map((product) => ({
    params: { 
      category: product.category,
      slug: product.slug 
    },
  }));

  return {
    paths,
    fallback: 'blocking', // Generate other products on-demand
  };
}

// 2. User Dashboard (SSR)
// pages/dashboard.js
export async function getServerSideProps({ req, res }) {
  const token = req.cookies.authToken;
  
  if (!token) {
    return {
      redirect: { destination: '/login', permanent: false }
    };
  }

  try {
    const [user, orders, recommendations] = await Promise.all([
      fetchUserProfile(token),
      fetchUserOrders(token),
      fetchRecommendations(token)
    ]);

    // Cache user data
    res.setHeader('Cache-Control', 'private, max-age=300');

    return {
      props: { user, orders, recommendations }
    };
  } catch (error) {
    return {
      redirect: { destination: '/login', permanent: false }
    };
  }
}

// 3. Shopping Cart (CSR with State Management)
// contexts/CartContext.js
const CartContext = createContext();

export function CartProvider({ children }) {
  const [cart, setCart] = useState([]);
  const [isLoading, setIsLoading] = useState(false);

  // Load cart from localStorage/API
  useEffect(() => {
    const savedCart = localStorage.getItem('cart');
    if (savedCart) {
      setCart(JSON.parse(savedCart));
    }
  }, []);

  // Sync cart to localStorage
  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(cart));
  }, [cart]);

  const addToCart = (product, quantity = 1) => {
    setCart(prev => {
      const existingItem = prev.find(item => item.id === product.id);
      
      if (existingItem) {
        return prev.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + quantity }
            : item
        );
      }
      
      return [...prev, { ...product, quantity }];
    });
  };

  const removeFromCart = (productId) => {
    setCart(prev => prev.filter(item => item.id !== productId));
  };

  const updateQuantity = (productId, quantity) => {
    if (quantity <= 0) {
      removeFromCart(productId);
      return;
    }

    setCart(prev =>
      prev.map(item =>
        item.id === productId
          ? { ...item, quantity }
          : item
      )
    );
  };

  const getTotalPrice = () => {
    return cart.reduce((total, item) => total + (item.price * item.quantity), 0);
  };

  const getItemCount = () => {
    return cart.reduce((total, item) => total + item.quantity, 0);
  };

  return (
    <CartContext.Provider value={{
      cart,
      addToCart,
      removeFromCart,
      updateQuantity,
      getTotalPrice,
      getItemCount,
      isLoading
    }}>
      {children}
    </CartContext.Provider>
  );
}

// 4. Search Implementation
// components/ProductSearch.js
function ProductSearch() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [filters, setFilters] = useState({
    category: '',
    priceRange: [0, 1000],
    rating: 0
  });

  const debouncedSearch = useMemo(
    () => debounce(async (searchQuery, searchFilters) => {
      setLoading(true);
      
      try {
        const response = await fetch('/api/search', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            query: searchQuery,
            filters: searchFilters,
            limit: 20
          })
        });

        const data = await response.json();
        setResults(data.products);
      } catch (error) {
        console.error('Search error:', error);
      } finally {
        setLoading(false);
      }
    }, 300),
    []
  );

  useEffect(() => {
    if (query.trim()) {
      debouncedSearch(query, filters);
    } else {
      setResults([]);
    }
  }, [query, filters, debouncedSearch]);

  return (
    <div className="search-container">
      <SearchInput
        value={query}
        onChange={setQuery}
        placeholder="Search products..."
      />
      
      <SearchFilters
        filters={filters}
        onChange={setFilters}
      />

      {loading ? (
        <SearchSkeleton />
      ) : (
        <SearchResults results={results} />
      )}
    </div>
  );
}
```

**Performance Optimizations:**

```javascript
// 1. Image Optimization
import Image from 'next/image';

const productImageLoader = ({ src, width, quality }) => {
  return `https://cdn.example.com/${src}?w=${width}&q=${quality || 75}`;
};

function ProductImage({ src, alt, ...props }) {
  return (
    <Image
      loader={productImageLoader}
      src={src}
      alt={alt}
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 25vw"
      {...props}
    />
  );
}

// 2. Code Splitting by Route
const CheckoutPage = dynamic(() => import('./pages/checkout'), {
  loading: () => <CheckoutSkeleton />,
  ssr: false // CSR only for checkout
});

// 3. API Response Caching
// pages/api/products/[id].js
export default async function handler(req, res) {
  const { id } = req.query;
  
  // Set cache headers
  res.setHeader('Cache-Control', 's-maxage=3600, stale-while-revalidate');
  
  try {
    const product = await getProduct(id);
    
    if (!product) {
      return res.status(404).json({ error: 'Product not found' });
    }
    
    res.status(200).json(product);
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
}

// 4. Bundle Analysis & Optimization
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  experimental: {
    optimizeCss: true,
  },
  images: {
    domains: ['cdn.example.com'],
    formats: ['image/webp', 'image/avif'],
  },
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          }
        ],
      },
    ];
  },
});
```

---

## 🎯 BEHAVIORAL QUESTIONS & ANSWERS

### **Q: "Describe a challenging bug you fixed"**

**A: STAR METHOD**

**Situation:**
"Trong dự án e-commerce trước đây, chúng tôi gặp phải một bug nghiêm trọng về memory leak trong trang product listing. Khi users scroll qua nhiều products, ứng dụng ngày càng chậm và cuối cùng crash browser."

**Task:**
"Tôi được giao nhiệm vụ investigate và fix bug này trong vòng 2 ngày vì nó đang ảnh hưởng đến user experience và conversion rate."

**Action:**
```
1. Root Cause Analysis:
   - Sử dụng Chrome DevTools Memory tab để profile memory usage
   - Phát hiện event listeners không được cleanup
   - IntersectionObserver instances bị duplicate

2. Investigation Process:
   - Code review toàn bộ product listing components
   - Identify useEffect dependencies không đúng
   - Event listeners được tạo mới mỗi re-render

3. Solution Implementation:
   - Implement proper cleanup functions
   - Fix useEffect dependencies
   - Optimize component re-rendering với React.memo()
```

**Code Example:**
```javascript
// Before (Memory Leak)
function ProductCard({ product }) {
  useEffect(() => {
    const observer = new IntersectionObserver(callback);
    observer.observe(elementRef.current);
    // Missing cleanup - Memory Leak!
  }, [product]); // Wrong dependency
}

// After (Fixed)
function ProductCard({ product }) {
  const elementRef = useRef();
  
  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const observer = new IntersectionObserver(callback);
    observer.observe(element);

    // Proper cleanup
    return () => {
      observer.unobserve(element);
      observer.disconnect();
    };
  }, []); // Correct dependencies
}
```

**Result:**
"Bug được fix hoàn toàn, memory usage giảm 70%, page load speed cải thiện 40%. Team học được importance của proper cleanup và memory profiling."

---

### **Q: "How do you handle code reviews?"**

**A: CODE REVIEW APPROACH**

**My Code Review Philosophy:**

```
1. Focus on Code, Not Person:
   ✅ "Consider using useMemo here for performance"
   ❌ "You always forget about performance"

2. Be Constructive và Specific:
   ✅ "This could cause re-renders. Try moving this outside the component"
   ❌ "This is wrong"

3. Suggest Improvements:
   ✅ "What about using React Query for this API call? It handles caching automatically"

4. Ask Questions để Understand:
   ✅ "Can you explain the reasoning behind this pattern?"

5. Praise Good Practices:
   ✅ "Great error handling implementation!"
```

**Practical Example:**
```javascript
// Code Review Comment Example
/*
💡 Performance Suggestion:
This effect runs on every render because `users` is recreated each time. 
Consider memoizing it:

const memoizedUsers = useMemo(() => 
  users.filter(user => user.active), [users]
);

This will prevent unnecessary re-computations.

📚 Reference: https://react.dev/reference/react/useMemo
*/

// Security Review Example  
/*
⚠️ Security Concern:
Direct innerHTML injection here could lead to XSS attacks.
Consider using DOMPurify or switching to a safe alternative:

import DOMPurify from 'dompurify';

const sanitizedHTML = DOMPurify.sanitize(userContent);
*/
```

**My Review Process:**
1. **Functionality First** - Does it work correctly?
2. **Performance** - Any optimization opportunities?
3. **Security** - Input validation, XSS prevention
4. **Maintainability** - Code clarity, documentation
5. **Testing** - Edge cases covered?

---

### **Q: "How do you stay updated with React ecosystem?"**

**A: CONTINUOUS LEARNING STRATEGY**

**Daily Learning Routine:**

```
1. Official Sources:
   - React.dev blog và documentation updates
   - Next.js blog for framework updates
   - TypeScript release notes

2. Community Resources:
   - Reddit r/reactjs for discussions
   - React Newsletter weekly digest
   - This Week in React by Dan Abramov

3. Technical Deep Dives:
   - Kent C. Dodds blog và courses
   - React Working Group discussions
   - RFC documents for upcoming features

4. Practical Application:
   - Build side projects with new features
   - Contribute to open source projects
   - Participate in hackathons
```

**Recent Learning Examples:**
```javascript
// 1. React Server Components (practiced with Next.js 13)
async function ServerComponent() {
  const data = await fetch('https://api.example.com/data');
  return <div>{data.title}</div>;
}

// 2. useId hook for accessibility
function FormField() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Name:</label>
      <input id={id} type="text" />
    </>
  );
}

// 3. Concurrent Rendering with Suspense
function App() {
  return (
    <Suspense fallback={<Loading />}>
      <UserProfile />
    </Suspense>
  );
}
```

**Learning Projects Portfolio:**
1. **E-commerce with Next.js 13** - App Router, Server Components
2. **Real-time Chat** - WebSocket, optimistic updates
3. **Performance Dashboard** - Data visualization, virtualization
4. **PWA Blog** - Service Workers, offline functionality

**Knowledge Sharing:**
- Write technical blog posts
- Present findings to team
- Mentor junior developers
- Create internal documentation

---

## 💡 ADDITIONAL QUESTIONS & ANSWERS

### **Q: "Explain React Fiber and Reconciliation"**

**A:**
```
React Fiber là complete rewrite của React's reconciliation algorithm:

Key Benefits:
✅ Incremental Rendering - Chia nhỏ rendering work
✅ Pausable Work - Có thể pause và resume rendering
✅ Priority-based Updates - High priority updates được xử lý trước
✅ Better Error Boundaries - Improved error handling

How it Works:
1. Work Loop: React processes work trong small chunks
2. Priority Queue: Updates được prioritized
3. Time Slicing: Yields control back to browser
4. Concurrent Mode: Multiple state updates concurrently

Practical Impact:
- Smoother animations
- Better user experience
- Responsive UI during heavy computations
```

### **Q: "Difference between Next.js App Router vs Pages Router"**

**A:**
```
App Router (Next.js 13+):
✅ File-based routing trong app/ directory
✅ React Server Components by default
✅ Nested layouts và templates
✅ Streaming support với Suspense
✅ Better TypeScript support

Pages Router (Legacy):
✅ Simpler mental model
✅ More tutorials và examples available
✅ Stable và mature
✅ Client Components by default

Migration Strategy:
- Có thể dùng both routers simultaneously
- Migrate incrementally page by page
- App Router for new features
```

### **Q: "How to implement optimistic updates?"**

**A:**
```javascript
function useOptimisticUpdate() {
  const [items, setItems] = useState([]);
  const [optimisticItems, setOptimisticItems] = useState([]);

  const addItem = async (newItem) => {
    // Optimistic update
    const tempId = `temp-${Date.now()}`;
    const optimisticItem = { ...newItem, id: tempId, pending: true };
    
    setOptimisticItems(prev => [...prev, optimisticItem]);

    try {
      // API call
      const savedItem = await createItem(newItem);
      
      // Replace optimistic item with real item
      setItems(prev => [...prev, savedItem]);
      setOptimisticItems(prev => 
        prev.filter(item => item.id !== tempId)
      );
    } catch (error) {
      // Rollback optimistic update
      setOptimisticItems(prev => 
        prev.filter(item => item.id !== tempId)
      );
      
      showErrorMessage('Failed to add item');
    }
  };

  const displayItems = [...items, ...optimisticItems];

  return { items: displayItems, addItem };
}
```

Với bộ câu trả lời này, bạn sẽ có thể tự tin trả lời các câu hỏi phỏng vấn Frontend Developer tại Ignisoft! 🚀
