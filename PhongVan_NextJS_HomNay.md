# CÂU HỎI PHỎNG VẤN NEXTJS - HÔM NAY

*Tổng hợp các câu hỏi và câu trả lời phỏng vấn NextJS*

---

## 1. ƯU ĐIỂM CỦA SSR (SERVER-SIDE RENDERING)

### ✅ Ưu điểm chính:

#### **1.1 SEO Xuất Sắc**
- HTML được render sẵn trên server
- Search engines (Google, Bing) crawl được nội dung ngay lập tức
- Không phụ thuộc vào JavaScript để render content
- Meta tags đầy đủ cho social sharing (Open Graph, Twitter Cards)

#### **1.2 Performance - First Contentful Paint (FCP)**
- User thấy nội dung ngay khi page load
- Không cần chờ JavaScript download + execute
- Tốt cho slow networks và low-end devices

#### **1.3 Fresh Data Mỗi Request**
- Phù hợp với user-specific content (dashboard, profile)
- Personalized data (recommendations, cart)
- Real-time data requirements
- Authentication-based content

#### **1.4 Social Media Sharing**
- Meta tags được render đúng cho preview cards
- Facebook, Twitter, LinkedIn hiển thị preview đẹp
- Dynamic OG images based on content

### ❌ Trade-offs cần lưu ý:

- **Server load cao hơn**: Mỗi request phải render trên server
- **TTFB chậm hơn SSG**: Time To First Byte cao hơn static pages
- **Hosting cost**: Cần serverless functions hoặc Node.js server
- **Scalability**: Cần caching strategy tốt (Redis, CDN)

### 📝 Ví dụ implementation:

```javascript
// pages/dashboard.js
export async function getServerSideProps({ req, res, query }) {
  // Set cache headers
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=10, stale-while-revalidate=59'
  );

  const token = req.cookies.token;
  
  // Redirect nếu chưa login
  if (!token) {
    return {
      redirect: {
        destination: '/login',
        permanent: false
      }
    };
  }

  // Fetch user-specific data
  const userData = await fetchUserData(token);
  
  return {
    props: { 
      userData,
      timestamp: Date.now() // Fresh data mỗi request
    }
  };
}

function Dashboard({ userData }) {
  return (
    <div>
      <h1>Welcome, {userData.name}</h1>
      <p>Your personalized content here</p>
    </div>
  );
}

export default Dashboard;
```

### 🎯 Khi nào dùng SSR?

✅ **Nên dùng:**
- User dashboards, profiles
- Personalized recommendations
- E-commerce cart, checkout
- Real-time data feeds
- SEO + Dynamic content

❌ **Không nên dùng:**
- Static content (blog, marketing pages) → dùng SSG
- Highly interactive apps → dùng CSR
- Public product listings → dùng ISR

---

## 2. TỐI ƯU LIST ẢNH .PNG/.JPG VỚI KÍCH THƯỚC KHÁC NHAU

### 🖼️ Giải pháp: Next.js Image Component

#### **2.1 Sử dụng `next/image`**

```jsx
import Image from 'next/image';

function ProductGallery({ images }) {
  return (
    <div className="gallery">
      {images.map((image) => (
        <Image
          key={image.id}
          src={image.url}
          alt={image.alt}
          width={500}
          height={500}
          sizes="(max-width: 768px) 100vw, 
                 (max-width: 1200px) 50vw, 
                 33vw"
          placeholder="blur"
          blurDataURL={image.blurDataURL}
          quality={85}
          priority={image.priority} // Cho ảnh above-the-fold
        />
      ))}
    </div>
  );
}
```

### ⚡ Tính năng tự động của next/image:

#### **1. Automatic Format Conversion**
- Tự động convert sang WebP, AVIF (nếu browser support)
- Fallback về JPEG/PNG cho old browsers
- Giảm 30-50% file size so với PNG/JPG

#### **2. Responsive Images**
- Generate multiple sizes tự động
- Sử dụng `srcset` và `sizes` attribute
- Browser tự chọn size phù hợp với viewport

```html
<!-- Next.js auto-generate -->
<img 
  srcset="
    /_next/image?url=/photo.jpg&w=640 640w,
    /_next/image?url=/photo.jpg&w=750 750w,
    /_next/image?url=/photo.jpg&w=1080 1080w
  "
  sizes="(max-width: 768px) 100vw, 50vw"
/>
```

#### **3. Lazy Loading**
- Default lazy load cho offscreen images
- Chỉ load khi ảnh gần viewport (Intersection Observer)
- `priority` prop để disable lazy loading cho critical images

#### **4. Image Optimization On-Demand**
- Optimize khi request lần đầu
- Cache optimized images
- Không tốn thời gian build

#### **5. Blur Placeholder**
```jsx
// Automatic blur placeholder với static imports
import profilePic from '../public/profile.jpg';

<Image
  src={profilePic}
  alt="Profile"
  placeholder="blur" // Tự động generate blur
/>

// Manual blur placeholder
<Image
  src="/remote-image.jpg"
  alt="Remote"
  width={500}
  height={500}
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..."
/>
```

### 🔧 Configuration trong next.config.js:

```javascript
// next.config.js
module.exports = {
  images: {
    // Cho phép remote images
    domains: ['cdn.example.com', 'images.unsplash.com'],
    
    // Device sizes cho responsive images
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    
    // Image sizes
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    
    // Formats hỗ trợ
    formats: ['image/avif', 'image/webp'],
    
    // Quality mặc định
    quality: 75,
    
    // Cache optimization
    minimumCacheTTL: 60,
  },
};
```

### 📊 So sánh performance:

| Method | Size | Format | Lazy Load | Responsive |
|--------|------|--------|-----------|------------|
| `<img>` tag | 500KB | JPG | Manual | Manual |
| `next/image` | 150KB | WebP | ✅ Auto | ✅ Auto |
| Savings | **70%** | Modern | Built-in | Built-in |

---

## 3. TỐI ƯU 2000 ẢNH HIỂN THỊ Ở CLIENT

### 🎯 Chiến lược tối ưu đa tầng:

#### **3.1 Pagination (Phân trang)**

```jsx
// pages/gallery.js
export default function Gallery({ images, currentPage, totalPages }) {
  return (
    <div>
      <div className="grid">
        {images.map(image => (
          <Image
            key={image.id}
            src={image.url}
            alt={image.alt}
            width={300}
            height={300}
          />
        ))}
      </div>
      
      <Pagination 
        currentPage={currentPage}
        totalPages={totalPages}
        pageSize={50} // 50 ảnh/page = 40 pages
      />
    </div>
  );
}

export async function getServerSideProps({ query }) {
  const page = parseInt(query.page) || 1;
  const pageSize = 50;
  
  const images = await fetchImages({
    skip: (page - 1) * pageSize,
    take: pageSize
  });
  
  return {
    props: {
      images,
      currentPage: page,
      totalPages: Math.ceil(2000 / pageSize)
    }
  };
}
```

#### **3.2 Infinite Scroll với Intersection Observer**

```jsx
import { useState, useEffect, useRef, useCallback } from 'react';
import Image from 'next/image';

function InfiniteGallery() {
  const [images, setImages] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  
  const observer = useRef();
  
  // Ref callback cho last element
  const lastImageRef = useCallback(node => {
    if (loading) return;
    if (observer.current) observer.current.disconnect();
    
    observer.current = new IntersectionObserver(entries => {
      if (entries[0].isIntersecting && hasMore) {
        setPage(prevPage => prevPage + 1);
      }
    });
    
    if (node) observer.current.observe(node);
  }, [loading, hasMore]);
  
  // Load more images
  useEffect(() => {
    const loadImages = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/images?page=${page}&limit=50`);
        const data = await response.json();
        
        setImages(prev => [...prev, ...data.images]);
        setHasMore(data.hasMore);
      } catch (error) {
        console.error('Error loading images:', error);
      } finally {
        setLoading(false);
      }
    };
    
    loadImages();
  }, [page]);
  
  return (
    <div className="grid grid-cols-4 gap-4">
      {images.map((image, index) => (
        <div
          key={image.id}
          ref={index === images.length - 1 ? lastImageRef : null}
        >
          <Image
            src={image.url}
            alt={image.alt}
            width={300}
            height={300}
            loading="lazy"
          />
        </div>
      ))}
      
      {loading && (
        <div className="col-span-4 text-center">
          Loading more images...
        </div>
      )}
      
      {!hasMore && (
        <div className="col-span-4 text-center">
          No more images
        </div>
      )}
    </div>
  );
}
```

#### **3.3 Virtual Scrolling (React Window)**

**Chỉ render ảnh trong viewport - tối ưu nhất cho list dài**

```bash
npm install react-window react-window-infinite-loader
```

```jsx
import { FixedSizeGrid } from 'react-window';
import InfiniteLoader from 'react-window-infinite-loader';
import Image from 'next/image';

