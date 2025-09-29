# KẾ HOẠCH CHUẨN BỊ PHỎNG VẤN IGNISOFT - FRONT-END DEVELOPER

## 📋 PHÂN TÍCH JOB DESCRIPTION

### YÊU CẦU CHÍNH:
- **Frontend**: HTML/CSS/SCSS, Responsive layout
- **JavaScript/TypeScript**: Vững chắc cho logic front-end và tích hợp backend
- **Frameworks**: ReactJS, VueJS
- **SSR Frameworks**: NextJS, NuxtJS
- **API Integration**: RESTful API, HTTP methods, JSON processing
- **Collaboration**: Phối hợp với backend, UI/UX, QA teams
- **Quality**: Code review, best practices, codebase optimization

### ĐIỂM CỘNG (NICE TO HAVE):
- **DevOps**: Linux server, terminal, deploy applications
- **CI/CD**: Jenkins, GitHub Actions, GitLab CI
- **Cloud**: Digital Ocean, AWS, Azure, Google Cloud Platform

---

## 🎯 KẾ HOẠCH CHUẨN BỊ 7 NGÀY

### **NGÀY 1-2: FRONTEND CORE (HTML/CSS/JS/TS)**
#### Buổi sáng (3-4 tiếng)
- [ ] **HTML Semantic & Accessibility**
  - HTML5 semantic elements
  - ARIA labels, accessibility best practices
  - SEO fundamentals

- [ ] **CSS/SCSS Advanced**
  - Flexbox, CSS Grid
  - Responsive design patterns
  - CSS-in-JS approaches
  - SCSS features (mixins, variables, nesting)

#### Buổi chiều (2-3 tiếng)
- [ ] **JavaScript ES6+**
  - Arrow functions, destructuring, spread operator
  - Promises, async/await
  - Module system (import/export)
  - DOM manipulation

- [ ] **TypeScript Fundamentals**
  - Basic types, interfaces, generics
  - Type assertions vs type guards
  - Utility types (Partial, Pick, Omit)

**Thực hành**: Build một responsive landing page với TypeScript

---

### **NGÀY 3-4: REACT.JS DEEP DIVE**
#### Buổi sáng (4 tiếng)
- [ ] **React Fundamentals**
  - Functional vs Class Components
  - JSX và reconciliation mechanism
  - Props, State, Event handling

- [ ] **React Hooks**
  - useState, useEffect, useContext
  - useCallback, useMemo performance optimization
  - Custom hooks pattern

#### Buổi chiều (3 tiếng)
- [ ] **Advanced React**
  - Context API vs Redux
  - Error boundaries
  - React.memo(), performance optimization
  - Large list virtualization

**Thực hành**: Build Todo App với hooks + performance optimization

---

### **NGÀY 5: NEXT.JS FRAMEWORK**
#### Buổi sáng (4 tiếng)
- [ ] **Next.js Rendering Strategies**
  - SSG vs SSR vs CSR vs ISR
  - getStaticProps, getServerSideProps
  - Dynamic routing, catch-all routes

- [ ] **Next.js Features**
  - App Router vs Pages Router
  - API routes design
  - Image optimization
  - Middleware for authentication

#### Buổi chiều (3 tiếng)
- [ ] **SEO & Performance**
  - Meta tags, structured data
  - Core Web Vitals optimization
  - Bundle optimization

**Thực hành**: Build blog với SSG + authentication middleware

---

### **NGÀY 6: API INTEGRATION & DEVOPS BASICS**
#### Buổi sáng (3 tiếng)
- [ ] **RESTful API Integration**
  - HTTP methods, status codes
  - Error handling patterns
  - Data fetching with SWR/React Query
  - Authentication (JWT, cookies)

#### Buổi chiều (3 tiếng)
- [ ] **DevOps Fundamentals**
  - Basic Linux commands
  - Docker basics (containerization)
  - CI/CD with GitHub Actions
  - Deployment strategies

**Thực hành**: Setup CI/CD pipeline cho React app

---

### **NGÀY 7: SYSTEM DESIGN & SOFT SKILLS**
#### Buổi sáng (2 tiếng)
- [ ] **System Design cho Frontend**
  - Component architecture
  - State management strategies
  - Performance optimization patterns
  - Caching strategies

#### Buổi chiều (2 tiếng)
- [ ] **Soft Skills Preparation**
  - Code review best practices
  - Team collaboration experience
  - Learning and growth mindset
  - Problem-solving approach

**Mock Interview**: Practice với câu hỏi system design

---

## 🔥 CÂU HỎI THƯỜNG GẶP & CHUẨN BỊ TRẢ LỜI

### **1. TECHNICAL QUESTIONS**

#### **React/Next.js**
- **Q**: "Giải thích sự khác biệt giữa SSG, SSR và CSR trong Next.js?"
- **Chuẩn bị**: Hiểu rõ use cases, trade-offs, và có ví dụ cụ thể

