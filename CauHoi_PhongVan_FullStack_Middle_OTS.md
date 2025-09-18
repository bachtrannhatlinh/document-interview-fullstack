# CÂU HỎI PHỎNG VẤN MIDDLE FULLSTACK DEVELOPER
## One Ocean Network Express (OTS)

### 📋 THÔNG TIN VỊ TRÍ
- **Vị trí**: Middle Full-stack Developer (Node.js, React.js, Next.js)
- **Level**: Middle (2-4 năm kinh nghiệm)
- **Công ty**: One Ocean Network Express
- **Địa điểm**: HCM City

---

## 🔥 CÂU HỎI KỸ THUẬT CƠ BẢN

### **A. NODE.JS CORE CONCEPTS**

#### **1. Event Loop & Asynchronous**
**Q: Giải thích Event Loop trong Node.js hoạt động như thế nào?**
- Call Stack, Callback Queue, Microtask Queue
- Thứ tự ưu tiên thực thi
- setTimeout vs setImmediate vs process.nextTick

**Q: Sự khác biệt giữa blocking và non-blocking operations?**
- I/O operations
- CPU-bound vs I/O-bound tasks

#### **2. Modules & Package Management**
**Q: CommonJS vs ES6 Modules trong Node.js?**
- require() vs import/export
- Module loading mechanism
- Circular dependencies

**Q: npm vs yarn vs pnpm?**
- Package lock files
- Dependency resolution
- Workspace management

#### **3. Error Handling**
**Q: Cách xử lý errors trong Node.js applications?**
- Try-catch với async/await
- Promise.catch()
- Global error handlers
- Process exit strategies

### **B. REACT.JS FUNDAMENTALS**

#### **1. Components & JSX**
**Q: Functional vs Class Components - ưu nhược điểm?**
- Hooks vs Lifecycle methods
- Performance differences
- Code reusability

**Q: Virtual DOM hoạt động như thế nào?**
- Reconciliation process
- Diffing algorithm
- Performance optimization

#### **2. Hooks Deep Dive**
**Q: useEffect dependencies array hoạt động ra sao?**
- Dependency comparison
- Cleanup functions
- Effect optimization

**Q: Khi nào dùng useCallback vs useMemo?**
- Function memoization
- Value memoization
- Performance implications

**Q: Tạo custom hooks như thế nào?**
- Rules of hooks
- State sharing
- Business logic extraction

#### **3. State Management**
**Q: Redux vs Context API - khi nào dùng cái nào?**
- State complexity
- Performance considerations
- Developer experience

**Q: Redux Toolkit vs Redux classic?**
- Boilerplate reduction
- Immutability handling
- DevTools integration

### **C. NEXT.JS FRAMEWORK**

#### **1. Rendering Strategies**
**Q: SSG vs SSR vs CSR - trade-offs của từng approach?**
- Performance implications
- SEO considerations
- Use cases

**Q: Incremental Static Regeneration (ISR) hoạt động như thế nào?**
- Revalidation strategies
- Cache management
- Build optimization

#### **2. Routing & API**
**Q: Dynamic routing trong Next.js?**
- File-based routing
- Catch-all routes
- Optional parameters

**Q: API Routes vs External API services?**
- Pros and cons
- Performance considerations
- Security implications

---

## 🚀 CÂU HỎI NÂNG CAO

### **D. PERFORMANCE OPTIMIZATION**

#### **1. Frontend Performance**
**Q: Cách optimize React application performance?**
- Code splitting strategies
- Bundle analysis
- Lazy loading components
- Image optimization

**Q: Core Web Vitals và cách cải thiện?**
- LCP, FID, CLS metrics
- Performance monitoring
- Optimization techniques

#### **2. Backend Performance**
**Q: Node.js performance optimization strategies?**
- Clustering và Worker Threads
- Memory management
- Database connection pooling
- Caching strategies

### **E. SECURITY & BEST PRACTICES**

#### **1. Web Security**
**Q: Common security vulnerabilities trong web applications?**
- XSS, CSRF, SQL Injection
- JWT security
- Input validation
- HTTPS và security headers

#### **2. Authentication & Authorization**
**Q: JWT vs Session-based authentication?**
- Token storage
- Refresh token strategy
- Security considerations

**Q: Implement role-based access control (RBAC)?**
- Permission models
- Middleware design
- Frontend route protection

### **F. DATABASE & API DESIGN**

#### **1. Database Design**
**Q: SQL vs NoSQL - khi nào dùng cái nào?**
- Data relationships
- Scalability requirements
- Transaction needs

**Q: Database indexing và query optimization?**
- Index types
- Query performance
- N+1 problems

#### **2. API Design**
**Q: RESTful API best practices?**
- HTTP methods và status codes
- Resource naming
- Pagination strategies
- Error responses

**Q: GraphQL vs REST API?**
- Overfetching/underfetching
- Type safety
- Caching strategies

---

## 💻 CODING CHALLENGES

### **Challenge 1: Authentication System**
```javascript
// Implement JWT authentication with refresh token
// Requirements:
// - Login/logout endpoints
// - Protected routes middleware
// - Token refresh mechanism
// - Rate limiting for login attempts
```

### **Challenge 2: Real-time Features**
```javascript
// Build a real-time notification system
// Requirements:
// - WebSocket connection
// - User-specific notifications
// - Offline message queuing
// - Connection recovery
```