function VirtualGallery({ totalImages = 2000 }) {
  const [images, setImages] = useState({});
  
  // Check nếu item đã load
  const isItemLoaded = index => !!images[index];
  
  // Load more items
  const loadMoreItems = async (startIndex, stopIndex) => {
    const response = await fetch(
      `/api/images?start=${startIndex}&end=${stopIndex}`
    );
    const data = await response.json();
    
    setImages(prev => ({
      ...prev,
      ...data.reduce((acc, img, i) => {
        acc[startIndex + i] = img;
        return acc;
      }, {})
    }));
  };
  
  // Cell renderer
  const Cell = ({ columnIndex, rowIndex, style }) => {
    const index = rowIndex * 4 + columnIndex; // 4 columns
    const image = images[index];
    
    if (!image) {
      return <div style={style}>Loading...</div>;
    }
    
    return (
      <div style={style}>
        <Image
          src={image.url}
          alt={image.alt}
          width={250}
          height={250}
          loading="lazy"
        />
      </div>
    );
  };
  
  return (
    <InfiniteLoader
      isItemLoaded={isItemLoaded}
      itemCount={totalImages}
      loadMoreItems={loadMoreItems}
    >
      {({ onItemsRendered, ref }) => (
        <FixedSizeGrid
          ref={ref}
          columnCount={4}
          columnWidth={250}
          height={800}
          rowCount={Math.ceil(totalImages / 4)}
          rowHeight={250}
          width={1000}
          onItemsRendered={gridProps => {
            onItemsRendered({
              overscanStartIndex: gridProps.overscanRowStartIndex * 4,
              overscanStopIndex: gridProps.overscanRowStopIndex * 4,
              visibleStartIndex: gridProps.visibleRowStartIndex * 4,
              visibleStopIndex: gridProps.visibleRowStopIndex * 4,
            });
          }}
        >
          {Cell}
        </FixedSizeGrid>
      )}
    </InfiniteLoader>
  );
}
```

**Hiệu quả:**
- Chỉ render ~20-30 ảnh thay vì 2000
- Smooth scroll với 60fps
- Memory usage thấp

#### **3.4 CDN Caching Strategy**

```javascript
// next.config.js
module.exports = {
  images: {
    domains: ['cdn.example.com'],
    
    // Cache images on CDN
    loader: 'cloudinary', // hoặc 'imgix', 'akamai'
    path: 'https://res.cloudinary.com/demo/image/upload/',
  },
};

// Hoặc custom loader
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './my-loader.js',
  },
};
```

```javascript
// my-loader.js
export default function cloudinaryLoader({ src, width, quality }) {
  const params = ['f_auto', 'c_limit', `w_${width}`, `q_${quality || 'auto'}`];
  return `https://res.cloudinary.com/demo/image/upload/${params.join(',')}${src}`;
}
```

#### **3.5 Thumbnail + Progressive Loading**

```jsx
function ProgressiveImage({ thumbnail, fullSize, alt }) {
  const [imgSrc, setImgSrc] = useState(thumbnail);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const img = new window.Image();
    img.src = fullSize;
    img.onload = () => {
      setImgSrc(fullSize);
      setLoading(false);
    };
  }, [fullSize]);
  
  return (
    <div className="relative">
      <Image
        src={imgSrc}
        alt={alt}
        width={500}
        height={500}
        className={`transition-all duration-300 ${
          loading ? 'blur-sm' : 'blur-0'
        }`}
      />
      {loading && <Spinner />}
    </div>
  );
}

// API route để generate thumbnails
// pages/api/thumbnail.js
import sharp from 'sharp';

export default async function handler(req, res) {
  const { url } = req.query;
  
  const response = await fetch(url);
  const buffer = await response.arrayBuffer();
  
  const thumbnail = await sharp(Buffer.from(buffer))
    .resize(50, 50)
    .webp({ quality: 20 })
    .toBuffer();
    
  res.setHeader('Content-Type', 'image/webp');
  res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
  res.send(thumbnail);
}
```

#### **3.6 Preload Critical Images**

```jsx
import Image from 'next/image';
import Head from 'next/head';

function Gallery({ heroImages, regularImages }) {
  return (
    <>
      <Head>
        {/* Preload hero images */}
        {heroImages.slice(0, 3).map(img => (
          <link
            key={img.id}
            rel="preload"
            as="image"
            href={img.url}
            imageSrcSet={`
              ${img.url}?w=640 640w,
              ${img.url}?w=1080 1080w,
              ${img.url}?w=1920 1920w
            `}
          />
        ))}
      </Head>
      
      <div className="hero">
        {heroImages.map(img => (
          <Image
            key={img.id}
            src={img.url}
            alt={img.alt}
            width={1920}
            height={1080}
            priority // Disable lazy loading
          />
        ))}
      </div>
      
      <div className="gallery">
        {regularImages.map(img => (
          <Image
            key={img.id}
            src={img.url}
            alt={img.alt}
            width={300}
            height={300}
            loading="lazy" // Lazy load for below-the-fold
          />
        ))}
      </div>
    </>
  );
}
```

### 📊 Tổng kết chiến lược:

| Kỹ thuật | Use Case | Performance Gain |
|----------|----------|------------------|
| **Pagination** | Simple galleries | Good (50 images/load) |
| **Infinite Scroll** | Social feeds | Better (progressive loading) |
| **Virtual Scrolling** | Massive lists | Best (20-30 images rendered) |
| **CDN Caching** | Global users | Excellent (edge caching) |
| **Thumbnail First** | Slow networks | Great (perceived performance) |
| **Preload Critical** | Above-the-fold | Instant (no delay) |

### 🎯 Recommended Stack cho 2000 ảnh:

```jsx
// Kết hợp các kỹ thuật
function OptimizedGallery() {
  return (
    <div>
      {/* Hero images - preload, priority */}
      <HeroSection images={heroImages} />
      
      {/* Virtual scrolling cho bulk images */}
      <VirtualGallery 
        images={allImages}
        itemsPerPage={50}
        cdn="cloudinary"
      />
      
      {/* Infinite scroll fallback cho mobile */}
      <InfiniteScrollGallery 
        images={allImages}
        threshold={0.8}
      />
    </div>
  );
}
```

---

## 4. TREE SHAKING

### 🌳 Định nghĩa:

**Tree Shaking** là kỹ thuật loại bỏ **dead code** (code không được sử dụng) khỏi JavaScript bundle trong quá trình build.

### 🔍 Cách hoạt động:

1. **Static Analysis**: Phân tích imports/exports trong build time
2. **Mark unused code**: Đánh dấu functions/variables không được reference
3. **Remove dead code**: Loại bỏ khỏi final bundle
4. **Minify**: Minification process hoàn tất việc cleanup

### ⚙️ Yêu cầu:

✅ **ES6 Modules** (import/export)
```javascript
// ✅ Tree-shakable
import { add } from './math';

// ❌ Không tree-shakable
const math = require('./math');
```

✅ **Production build**
```bash
# Next.js tự động enable tree shaking trong production
npm run build
```

✅ **Side-effect free code**
```javascript
// package.json
{
  "sideEffects": false, // Báo cho bundler biết code không có side effects
  // Hoặc specify files có side effects
  "sideEffects": ["*.css", "*.scss"]
}
```

### 📝 Ví dụ thực tế:

#### **Before Tree Shaking:**

```javascript
// utils/math.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export function multiply(a, b) {
  return a * b;
}