- **Q**: "Khi nào dùng useCallback vs useMemo?"
- **Chuẩn bị**: Có ví dụ performance optimization thực tế

- **Q**: "Cách optimize large list rendering?"
- **Chuẩn bị**: Virtualization, React Window, pagination strategies

#### **API Integration**
- **Q**: "Cách handle API errors trong React app?"
- **Chuẩn bị**: Error boundaries, try-catch patterns, user experience

- **Q**: "JWT authentication workflow?"
- **Chuẩn bị**: Token storage, refresh mechanism, security considerations

#### **Performance**
- **Q**: "Cách optimize Core Web Vitals?"
- **Chuẩn bị**: Image optimization, code splitting, lazy loading

### **2. SYSTEM DESIGN QUESTIONS**

#### **E-commerce Website Architecture**
```
Frontend (Next.js)
├── SSG: Product catalog, SEO pages
├── SSR: User dashboard, personalized content  
├── CSR: Shopping cart, real-time features
└── API Integration: Payment, inventory

Performance Considerations:
- CDN for static assets
- Image optimization
- Code splitting by routes
- Caching strategies
```

#### **Component Architecture**
```
src/
├── components/
│   ├── ui/ (Button, Input, Modal)
│   ├── features/ (UserProfile, ProductCard)
│   └── layout/ (Header, Footer, Sidebar)
├── hooks/ (useAuth, useApi, useLocalStorage)
├── contexts/ (AuthContext, ThemeContext)
├── utils/ (validation, formatting, api)
└── types/ (TypeScript definitions)
```

### **3. BEHAVIORAL QUESTIONS**

- **Q**: "Describe a challenging bug you fixed"
- **Chuẩn bị**: STAR method, technical details, learning outcome

- **Q**: "How do you handle code reviews?"
- **Chuẩn bị**: Constructive feedback examples, collaboration approach

- **Q**: "How do you stay updated with React ecosystem?"
- **Chuẩn bị**: Learning resources, community involvement, practice projects

---

## 💻 CODING CHALLENGES PRACTICE

### **Level 1: Fundamentals**
1. **Todo App với TypeScript**
   - CRUD operations
   - Local storage persistence
   - Filtering (all, active, completed)

2. **Weather App với API**
   - External API integration
   - Error handling
   - Loading states

### **Level 2: Advanced**
1. **Infinite Scroll Component**
   - Intersection Observer
   - Performance optimization
   - Error boundary

2. **Real-time Chat Interface**
   - WebSocket integration
   - Message history
   - Online status

### **Level 3: System Design**
1. **E-commerce Product Catalog**
   - Search and filtering
   - Pagination
   - Shopping cart state management

---

## 🛠️ SETUP DEVELOPMENT ENVIRONMENT

### **Essential Tools:**
```bash
# Frontend tooling
npm create next-app@latest my-project --typescript --tailwind --app
npm install @tanstack/react-query axios zod

# Development tools  
npm install -D @types/node @typescript-eslint/eslint-plugin
npm install -D jest @testing-library/react playwright

# Optional: Docker for deployment practice
docker init
```

### **VS Code Extensions:**
- TypeScript Hero
- ES7+ React/Redux/Next.js snippets
- Tailwind CSS IntelliSense
- Auto Rename Tag
- GitLens

---

## ✅ CHECKLIST NGÀY PHỎNG VẤN

### **Technical Preparation:**
- [ ] Review key concepts: React Hooks, Next.js rendering
- [ ] Practice coding challenges trên whiteboard/laptop
- [ ] Prepare questions about tech stack, team structure
- [ ] Review recent projects và challenges solved

### **Soft Skills Preparation:**
- [ ] STAR method examples ready
- [ ] Questions about company culture, growth opportunities
- [ ] Career goals alignment with role
- [ ] Salary expectation research

### **Materials:**
- [ ] Updated CV với relevant projects
- [ ] GitHub profile với clean, documented code
- [ ] Portfolio website (nếu có)
- [ ] Questions list for interviewer

---

## 🎯 KEY SUCCESS FACTORS

### **Technical Excellence:**
1. **Show code quality awareness** - Clean code, proper naming, comments
2. **Performance mindset** - Always consider optimization opportunities
3. **User experience focus** - Accessibility, responsive design, error handling
4. **Continuous learning** - Mention latest React features, ecosystem trends

### **Collaboration Skills:**
1. **Communication** - Explain technical concepts clearly
2. **Problem-solving** - Break down complex problems systematically
3. **Adaptability** - Show willingness to learn new technologies
4. **Quality focus** - Code review practices, testing mindset

---

## 📚 TÀI LIỆU THAM KHẢO NHANH

- **React Docs**: https://react.dev/
- **Next.js Docs**: https://nextjs.org/docs
- **TypeScript Handbook**: https://typescriptlang.org/docs
- **MDN Web Docs**: https://developer.mozilla.org/
- **Web.dev Performance**: https://web.dev/performance/

**Chúc bạn thành công trong buổi phỏng vấn! 🚀**