### **Challenge 3: Data Processing**
```javascript
// Create a CSV file processor API
// Requirements:
// - Stream processing for large files
// - Data validation
// - Progress tracking
// - Error handling and rollback
```

### **Challenge 4: React Component**
```javascript
// Build an infinite scroll component
// Requirements:
// - Intersection Observer
// - Virtual scrolling for performance
// - Loading states
// - Error boundaries
```

### **Challenge 5: Next.js Application**
```javascript
// Create a blog with admin panel
// Requirements:
// - SSG for public pages
// - SSR for admin pages
// - Dynamic routes
// - SEO optimization
```

---

## 🏗️ SYSTEM DESIGN QUESTIONS

### **1. E-commerce Platform**
**Q: Thiết kế một e-commerce platform có thể scale?**

**Yêu cầu:**
- Product catalog với search
- Shopping cart và checkout
- User accounts và order history
- Admin panel

**Considerations:**
- Database design
- Caching strategies
- Payment integration
- Security measures
- Performance optimization

### **2. Social Media Dashboard**
**Q: Thiết kế dashboard cho social media management?**

**Features:**
- Multiple platform integration
- Post scheduling
- Analytics và reporting
- Team collaboration

**Technical Challenges:**
- Real-time updates
- Data aggregation
- Rate limiting từ external APIs
- File upload và processing

### **3. Microservices Architecture**
**Q: Khi nào nên chuyển từ monolith sang microservices?**

**Factors:**
- Team size và organization
- Deployment requirements
- Scalability needs
- Technical debt

---

## 🎯 CÂU HỎI HÀNH VI

### **A. Technical Leadership**

**Q: Kể về lần bạn phải make technical decision quan trọng?**
- Decision process
- Stakeholder communication
- Risk assessment
- Outcome evaluation

**Q: Cách bạn handle disagreements trong technical discussions?**
- Evidence-based arguments
- Compromise solutions
- Team consensus building

### **B. Project Management**

**Q: Làm thế nào để estimate development time accurately?**
- Breaking down tasks
- Risk factors
- Buffer time
- Historical data

**Q: Cách bạn handle changing requirements?**
- Agile methodologies
- Stakeholder communication
- Technical flexibility

### **C. Learning & Growth**

**Q: Kể về công nghệ mới bạn đã học gần đây?**
- Learning process
- Application in projects
- Knowledge sharing

**Q: Cách bạn stay updated với rapidly changing tech landscape?**
- Information sources
- Practical application
- Community involvement

---

## 📝 TIPS TRẢ LỜI PHỎNG VẤN

### **1. Technical Questions**
- **Think out loud**: Giải thích approach và reasoning
- **Start simple**: Implement basic solution trước
- **Consider edge cases**: Error handling, validation
- **Discuss trade-offs**: Performance vs complexity
- **Be honest**: Nếu không biết, đề xuất cách tìm hiểu

### **2. Coding Challenges**
- **Clarify requirements**: Ask questions trước khi code
- **Write clean code**: Readable và maintainable
- **Test your solution**: Edge cases và error conditions
- **Optimize if needed**: Time và space complexity
- **Explain your approach**: Why choose this solution

### **3. System Design**
- **Ask clarifying questions**: Scale, requirements, constraints
- **Start high-level**: Overall architecture trước
- **Drill down**: Detailed components
- **Consider non-functional requirements**: Security, performance, scalability
- **Discuss alternatives**: Different approaches và trade-offs

### **4. Behavioral Questions**
- **Use STAR method**: Situation, Task, Action, Result
- **Be specific**: Concrete examples
- **Show growth**: What you learned
- **Highlight collaboration**: Team work skills
- **Demonstrate impact**: Measurable results

---

## 🔍 QUESTIONS TO ASK THE INTERVIEWER

### **Technical & Architecture**
- "Can you describe the current tech stack và architecture?"
- "What are the biggest technical challenges the team is facing?"
- "How does the team handle code reviews và deployment?"
- "What's the testing strategy for the applications?"

### **Team & Culture**
- "How is the development team structured?"
- "What does a typical day look like for this role?"
- "How does the team handle knowledge sharing?"
- "What opportunities are there for learning và growth?"

### **Product & Business**
- "What are the main products/services the team works on?"
- "How does the engineering team collaborate with other departments?"
- "What are the company's plans for technical innovation?"

---

## 📚 LAST-MINUTE REVIEW CHECKLIST

### **Day Before Interview**
- [ ] Review Node.js Event Loop concepts
- [ ] Practice React Hooks examples
- [ ] Go through Next.js rendering strategies
- [ ] Review recent projects và be ready to discuss
- [ ] Prepare questions to ask interviewer
- [ ] Test coding environment setup

### **Day of Interview**
- [ ] Review authentication/authorization concepts
- [ ] Quick practice with coding challenges
- [ ] Review system design principles
- [ ] Prepare behavioral question answers
- [ ] Bring portfolio/projects to showcase

---

**Chúc bạn phỏng vấn thành công! 🚀**

---

## 📞 THÔNG TIN LIÊN HỆ
Nếu cần hỗ trợ thêm trong việc chuẩn bị phỏng vấn, hãy liên hệ để được tư vấn chi tiết hơn.