export function divide(a, b) {
  return a / b;
}

// app.js
import { add } from './utils/math';

console.log(add(1, 2));

// ❌ Bundle size: Tất cả 4 functions (~500 bytes)
```

#### **After Tree Shaking:**

```javascript
// app.js - same code
import { add } from './utils/math';

console.log(add(1, 2));

// ✅ Bundle size: Chỉ function add (~100 bytes)
// subtract, multiply, divide bị loại bỏ
```

### 🎯 Best Practices:

#### **1. Named Imports thay vì Default/Namespace**

```javascript
// ❌ Không tree-shakable - import cả library
import _ from 'lodash';
_.map([1, 2, 3], n => n * 2);

// ❌ Namespace import
import * as utils from './utils';
utils.add(1, 2);

// ✅ Tree-shakable - chỉ import cần thiết
import { map } from 'lodash-es';
map([1, 2, 3], n => n * 2);

// ✅ Named imports
import { add, subtract } from './utils';
add(1, 2);
```

#### **2. Lodash-es thay vì Lodash**

```bash
# ❌ CommonJS - không tree-shakable
npm install lodash

# ✅ ES Modules - tree-shakable
npm install lodash-es
```

```javascript
// ❌ Import toàn bộ lodash (~70KB)
import _ from 'lodash';
_.debounce(func, 300);

// ✅ Import individual function (~2KB)
import debounce from 'lodash-es/debounce';
debounce(func, 300);
```

#### **3. Kiểm tra Side Effects**

```javascript
// utils.js - ❌ Có side effect
console.log('Utils loaded!'); // Side effect khi import

export function add(a, b) {
  return a + b;
}

// utils.js - ✅ Không side effect
export function add(a, b) {
  return a + b;
}
// Pure function, no side effects
```

#### **4. Analyze Bundle Size**

```bash
# Install bundle analyzer
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // your config
});

# Run analysis
ANALYZE=true npm run build
```

### 📊 So sánh thực tế:

```javascript
// Component.jsx
import { Button, Modal, Dropdown } from 'antd'; // Chỉ dùng Button

// ❌ Không optimize
// Bundle: antd entire library (500KB)

// ✅ Với Tree Shaking
// Bundle: Chỉ Button component (50KB)
// Tiết kiệm: 450KB (90%)
```

### 🔧 Configuration trong Next.js:

```javascript
// next.config.js
module.exports = {
  // Tree shaking tự động enable trong production
  
  // Optimize imports
  experimental: {
    optimizePackageImports: ['lodash-es', 'date-fns', 'recharts'],
  },
  
  // Webpack config
  webpack: (config, { isServer }) => {
    if (!isServer) {
      // Optimize tree shaking
      config.optimization.usedExports = true;
      config.optimization.sideEffects = true;
    }
    return config;
  },
};
```

### ✅ Libraries Tree-shakable tốt:

- ✅ **lodash-es** (not lodash)
- ✅ **date-fns** (not moment)
- ✅ **rxjs** 
- ✅ **ramda**
- ✅ **antd** (Ant Design)
- ✅ **material-ui** (MUI)

### ❌ Libraries không Tree-shakable:

- ❌ **lodash** (CommonJS)
- ❌ **moment** (monolithic)
- ❌ **jquery**

### 🎯 Kiểm tra Tree Shaking:

```bash
# Build production
npm run build

# Check bundle size
du -h .next/static/chunks/*.js

# Hoặc dùng bundle analyzer
ANALYZE=true npm run build
```

**Kết quả mong đợi:**
```
Before tree shaking: 500KB bundle
After tree shaking:  150KB bundle
Reduction: 70%
```

---

## 5. CORE WEB VITALS

### 🎯 Định nghĩa:

**Core Web Vitals** là tập hợp **3 metrics quan trọng nhất** mà Google sử dụng để đánh giá **User Experience (UX)** của website. Các metrics này ảnh hưởng trực tiếp đến **SEO ranking**.

---

### 📊 Ba Metrics Chính:

#### **5.1 LCP - Largest Contentful Paint**

**Định nghĩa:** Thời gian để phần tử nội dung lớn nhất trên viewport được render.

**Tiêu chuẩn:**
- ✅ **Good**: < 2.5 giây
- ⚠️ **Needs Improvement**: 2.5 - 4.0 giây
- ❌ **Poor**: > 4.0 giây

**Phần tử thường được tính:**
- `<img>` tags
- `<video>` thumbnail
- Background images (CSS)
- Block-level text elements

**Cách optimize:**

```jsx
// ❌ Bad - không optimize
<img src="/hero-image.jpg" alt="Hero" />

// ✅ Good - Next.js Image với priority
import Image from 'next/image';

<Image
  src="/hero-image.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority // Preload ngay lập tức
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>
```

**Additional optimizations:**
```javascript
// Preload critical resources
<Head>
  <link
    rel="preload"
    as="image"
    href="/hero.jpg"
    imageSrcSet="/hero-640.jpg 640w, /hero-1280.jpg 1280w"
  />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
</Head>

// Server-side rendering cho above-the-fold content
export async function getServerSideProps() {
  const criticalData = await fetchCriticalData();
  return { props: { criticalData } };
}

// Optimize fonts
import { Inter } from 'next/font/google';

const inter = Inter({ 
  subsets: ['latin'],
  display: 'swap', // Prevent FOIT (Flash of Invisible Text)
});
```

---

#### **5.2 FID - First Input Delay** *(Thay bằng INP từ 2024)*

**Định nghĩa:** Thời gian từ khi user tương tác đầu tiên (click, tap) đến khi browser có thể phản hồi.

**Tiêu chuẩn:**
- ✅ **Good**: < 100ms
- ⚠️ **Needs Improvement**: 100 - 300ms
- ❌ **Poor**: > 300ms

**Nguyên nhân chậm:**
- JavaScript execution blocks main thread
- Heavy tasks khi page load
- Third-party scripts

**Cách optimize:**

```jsx
// ❌ Bad - blocking script
import HeavyLibrary from 'heavy-library';

function MyComponent() {
  const result = HeavyLibrary.calculate(); // Blocks render
  return <div>{result}</div>;
}

// ✅ Good - code splitting
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>Loading...</p>,
  ssr: false // Chỉ load ở client khi cần
});

function MyComponent() {
  return <HeavyComponent />;
}
```

**Web Workers cho heavy computation:**
```javascript
// worker.js
self.addEventListener('message', (e) => {
  const result = heavyCalculation(e.data);
  self.postMessage(result);
});

// Component.jsx
import { useEffect, useState } from 'react';

function Component() {
  const [result, setResult] = useState(null);
  
  useEffect(() => {
    const worker = new Worker('/worker.js');
    
    worker.postMessage(data);
    worker.onmessage = (e) => {
      setResult(e.data);
    };
    
    return () => worker.terminate();
  }, []);
  
  return <div>{result}</div>;
}
```

**Debounce user interactions:**
```javascript
import { useCallback } from 'react';
import debounce from 'lodash-es/debounce';

function SearchComponent() {
  const handleSearch = useCallback(
    debounce((query) => {
      // Heavy search operation
      performSearch(query);
    }, 300), // Wait 300ms after user stops typing
    []
  );
  
  return <input onChange={(e) => handleSearch(e.target.value)} />;
}
```

---

#### **5.3 CLS - Cumulative Layout Shift**

**Định nghĩa:** Tổng số layout shifts không mong muốn trong suốt page lifetime.

**Tiêu chuẩn:**
- ✅ **Good**: < 0.1
- ⚠️ **Needs Improvement**: 0.1 - 0.25
- ❌ **Poor**: > 0.25

**Nguyên nhân:**
- Images không có dimensions
- Ads/embeds inject động
- Fonts loading gây re-layout (FOUT - Flash of Unstyled Text)
- Animations trigger layout

**Cách optimize:**

```jsx
// ❌ Bad - không specify dimensions
<img src="/photo.jpg" alt="Photo" />

// ✅ Good - specify dimensions
<Image
  src="/photo.jpg"
  alt="Photo"
  width={500}
  height={500}
/>

// ❌ Bad - dynamic content without space
<div>
  <h1>Title</h1>
  {ads && <AdBanner />} {/* Đẩy content xuống */}
</div>

// ✅ Good - reserve space
<div>
  <h1>Title</h1>
  <div className="ad-container" style={{ minHeight: '250px' }}>
    {ads && <AdBanner />}
  </div>
</div>
```

**Font loading strategy:**
```javascript
// next.config.js
const { Inter } = require('next/font/google');

const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // Prevent layout shift
  preload: true,
});

// _app.js
<main className={inter.className}>
  {children}
</main>
```

**CSS containment:**
```css
/* Prevent layout shifts */
.card {
  contain: layout; /* Isolate layout calculations */
}

.image-container {
  aspect-ratio: 16 / 9; /* Reserve space */
}
```

---

### 📈 Measurement & Monitoring trong Next.js:

#### **5.4.1 Built-in Web Vitals Reporting**

```javascript
// pages/_app.js
export function reportWebVitals(metric) {
  // Metric object structure:
  // {
  //   id: 'v3-1234567890',
  //   name: 'FCP' | 'LCP' | 'CLS' | 'FID' | 'TTFB',
  //   value: 123.45,
  //   label: 'web-vital' | 'custom',
  //   startTime: 0,
  // }
  
  if (metric.label === 'web-vital') {
    console.log(metric.name, metric.value);
    
    // Send to analytics
    switch (metric.name) {
      case 'LCP':
        console.log(`LCP: ${metric.value}ms`);
        break;
      case 'FID':
        console.log(`FID: ${metric.value}ms`);
        break;
      case 'CLS':
        console.log(`CLS: ${metric.value}`);
        break;
      case 'FCP':
        console.log(`FCP: ${metric.value}ms`);
        break;
      case 'TTFB':
        console.log(`TTFB: ${metric.value}ms`);
        break;
    }
    
    // Send to Google Analytics
    if (window.gtag) {
      window.gtag('event', metric.name, {
        value: Math.round(metric.value),
        event_label: metric.id,
        non_interaction: true,
      });
    }
    
    // Send to custom analytics endpoint
    fetch('/api/analytics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(metric),
    });
  }
}

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}

export default MyApp;
```

#### **5.4.2 Real User Monitoring (RUM)**

```javascript
// lib/analytics.js
export function sendToAnalytics(metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    rating: metric.rating,
    delta: metric.delta,
    id: metric.id,
    navigationType: metric.navigationType,
  });
  
  // Use sendBeacon if available (không block page unload)
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/analytics', body);
  } else {
    fetch('/api/analytics', {
      method: 'POST',
      body,
      keepalive: true,
    });
  }
}

// pages/_app.js
import { sendToAnalytics } from '../lib/analytics';

export function reportWebVitals(metric) {
  if (metric.label === 'web-vital') {
    sendToAnalytics(metric);
  }
}
```

#### **5.4.3 Integration với Vercel Analytics**

```bash
npm install @vercel/analytics
```

```javascript
// pages/_app.js
import { Analytics } from '@vercel/analytics/react';

function MyApp({ Component, pageProps }) {
  return (
    <>
      <Component {...pageProps} />
      <Analytics />
    </>
  );
}

export default MyApp;
```

#### **5.4.4 Chrome User Experience Report (CrUX)**

```javascript
// pages/api/crux-report.js
export default async function handler(req, res) {
  const { url } = req.query;
  
  const response = await fetch(
    `https://chromeuxreport.googleapis.com/v1/records:queryRecord?key=${process.env.CRUX_API_KEY}`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        url: url,
        formFactor: 'PHONE',
        metrics: ['largest_contentful_paint', 'first_input_delay', 'cumulative_layout_shift'],
      }),
    }
  );
  
  const data = await response.json();
  res.json(data);
}
```

---

### 🛠️ Tools để đo Core Web Vitals:

1. **Lighthouse (Lab data)**
   ```bash
   # Chrome DevTools > Lighthouse tab
   # Hoặc CLI
   npm install -g lighthouse
   lighthouse https://example.com --view
   ```

2. **PageSpeed Insights (Field + Lab data)**
   - https://pagespeed.web.dev/
   - Real user data từ CrUX report

3. **Chrome DevTools Performance**
   - Record performance profile
   - Analyze rendering timeline

4. **Web Vitals Extension**
   - Chrome extension: "Web Vitals"
   - Real-time metrics overlay

5. **Vercel Analytics**
   - Automatic tracking cho Next.js apps deployed on Vercel

---

### 📋 Checklist tối ưu Core Web Vitals:

#### **LCP Optimization:**
- [ ] Sử dụng `next/image` với `priority` cho hero images
- [ ] Preload critical resources
- [ ] Optimize server response time (TTFB)
- [ ] Remove render-blocking resources
- [ ] Use CDN cho static assets
- [ ] Implement proper caching headers

#### **FID/INP Optimization:**
- [ ] Code splitting với dynamic imports
- [ ] Defer non-critical JavaScript
- [ ] Use Web Workers cho heavy tasks
- [ ] Minimize third-party scripts
- [ ] Debounce/throttle event handlers
- [ ] Optimize JavaScript execution time

#### **CLS Optimization:**
- [ ] Set dimensions cho images/videos
- [ ] Reserve space cho ads/embeds
- [ ] Use `font-display: swap`
- [ ] Avoid inserting content above existing content
- [ ] Use CSS `aspect-ratio`
- [ ] Use CSS containment

---

### 🎯 Mục tiêu Performance cho Next.js app:

```
Perfect Score:
✅ LCP: < 2.5s
✅ FID: < 100ms  
✅ CLS: < 0.1

Lighthouse Score: 90-100 (Green)
PageSpeed Score: 90-100
```

**Production checklist:**
```bash
# Build production
npm run build

# Analyze bundle
ANALYZE=true npm run build

# Test with Lighthouse
lighthouse https://your-site.com --view

# Monitor with Vercel Analytics
# Deploy lên Vercel và check Analytics dashboard
```

---

## 🎓 TÓM TẮT & TAKEAWAYS

### ✅ 5 Điểm chính cần nhớ:

1. **SSR** = SEO + Fresh Data (user dashboards, personalization)
2. **next/image** = Auto optimization (WebP, responsive, lazy load)
3. **2000 ảnh** = Virtual scrolling + Pagination + CDN
4. **Tree Shaking** = Named imports + ES modules = Smaller bundles
5. **Core Web Vitals** = LCP < 2.5s + FID < 100ms + CLS < 0.1

### 💡 Tips trả lời phỏng vấn:

- ✅ Show code examples (prove you know it)
- ✅ Explain trade-offs (when to use, when not to use)
- ✅ Mention metrics/numbers (70% reduction, < 2.5s)
- ✅ Discuss alternatives (SSR vs SSG vs ISR)
- ✅ Connect to real-world use cases

---

## 6. CÂU HỎI BỔ SUNG - CHUẨN BỊ PHỎNG VẤN

### 📚 NEXTJS ADVANCED QUESTIONS

#### **6.1 Middleware trong Next.js là gì? Khi nào dùng?**

**Định nghĩa:**
Middleware chạy **trước khi request** đến page/API route, trên **Edge Runtime** (nhanh hơn Node.js).

**Use cases:**
- ✅ Authentication/Authorization
- ✅ Redirects và Rewrites
- ✅ A/B Testing
- ✅ Geolocation-based routing
- ✅ Rate limiting
- ✅ Header modification
- ✅ Bot detection

**Ví dụ:**

```javascript
// middleware.js (root level)
import { NextResponse } from 'next/server';
import { verify } from 'jsonwebtoken';

export async function middleware(request) {
  const { pathname } = request.nextUrl;
  
  // 1. Authentication check
  if (pathname.startsWith('/dashboard')) {
    const token = request.cookies.get('auth-token')?.value;
    
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
    
    try {
      await verify(token, process.env.JWT_SECRET);
    } catch {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }
  
  // 2. A/B Testing
  if (pathname === '/') {
    const bucket = request.cookies.get('ab-test')?.value || 
                   Math.random() > 0.5 ? 'a' : 'b';
    
    const response = NextResponse.next();
    response.cookies.set('ab-test', bucket);
    
    // Rewrite to variant
    if (bucket === 'b') {
      return NextResponse.rewrite(new URL('/home-variant-b', request.url));
    }
  }
  
  // 3. Geolocation redirect
  const country = request.geo?.country;
  if (country === 'VN' && !pathname.startsWith('/vi')) {
    return NextResponse.redirect(new URL('/vi' + pathname, request.url));
  }
  
  // 4. Add custom headers
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'my-value');
  response.headers.set('x-request-time', Date.now().toString());
  
  return response;
}

// Specify which routes middleware runs on
export const config = {
  matcher: [
    '/dashboard/:path*',
    '/admin/:path*',
    '/',
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};
```

**Edge Runtime limitations:**
```javascript
// ❌ KHÔNG thể dùng trong middleware
- Node.js APIs (fs, path, crypto.createHash)
- Database connections (Prisma, Mongoose)
- Heavy computations
- npm packages với native dependencies

// ✅ CÓ thể dùng
- fetch API
- Web Crypto API
- JWT verification (jose library)
- Cookies, headers manipulation
- URL redirects/rewrites
```

**Advanced: Rate Limiting với Middleware**

```javascript
// middleware.js
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'), // 10 requests per 10s
});

export async function middleware(request) {
  const ip = request.ip || request.headers.get('x-forwarded-for') || '127.0.0.1';
  
  const { success, limit, reset, remaining } = await ratelimit.limit(ip);
  
  if (!success) {
    return new NextResponse('Rate limit exceeded', {
      status: 429,
      headers: {
        'X-RateLimit-Limit': limit.toString(),
        'X-RateLimit-Remaining': remaining.toString(),
        'X-RateLimit-Reset': reset.toString(),
      },
    });
  }
  
  const response = NextResponse.next();
  response.headers.set('X-RateLimit-Remaining', remaining.toString());
  
  return response;
}
```

---

#### **6.2 ISR (Incremental Static Regeneration) vs SSR - Khi nào dùng?**

**ISR - Best of Both Worlds:**
- Static generation + Auto-update content
- Build 1 lần, regenerate theo schedule
- Scale tốt như SSG, fresh như SSR

**So sánh:**

| Feature | SSG | ISR | SSR |
|---------|-----|-----|-----|
| Build time | Slow (all pages) | Fast (on-demand) | N/A |
| Performance | ⚡ Fastest | ⚡⚡ Very Fast | 🐢 Slower |
| Data freshness | ❌ Stale until rebuild | ✅ Auto-update | ✅ Always fresh |
| Server load | ✅ Minimal | ✅ Minimal | ❌ High |
| Scalability | ✅ Excellent | ✅ Excellent | ⚠️ Limited |
| Use case | Static content | Semi-dynamic | User-specific |

**Khi nào dùng ISR:**

```javascript
// pages/blog/[slug].js - Perfect for ISR
export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  
  return {
    props: { post },
    revalidate: 60, // Regenerate every 60 seconds
  };
}

export async function getStaticPaths() {
  const posts = await fetchPopularPosts(10); // Pre-build 10 popular posts
  
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: 'blocking', // Generate other posts on-demand
  };
}
```

**Fallback options:**

```javascript
// fallback: false
// - Chỉ pre-render paths từ getStaticPaths
// - 404 cho other paths
return { paths, fallback: false };

// fallback: true
// - Show loading state
// - Generate page in background
// - Cache for next request
return { paths, fallback: true };

function BlogPost({ post }) {
  const router = useRouter();
  
  if (router.isFallback) {
    return <div>Loading...</div>;
  }
  
  return <div>{post.title}</div>;
}

// fallback: 'blocking'
// - Wait for page generation
// - No loading state needed
// - Better for SEO
return { paths, fallback: 'blocking' };
```

**On-Demand Revalidation:**

```javascript
// pages/api/revalidate.js
export default async function handler(req, res) {
  // Check secret to confirm this is a valid request
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  try {
    // Revalidate specific path
    await res.revalidate('/blog/post-1');
    await res.revalidate('/blog/post-2');
    
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// Webhook từ CMS khi content update
// POST https://your-site.com/api/revalidate?secret=YOUR_SECRET&path=/blog/new-post
```

**Tag-based Revalidation (Next.js 13 App Router):**

```javascript
// app/blog/[slug]/page.tsx
export default async function BlogPost({ params }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`, {
    next: { 
      tags: ['posts', `post-${params.slug}`],
      revalidate: 3600 // 1 hour
    }
  });
  
  return <div>{post.title}</div>;
}

// app/api/revalidate/route.ts
import { revalidateTag } from 'next/cache';

export async function POST(request) {
  const { tag } = await request.json();
  
  revalidateTag(tag); // Invalidate specific tag
  
  return Response.json({ revalidated: true, now: Date.now() });
}
```

---

#### **6.3 App Router vs Pages Router - Khác biệt và Migration?**

**Architecture comparison:**

```
Pages Router (Legacy)          App Router (Next.js 13+)
pages/                         app/
├── _app.js                   ├── layout.tsx (root)
├── _document.js              ├── page.tsx
├── index.js                  ├── loading.tsx
├── about.js                  ├── error.tsx
├── blog/                     ├── about/
│   ├── [slug].js            │   └── page.tsx
│   └── index.js             └── blog/
└── api/                          ├── layout.tsx
    └── users.js                  ├── [slug]/
                                  │   └── page.tsx
                                  └── page.tsx
```

**Key differences:**

| Feature | Pages Router | App Router |
|---------|-------------|------------|
| **Components** | Client by default | Server by default |
| **Data Fetching** | getServerSideProps | async components |
| **Layouts** | Manual with _app.js | Nested layouts |
| **Loading States** | Manual | loading.tsx |
| **Error Handling** | _error.js | error.tsx |
| **Streaming** | No | Yes (Suspense) |
| **Metadata** | Head component | Metadata API |

**Server vs Client Components:**

```tsx
// app/blog/page.tsx - Server Component (default)
async function BlogPage() {
  // Can access database directly, no API route needed
  const posts = await db.post.findMany();
  
  return (
    <div>
      {posts.map(post => (
        <PostCard key={post.id} post={post} />
      ))}
    </div>
  );
}

// app/components/Counter.tsx - Client Component
'use client'; // Must declare

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

**Nested Layouts:**

```tsx
// app/layout.tsx - Root layout
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  );
}

// app/blog/layout.tsx - Blog layout
export default function BlogLayout({ children }) {
  return (
    <div className="blog-container">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}

// app/blog/[slug]/page.tsx - Blog post page
// Inherits: RootLayout + BlogLayout
```

**Streaming with Suspense:**

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Fast component renders immediately */}
      <UserInfo />
      
      {/* Slow components stream in */}
      <Suspense fallback={<AnalyticsSkeleton />}>
        <Analytics />
      </Suspense>
      
      <Suspense fallback={<ChartsSkeleton />}>
        <Charts />
      </Suspense>
    </div>
  );
}

// Server Component with slow data fetch
async function Analytics() {
  const data = await fetchAnalytics(); // Slow query
  return <div>{data}</div>;
}
```

**Metadata API:**

```tsx
// Pages Router
import Head from 'next/head';

function Page() {
  return (
    <>
      <Head>
        <title>My Page</title>
        <meta name="description" content="Description" />
      </Head>
      <div>Content</div>
    </>
  );
}

// App Router - Static metadata
export const metadata = {
  title: 'My Page',
  description: 'Description',
  openGraph: {
    title: 'My Page',
    description: 'Description',
    images: ['/og-image.jpg'],
  },
};

export default function Page() {
  return <div>Content</div>;
}

// App Router - Dynamic metadata
export async function generateMetadata({ params }) {
  const post = await fetchPost(params.slug);
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.coverImage],
    },
  };
}
```

**Migration strategy:**

```javascript
// Can use both routers simultaneously
my-app/
├── app/           // New App Router routes
│   └── new-page/
│       └── page.tsx
├── pages/         // Existing Pages Router routes
│   └── old-page.js
│   
// Priority: app/ takes precedence over pages/
// Migrate gradually, route by route
```

---

#### **6.4 getStaticProps vs getServerSideProps vs Client-side fetching?**

**Decision tree:**

```
Content thay đổi như thế nào?
│
├── Không thay đổi / Ít thay đổi
│   └── SEO quan trọng?
│       ├── Yes → getStaticProps (SSG)
│       └── No  → Client-side
│
├── Thay đổi định kỳ (hourly/daily)
│   └── → getStaticProps + revalidate (ISR)
│
└── Thay đổi mỗi request
    └── SEO quan trọng?
        ├── Yes → getServerSideProps (SSR)
        └── No  → Client-side
```

**Comparison table:**

| Criteria | getStaticProps | getServerSideProps | Client-side |
|----------|----------------|-------------------|-------------|
| **Performance** | ⚡⚡⚡ | ⚡ | ⚡⚡ |
| **SEO** | ✅ Perfect | ✅ Good | ❌ Poor |
| **Data freshness** | ❌ Stale | ✅ Fresh | ✅ Fresh |
| **Server load** | ✅ None | ❌ High | ✅ None |
| **Build time** | ⚠️ Slow if many pages | ✅ Fast | ✅ Fast |
| **Private data** | ❌ No | ✅ Yes | ✅ Yes |
| **When runs** | Build time | Request time | Browser |

**Examples:**

```javascript
// 1. getStaticProps - Blog, Marketing, Documentation
export async function getStaticProps() {
  const posts = await fetchPosts();
  
  return {
    props: { posts },
    revalidate: 3600, // ISR: Update every hour
  };
}

// 2. getServerSideProps - Dashboard, User Profile
export async function getServerSideProps({ req, res }) {
  const session = await getSession({ req });
  
  if (!session) {
    return {
      redirect: { destination: '/login', permanent: false }
    };
  }
  
  const userData = await fetchUserData(session.user.id);
  
  // Set cache headers
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=10, stale-while-revalidate=59'
  );
  
  return {
    props: { userData }
  };
}

// 3. Client-side - Real-time, Interactive
function Dashboard() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error</div>;
  
  return <div>{data.name}</div>;
}
```

**Hybrid approach (Best practice):**

```jsx
// Combine multiple strategies
function ProductPage({ product }) {
  // Static data from getStaticProps
  const { title, description } = product;
  
  // Real-time data from client
  const { data: inventory } = useSWR(`/api/inventory/${product.id}`);
  const { data: reviews } = useSWR(`/api/reviews/${product.id}`);
  
  return (
    <div>
      <h1>{title}</h1>
      <p>{description}</p>
      
      {/* Static content */}
      <ProductDetails product={product} />
      
      {/* Dynamic content */}
      <Inventory count={inventory?.count} />
      <Reviews reviews={reviews} />
    </div>
  );
}

export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  
  return {
    props: { product },
    revalidate: 300, // Update every 5 minutes
  };
}
```

---

#### **6.5 Dynamic Routes - Catch-all và Optional Catch-all?**

**File naming patterns:**

```
pages/
├── [id].js              // /posts/1, /posts/abc
├── [...slug].js         // /blog/a, /blog/a/b, /blog/a/b/c
└── [[...slug]].js       // /shop, /shop/a, /shop/a/b
```

**Examples:**

```javascript
// pages/blog/[slug].js - Single dynamic param
export async function getStaticPaths() {
  return {
    paths: [
      { params: { slug: 'hello-world' } },
      { params: { slug: 'nextjs-tutorial' } }
    ],
    fallback: 'blocking'
  };
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  return { props: { post } };
}

// pages/blog/[...slug].js - Catch-all routes
// Matches: /blog/2023/01/hello, /blog/category/tech/post-1
export async function getStaticProps({ params }) {
  const { slug } = params; // slug is array: ['2023', '01', 'hello']
  
  const post = await fetchPostByPath(slug.join('/'));
  return { props: { post } };
}

// pages/shop/[[...slug]].js - Optional catch-all
// Matches: /shop, /shop/electronics, /shop/electronics/phones
export async function getStaticProps({ params }) {
  const slug = params?.slug || []; // Empty array if /shop
  
  if (slug.length === 0) {
    // Homepage
    const categories = await fetchCategories();
    return { props: { categories } };
  } else if (slug.length === 1) {
    // Category page
    const products = await fetchProductsByCategory(slug[0]);
    return { props: { products } };
  } else {
    // Subcategory page
    const products = await fetchProductsBySubcategory(slug.join('/'));
    return { props: { products } };
  }
}
```

**Priority order:**

```
pages/
├── blog.js                    // 1st priority: /blog
├── blog/first-post.js         // 2nd priority: /blog/first-post
├── blog/[slug].js            // 3rd priority: /blog/any-post
└── blog/[...slug].js         // 4th priority: /blog/a/b/c
```

---

### 🎨 REACT PATTERNS & OPTIMIZATION

#### **6.6 useMemo vs useCallback - Khi nào dùng?**

**Core difference:**

```javascript
// useMemo - Memoize VALUES
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b); // Return value
}, [a, b]);

// useCallback - Memoize FUNCTIONS
const expensiveFunction = useCallback(() => {
  doSomething(a, b); // Return function
}, [a, b]);

// useCallback is shorthand for:
const expensiveFunction = useMemo(() => {
  return () => doSomething(a, b);
}, [a, b]);
```

**When to use useMemo:**

```jsx
import { useMemo } from 'react';

function ProductList({ products, filter }) {
  // ❌ Without useMemo - recalculate every render
  const filteredProducts = products.filter(p => p.category === filter);
  
  // ✅ With useMemo - only recalculate when dependencies change
  const filteredProducts = useMemo(() => {
    console.log('Filtering products...');
    return products.filter(p => p.category === filter);
  }, [products, filter]);
  
  return (
    <div>
      {filteredProducts.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}
```

**When to use useCallback:**

```jsx
import { useCallback, memo } from 'react';

// Child component memoized
const Button = memo(({ onClick, label }) => {
  console.log('Button rendered:', label);
  return <button onClick={onClick}>{label}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  // ❌ Without useCallback - new function every render
  const handleClick = () => {
    console.log('Clicked');
  };
  
  // ✅ With useCallback - same function reference
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []); // No dependencies = never changes
  
  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      {/* Button won't re-render when text changes */}
      <Button onClick={handleClick} label="Click me" />
      <p>Count: {count}</p>
    </div>
  );
}
```

**Common pitfalls:**

```jsx
// ❌ WRONG - Missing dependencies
const value = useMemo(() => {
  return a + b; // Uses a, b but not in deps
}, []); // Should be [a, b]

// ❌ WRONG - Premature optimization
const sum = useMemo(() => a + b, [a, b]); // Overkill for simple calc

// ✅ RIGHT - Expensive calculation
const sortedList = useMemo(() => {
  return hugeArray.sort((a, b) => complexCompare(a, b));
}, [hugeArray]);

// ❌ WRONG - Object dependency changes every render
const config = { theme: 'dark' };
const value = useMemo(() => compute(config), [config]); // config is new object

// ✅ RIGHT - Stable dependency
const theme = 'dark';
const value = useMemo(() => compute(theme), [theme]);
```

**Performance testing:**

```jsx
function App() {
  const [count, setCount] = useState(0);
  
  // Measure render time
  console.time('render');
  
  const expensiveValue = useMemo(() => {
    let sum = 0;
    for (let i = 0; i < 1000000; i++) {
      sum += i;
    }
    return sum;
  }, []);
  
  console.timeEnd('render');
  
  return <div>{expensiveValue}</div>;
}
```

---

#### **6.7 Custom Hooks - Best Practices?**

**Basic structure:**

```javascript
// hooks/useLocalStorage.js
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  // Initialize state from localStorage
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Update localStorage when state changes
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setStoredValue];
}

// Usage
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current theme: {theme}
    </button>
  );
}
```

**Advanced: useDebounce**

```javascript
// hooks/useDebounce.js
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage - Search component
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // API call only after user stops typing for 500ms
      fetchResults(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={e => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

**useAsync - Data fetching hook**

```javascript
// hooks/useAsync.js
import { useState, useEffect, useCallback } from 'react';

function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState('idle');
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);
  
  const execute = useCallback(async (...params) => {
    setStatus('pending');
    setValue(null);
    setError(null);
    
    try {
      const response = await asyncFunction(...params);
      setValue(response);
      setStatus('success');
      return response;
    } catch (error) {
      setError(error);
      setStatus('error');
      throw error;
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);
  
  return { execute, status, value, error, isLoading: status === 'pending' };
}

// Usage
function UserProfile({ userId }) {
  const getUser = useCallback(() => {
    return fetch(`/api/users/${userId}`).then(r => r.json());
  }, [userId]);
  
  const { value: user, isLoading, error, execute } = useAsync(getUser);
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      <h1>{user?.name}</h1>
      <button onClick={execute}>Refresh</button>
    </div>
  );
}
```

**useWindowSize - Responsive hook**

```javascript
// hooks/useWindowSize.js
import { useState, useEffect } from 'react';

function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });
  
  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }
    
    window.addEventListener('resize', handleResize);
    handleResize(); // Initial size
    
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return windowSize;
}

// Usage
function ResponsiveComponent() {
  const { width } = useWindowSize();
  
  return (
    <div>
      {width < 768 ? <MobileView /> : <DesktopView />}
    </div>
  );
}
```

---

### 🏗️ SYSTEM DESIGN & ARCHITECTURE

#### **6.8 Thiết kế Real-time Chat Application với Next.js?**

**Architecture:**

```
Frontend:              Backend:               Database:
Next.js App           Node.js/Next.js API    PostgreSQL (messages)
├── UI Components     ├── Socket.IO Server   Redis (real-time)
├── WebSocket Client  ├── REST API           S3 (file uploads)
└── State Mgmt        └── Auth Middleware    
                      
CDN: Vercel/CloudFlare
Message Queue: Redis/RabbitMQ
```

**Implementation:**

```javascript
// pages/api/socket.js - Socket.IO server
import { Server } from 'socket.io';

export default function SocketHandler(req, res) {
  if (res.socket.server.io) {
    res.end();
    return;
  }

  const io = new Server(res.socket.server);
  res.socket.server.io = io;

  io.on('connection', (socket) => {
    console.log('Client connected:', socket.id);
    
    // Join room
    socket.on('join-room', (roomId) => {
      socket.join(roomId);
      socket.to(roomId).emit('user-joined', socket.id);
    });
    
    // Send message
    socket.on('send-message', async ({ roomId, message, userId }) => {
      // Save to database
      const savedMessage = await db.message.create({
        data: { roomId, content: message, userId }
      });
      
      // Broadcast to room
      io.to(roomId).emit('new-message', savedMessage);
      
      // Update Redis cache
      await redis.lpush(`room:${roomId}:messages`, JSON.stringify(savedMessage));
    });
    
    // Typing indicator
    socket.on('typing', ({ roomId, userId }) => {
      socket.to(roomId).emit('user-typing', userId);
    });
    
    socket.on('disconnect', () => {
      console.log('Client disconnected:', socket.id);
    });
  });

  res.end();
}

// pages/chat/[roomId].js - Chat component
import { useEffect, useState } from 'react';
import io from 'socket.io-client';

let socket;

function ChatRoom({ roomId, currentUser }) {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [typing, setTyping] = useState([]);
  
  useEffect(() => {
    socketInitializer();
    
    return () => {
      if (socket) socket.disconnect();
    };
  }, [roomId]);
  
  const socketInitializer = async () => {
    await fetch('/api/socket');
    socket = io();
    
    socket.on('connect', () => {
      socket.emit('join-room', roomId);
    });
    
    socket.on('new-message', (message) => {
      setMessages(prev => [...prev, message]);
    });
    
    socket.on('user-typing', (userId) => {
      setTyping(prev => [...prev, userId]);
      setTimeout(() => {
        setTyping(prev => prev.filter(id => id !== userId));
      }, 3000);
    });
  };
  
  const sendMessage = () => {
    if (!input.trim()) return;
    
    socket.emit('send-message', {
      roomId,
      message: input,
      userId: currentUser.id
    });
    
    setInput('');
  };
  
  const handleTyping = () => {
    socket.emit('typing', { roomId, userId: currentUser.id });
  };
  
  return (
    <div className="chat-container">
      <div className="messages">
        {messages.map(msg => (
          <div key={msg.id} className={msg.userId === currentUser.id ? 'own' : 'other'}>
            <p>{msg.content}</p>
            <span>{new Date(msg.createdAt).toLocaleTimeString()}</span>
          </div>
        ))}
        {typing.length > 0 && (
          <div className="typing-indicator">
            {typing.length} user(s) typing...
          </div>
        )}
      </div>
      
      <div className="input-area">
        <input
          value={input}
          onChange={(e) => {
            setInput(e.target.value);
            handleTyping();
          }}
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
          placeholder="Type a message..."
        />
        <button onClick={sendMessage}>Send</button>
      </div>
    </div>
  );
}

// API route cho message history
// pages/api/messages/[roomId].js
export default async function handler(req, res) {
  const { roomId } = req.query;
  
  // Try Redis cache first
  const cached = await redis.lrange(`room:${roomId}:messages`, 0, 49);
  
  if (cached.length > 0) {
    return res.json(cached.map(m => JSON.parse(m)));
  }
  
  // Fallback to database
  const messages = await db.message.findMany({
    where: { roomId },
    orderBy: { createdAt: 'desc' },
    take: 50
  });
  
  // Cache for next time
  await redis.lpush(`room:${roomId}:messages`, ...messages.map(m => JSON.stringify(m)));
  
  res.json(messages);
}
```

**Scaling considerations:**
- Redis for presence/typing indicators
- Message queue for offline messages
- WebSocket load balancing (sticky sessions)
- CDN for static assets
- Database sharding by room/user

---

#### **6.9 Caching Strategy trong Next.js Application?**

**Multi-layer caching:**

```
1. Browser Cache (client-side)
   └── localStorage, sessionStorage
   
2. Next.js Cache (build-time)
   └── .next/cache
   
3. CDN Cache (edge)
   └── Vercel Edge, CloudFlare
   
4. Application Cache (server-side)
   └── Redis, Memcached
   
5. Database Cache
   └── Query cache, indexes
```

**Implementation:**

```javascript
// 1. Browser caching với SWR
import useSWR from 'swr';

function Profile() {
  const { data, error, isLoading } = useSWR(
    '/api/user',
    fetcher,
    {
      revalidateOnFocus: false,
      revalidateOnReconnect: false,
      dedupingInterval: 60000, // 1 minute
    }
  );
  
  return <div>{data?.name}</div>;
}

// 2. CDN caching với headers
// pages/api/data.js
export default async function handler(req, res) {
  const data = await fetchData();
  
  // Cache on CDN for 1 hour, stale-while-revalidate for 1 day
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=3600, stale-while-revalidate=86400'
  );
  
  res.json(data);
}

// 3. Redis caching
import { Redis } from '@upstash/redis';

const redis = Redis.fromEnv();

export default async function handler(req, res) {
  const cacheKey = `products:${req.query.category}`;
  
  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return res.json(cached);
  }
  
  // Fetch from database
  const products = await db.product.findMany({
    where: { category: req.query.category }
  });
  
  // Cache for 5 minutes
  await redis.set(cacheKey, products, { ex: 300 });
  
  res.json(products);
}

// 4. ISR caching
export async function getStaticProps() {
  const data = await fetchData();
  
  return {
    props: { data },
    revalidate: 60, // Regenerate every 60 seconds
  };
}

// 5. Request memoization (App Router)
import { cache } from 'react';

const getUser = cache(async (id) => {
  const user = await db.user.findUnique({ where: { id } });
  return user;
});

// Multiple calls to getUser(1) in same request only fetch once
```

**Cache invalidation strategies:**

```javascript
// 1. Time-based (TTL)
await redis.set('key', value, { ex: 3600 }); // 1 hour

// 2. Tag-based (Next.js 13+)
import { revalidateTag } from 'next/cache';

// In API route
export async function POST(request) {
  await createPost(request.body);
  revalidateTag('posts'); // Invalidate all 'posts' cache
  return Response.json({ success: true });
}

// 3. On-demand revalidation
// pages/api/revalidate.js
export default async function handler(req, res) {
  await res.revalidate('/blog');
  await res.revalidate('/blog/[slug]');
  return res.json({ revalidated: true });
}

// 4. Event-driven (Webhook)
// pages/api/webhook.js
export default async function handler(req, res) {
  const { event, data } = req.body;
  
  if (event === 'post.created') {
    await redis.del('posts:all');
    await res.revalidate('/blog');
  }
  
  res.json({ received: true });
}
```

---

### 💻 CODING CHALLENGES

#### **6.10 Implement Infinite Scroll với Virtualization**

```jsx
import { useState, useEffect, useRef, useCallback } from 'react';
import { FixedSizeList } from 'react-window';
import InfiniteLoader from 'react-window-infinite-loader';

function VirtualizedInfiniteList() {
  const [items, setItems] = useState([]);
  const [hasNextPage, setHasNextPage] = useState(true);
  const [isNextPageLoading, setIsNextPageLoading] = useState(false);
  
  const loadNextPage = async (startIndex, stopIndex) => {
    setIsNextPageLoading(true);
    
    try {
      const response = await fetch(
        `/api/items?start=${startIndex}&end=${stopIndex}`
      );
      const newItems = await response.json();
      
      setItems(prev => {
        const next = [...prev];
        newItems.forEach((item, index) => {
          next[startIndex + index] = item;
        });
        return next;
      });
      
      setHasNextPage(newItems.length === (stopIndex - startIndex));
    } finally {
      setIsNextPageLoading(false);
    }
  };
  
  const isItemLoaded = index => !!items[index];
  
  const itemCount = hasNextPage ? items.length + 1 : items.length;
  
  const Item = ({ index, style }) => {
    if (!isItemLoaded(index)) {
      return <div style={style}>Loading...</div>;
    }
    
    const item = items[index];
    return (
      <div style={style} className="item">
        <h3>{item.title}</h3>
        <p>{item.description}</p>
      </div>
    );
  };
  
  return (
    <InfiniteLoader
      isItemLoaded={isItemLoaded}
      itemCount={itemCount}
      loadMoreItems={isNextPageLoading ? () => {} : loadNextPage}
    >
      {({ onItemsRendered, ref }) => (
        <FixedSizeList
          height={600}
          itemCount={itemCount}
          itemSize={100}
          onItemsRendered={onItemsRendered}
          ref={ref}
          width="100%"
        >
          {Item}
        </FixedSizeList>
      )}
    </InfiniteLoader>
  );
}
```

---

#### **6.11 Form Validation với React Hook Form + Zod**

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  confirmPassword: z.string(),
  age: z.number().min(18, 'Must be 18 or older'),
  terms: z.boolean().refine(val => val === true, 'Must accept terms'),
}).refine(data => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ['confirmPassword'],
});

function RegistrationForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset,
  } = useForm({
    resolver: zodResolver(schema),
    defaultValues: {
      email: '',
      password: '',
      confirmPassword: '',
      age: 18,
      terms: false,
    },
  });
  
  const onSubmit = async (data) => {
    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      
      if (response.ok) {
        alert('Registration successful!');
        reset();
      }
    } catch (error) {
      console.error(error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} placeholder="Email" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <div>
        <input
          {...register('password')}
          type="password"
          placeholder="Password"
        />
        {errors.password && <span>{errors.password.message}</span>}
      </div>
      
      <div>
        <input
          {...register('confirmPassword')}
          type="password"
          placeholder="Confirm Password"
        />
        {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}
      </div>
      
      <div>
        <input
          {...register('age', { valueAsNumber: true })}
          type="number"
          placeholder="Age"
        />
        {errors.age && <span>{errors.age.message}</span>}
      </div>
      
      <div>
        <label>
          <input {...register('terms')} type="checkbox" />
          Accept Terms & Conditions
        </label>
        {errors.terms && <span>{errors.terms.message}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Register'}
      </button>
    </form>
  );
}
```

---

## 🎯 BEHAVIORAL QUESTIONS

### **6.12 Debugging Strategies**

**Q: Describe your approach to debugging a production issue?**

**A: Systematic approach:**

1. **Reproduce the issue**
   - Get exact steps to reproduce
   - Check environment (browser, device)
   - Review error logs, Sentry reports

2. **Isolate the problem**
   - Binary search through code
   - Use console.log / debugger
   - Check recent changes (git blame)

3. **Fix and verify**
   - Write test to cover the bug
   - Implement fix
   - Verify in all environments

4. **Post-mortem**
   - Document root cause
   - Add monitoring/alerts
   - Share learnings with team

**Example:**

```javascript
// Production issue: Users can't checkout

// 1. Check error monitoring (Sentry)
// Error: "Cannot read property 'total' of undefined"

// 2. Add defensive logging
export async function checkout(cart) {
  console.log('Checkout called with:', cart);
  
  if (!cart || !cart.items) {
    console.error('Invalid cart:', cart);
    throw new Error('Invalid cart data');
  }
  
  const total = cart.items.reduce((sum, item) => {
    console.log('Processing item:', item);
    return sum + (item.price * item.quantity);
  }, 0);
  
  // ... rest of checkout
}

// 3. Found: Some users have null items in cart
// 4. Fix: Add validation
const validItems = cart.items.filter(item => item && item.price);

// 5. Add test
test('handles null items in cart', () => {
  const cart = { items: [null, { price: 10, quantity: 1 }] };
  expect(() => checkout(cart)).not.toThrow();
});
```

---

### **6.13 Code Review Best Practices**

**Q: What do you look for in code reviews?**

**A: Checklist:**

```markdown
### Functionality
- [ ] Code does what it's supposed to do
- [ ] Edge cases handled
- [ ] Error handling present

### Code Quality
- [ ] Readable and maintainable
- [ ] Follows project conventions
- [ ] No code duplication
- [ ] Proper naming

### Performance
- [ ] No unnecessary re-renders
- [ ] Database queries optimized
- [ ] Images optimized
- [ ] Bundle size impact

### Security
- [ ] Input validation
- [ ] No exposed secrets
- [ ] Proper authentication
- [ ] XSS/CSRF protection

### Testing
- [ ] Tests written and passing
- [ ] Test coverage adequate
- [ ] Tests are meaningful

### Documentation
- [ ] Complex logic explained
- [ ] API changes documented
- [ ] README updated if needed
```

**Good review comment examples:**

```javascript
// ❌ Bad comment
"This is wrong"

// ✅ Good comment
"This could cause a memory leak because the event listener isn't cleaned up. 
Consider using useEffect with a cleanup function:

useEffect(() => {
  const handler = () => { ... };
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
}, []);
"

// ✅ Praising good code
"Nice use of useMemo here! This will prevent unnecessary recalculations."

// ✅ Asking questions
"Have we considered what happens if this API call fails? 
Should we show an error message to the user?"
```

---

## 📚 RESOURCES & STUDY PLAN

### **Study Plan - 2 Weeks**

**Week 1: Fundamentals**
- Day 1-2: React Hooks (useState, useEffect, useMemo, useCallback)
- Day 3-4: Next.js Rendering (SSG, SSR, ISR)
- Day 5: Performance Optimization (Images, Bundle)
- Day 6-7: Practice coding challenges

**Week 2: Advanced**
- Day 8-9: App Router, Server Components
- Day 10: Middleware, Caching
- Day 11: System Design
- Day 12: Mock interviews
- Day 13-14: Review weak areas

### **Mock Interview Questions**

```markdown
## Round 1: Technical Fundamentals (30 min)
1. Explain React rendering lifecycle
2. SSG vs SSR vs ISR trade-offs
3. How does next/image optimize performance?
4. What is tree shaking?
5. Live coding: Implement debounce hook

## Round 2: System Design (45 min)
1. Design Instagram-like feed
2. Real-time notification system
3. Image gallery with 10,000+ images
4. Caching strategy for e-commerce

## Round 3: Coding Challenge (60 min)
1. Infinite scroll with API
2. Form with complex validation
3. Optimize slow component
4. Debug production issue
```

---

**Good luck với interview! 🚀**
