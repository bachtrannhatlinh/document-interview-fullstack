# CÂU HỎI PHỎNG VẤN REACT.JS & NEXT.JS

## ⚛️ REACT.JS FUNDAMENTALS

### 1. **Components & JSX**
**Q: Sự khác biệt giữa Functional và Class Components?**

**A:**
- **Functional Components** (Modern):
  - Sử dụng Hooks
  - Code ngắn gọn hơn
  - Performance tốt hơn
  - Dễ test và maintain

- **Class Components** (Legacy):
  - Có lifecycle methods
  - State management với this.state
  - Phức tạp hơn

**Ví dụ:**
```jsx
// Functional Component
function Welcome({ name }) {
    return <h1>Hello, {name}</h1>;
}

// Class Component
class Welcome extends React.Component {
    render() {
        return <h1>Hello, {this.props.name}</h1>;
    }
}
```

---

### 2. **Hooks**
**Q: Giải thích useState và useEffect?**

**A:**
- **useState**: Quản lý state trong functional component
- **useEffect**: Thay thế lifecycle methods (componentDidMount, componentDidUpdate, componentWillUnmount)

**Ví dụ:**
```jsx
import React, { useState, useEffect } from 'react';

function Counter() {
    const [count, setCount] = useState(0);
    const [name, setName] = useState('');

    // ComponentDidMount + ComponentDidUpdate
    useEffect(() => {
        document.title = `You clicked ${count} times`;
    }, [count]); // Dependency array

    // ComponentWillUnmount
    useEffect(() => {
        const timer = setInterval(() => {
            console.log('Timer tick');
        }, 1000);

        return () => clearInterval(timer); // Cleanup
    }, []);

    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={() => setCount(count + 1)}>
                Click me
            </button>
            <input 
                value={name} 
                onChange={(e) => setName(e.target.value)} 
            />
        </div>
    );
}
```

---

### 3. **State Management**
**Q: Khi nào nên dùng Redux vs Context API?**

**A:**
- **Redux**: 
  - State phức tạp, nhiều components
  - Cần time-travel debugging
  - Predictable state updates
  - Middleware support

- **Context API**:
  - State đơn giản, ít components
  - Tránh prop drilling
  - Built-in React

**Ví dụ Context API:**
```jsx
// Context
const ThemeContext = React.createContext();

// Provider
function ThemeProvider({ children }) {
    const [theme, setTheme] = useState('light');
    
    return (
        <ThemeContext.Provider value={{ theme, setTheme }}>
            {children}
        </ThemeContext.Provider>
    );
}

// Consumer
function ThemedButton() {
    const { theme, setTheme } = useContext(ThemeContext);
    
    return (
        <button 
            style={{ background: theme === 'light' ? '#fff' : '#000' }}
            onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
        >
            Toggle Theme
        </button>
    );
}
```

---

### 4. **Performance Optimization**
**Q: React.memo(), useMemo(), useCallback() khác nhau như thế nào?**

**A:**
- **React.memo()**: Memoize component, re-render khi props thay đổi
- **useMemo()**: Memoize expensive calculations
- **useCallback()**: Memoize functions

**Ví dụ:**
```jsx
// React.memo - Component memoization
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
    return <div>{data.name}</div>;
});

// useMemo - Value memoization
function ExpensiveCalculation({ items }) {
    const expensiveValue = useMemo(() => {
        return items.reduce((sum, item) => sum + item.value, 0);
    }, [items]); // Recalculate when items change

    return <div>Total: {expensiveValue}</div>;
}

// useCallback - Function memoization
function Parent({ items }) {
    const [count, setCount] = useState(0);
    
    const handleClick = useCallback(() => {
        console.log('Button clicked');
    }, []); // Function reference stays same

    return (
        <div>
            <button onClick={() => setCount(count + 1)}>
                Count: {count}
            </button>
            <ExpensiveComponent onUpdate={handleClick} />
        </div>
    );
}
```

---

## 🚀 NEXT.JS FRAMEWORK

### 5. **Rendering Strategies**
**Q: SSG vs SSR vs CSR - Khi nào dùng cái nào?**

**A:**
- **SSG (Static Site Generation)**:
  - Content tĩnh, ít thay đổi
  - SEO tốt nhất
  - Performance cao nhất
  - Build time generation

- **SSR (Server-Side Rendering)**:
  - Content động, personalization
  - SEO tốt
  - Server load cao
  - Runtime generation

- **CSR (Client-Side Rendering)**:
  - SPA, interactive
  - SEO kém
  - Performance phụ thuộc client
  - Runtime generation

**Ví dụ Next.js:**
```jsx
// SSG - getStaticProps
export async function getStaticProps() {
    const data = await fetchData();
    return {
        props: { data },
        revalidate: 60 // ISR - revalidate every 60 seconds
    };
}

// SSR - getServerSideProps
export async function getServerSideProps(context) {
    const { req, res } = context;
    const data = await fetchDataFromAPI(req);
    
    return {
        props: { data }
    };
}

// CSR - useEffect
function ClientSideComponent() {
    const [data, setData] = useState(null);
    
    useEffect(() => {
        fetchData().then(setData);
    }, []);
    
    return <div>{data?.name}</div>;
}
```

---

### 6. **Routing**
**Q: Dynamic routing trong Next.js?**

**A:**
- **File-based routing**: Tạo file/folder structure
- **Dynamic routes**: Sử dụng `[param].js`
- **Catch-all routes**: `[...slug].js`
- **Optional catch-all**: `[[...slug]].js`

**Ví dụ:**
```
pages/
  index.js          // /
  about.js          // /about
  blog/
    index.js        // /blog
    [slug].js       // /blog/hello-world
    [...slug].js    // /blog/2023/01/hello-world
    [[...slug]].js  // /blog (optional catch-all)
```

**Dynamic route implementation:**
```jsx
// pages/blog/[slug].js
export async function getStaticPaths() {
    const posts = await fetchPosts();
    
    const paths = posts.map(post => ({
        params: { slug: post.slug }
    }));
    
    return {
        paths,
        fallback: 'blocking' // or true, false
    };
}

export async function getStaticProps({ params }) {
    const post = await fetchPost(params.slug);
    
    return {
        props: { post },
        revalidate: 60
    };
}

function BlogPost({ post }) {
    return (
        <article>
            <h1>{post.title}</h1>
            <div>{post.content}</div>
        </article>
    );
}
```

---

### 7. **API Routes**
**Q: Tạo REST API với Next.js API Routes?**

**A:**
```jsx
// pages/api/users/[id].js
export default async function handler(req, res) {
    const { id } = req.query;
    const { method } = req;
    
    switch (method) {
        case 'GET':
            try {
                const user = await getUserById(id);
                res.status(200).json(user);
            } catch (error) {
                res.status(404).json({ error: 'User not found' });
            }
            break;
            
        case 'PUT':
            try {
                const updatedUser = await updateUser(id, req.body);
                res.status(200).json(updatedUser);
            } catch (error) {
                res.status(400).json({ error: 'Invalid data' });
            }
            break;
            
        case 'DELETE':
            try {
                await deleteUser(id);
                res.status(200).json({ message: 'User deleted' });
            } catch (error) {
                res.status(400).json({ error: 'Delete failed' });
            }
            break;
            
        default:
            res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
            res.status(405).end(`Method ${method} Not Allowed`);
    }
}
```

---

## 🏗️ SYSTEM DESIGN QUESTIONS

### 8. **E-commerce Architecture**
**Q: Thiết kế e-commerce website với Next.js?**

**A:**
**Architecture:**
- **Frontend**: Next.js (SSG/SSR)
- **Backend**: Node.js API
- **Database**: PostgreSQL + Redis
- **CDN**: Vercel/CloudFlare
- **Payment**: Stripe/PayPal
- **Search**: Elasticsearch

**Key Features:**
- Product catalog (SSG)
- User authentication (SSR)
- Shopping cart (Client-side)
- Checkout process (SSR)
- Order management (SSR)

---

### 9. **Performance Optimization**
**Q: Cách optimize Next.js application?**

**A:**
- **Image Optimization**: `next/image`
- **Font Optimization**: `next/font`
- **Code Splitting**: Dynamic imports
- **Bundle Analysis**: `@next/bundle-analyzer`
- **Caching**: ISR, CDN
- **Lazy Loading**: Dynamic components

**Ví dụ:**
```jsx
// Image optimization
import Image from 'next/image';

function ProductImage({ src, alt }) {
    return (
        <Image
            src={src}
            alt={alt}
            width={500}
            height={500}
            placeholder="blur"
            blurDataURL="data:image/jpeg;base64,..."
        />
    );
}

// Code splitting
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
    loading: () => <p>Loading...</p>,
    ssr: false
});
```

---

## 💻 CODING CHALLENGES

### 10. **Todo App với Hooks**
```jsx
import React, { useState, useEffect } from 'react';

function TodoApp() {
    const [todos, setTodos] = useState([]);
    const [input, setInput] = useState('');
    const [filter, setFilter] = useState('all');

    // Load from localStorage
    useEffect(() => {
        const saved = localStorage.getItem('todos');
        if (saved) {
            setTodos(JSON.parse(saved));
        }
    }, []);

    // Save to localStorage
    useEffect(() => {
        localStorage.setItem('todos', JSON.stringify(todos));
    }, [todos]);

    const addTodo = () => {
        if (input.trim()) {
            setTodos([...todos, {
                id: Date.now(),
                text: input,
                completed: false
            }]);
            setInput('');
        }
    };

    const toggleTodo = (id) => {
        setTodos(todos.map(todo =>
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
        ));
    };

    const deleteTodo = (id) => {
        setTodos(todos.filter(todo => todo.id !== id));
    };

    const filteredTodos = todos.filter(todo => {
        if (filter === 'active') return !todo.completed;
        if (filter === 'completed') return todo.completed;
        return true;
    });

    return (
        <div>
            <input
                value={input}
                onChange={(e) => setInput(e.target.value)}
                onKeyPress={(e) => e.key === 'Enter' && addTodo()}
                placeholder="Add todo..."
            />
            <button onClick={addTodo}>Add</button>
            
            <div>
                <button onClick={() => setFilter('all')}>All</button>
                <button onClick={() => setFilter('active')}>Active</button>
                <button onClick={() => setFilter('completed')}>Completed</button>
            </div>
            
            <ul>
                {filteredTodos.map(todo => (
                    <li key={todo.id}>
                        <input
                            type="checkbox"
                            checked={todo.completed}
                            onChange={() => toggleTodo(todo.id)}
                        />
                        <span style={{
                            textDecoration: todo.completed ? 'line-through' : 'none'
                        }}>
                            {todo.text}
                        </span>
                        <button onClick={() => deleteTodo(todo.id)}>Delete</button>
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

---

### 11. **Infinite Scroll Component**
```jsx
import React, { useState, useEffect, useCallback, useRef } from 'react';

function InfiniteScroll({ fetchData, renderItem }) {
    const [items, setItems] = useState([]);
    const [loading, setLoading] = useState(false);
    const [hasMore, setHasMore] = useState(true);
    const [page, setPage] = useState(1);
    const observer = useRef();

    const lastItemRef = useCallback(node => {
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
        try {
            const newData = await fetchData(page);
            setItems(prev => [...prev, ...newData.items]);
            setHasMore(newData.hasMore);
            setPage(prev => prev + 1);
        } catch (error) {
            console.error('Error loading more data:', error);
        } finally {
            setLoading(false);
        }
    };

    useEffect(() => {
        loadMore();
    }, []);

    return (
        <div>
            {items.map((item, index) => (
                <div
                    key={item.id}
                    ref={index === items.length - 1 ? lastItemRef : null}
                >
                    {renderItem(item)}
                </div>
            ))}
            {loading && <div>Loading...</div>}
            {!hasMore && <div>No more items</div>}
        </div>
    );
}
```

---

## 🎯 CÂU HỎI HÀNH VI

### 12. **Team Collaboration**
**Q: Làm thế nào để code review hiệu quả?**

**A:**
- **Focus on code, not person**
- **Be constructive và specific**
- **Suggest improvements, not just problems**
- **Ask questions để understand context**
- **Praise good practices**

### 13. **Learning & Growth**
**Q: Cách bạn stay updated với React ecosystem?**

**A:**
- **Official docs**: React, Next.js
- **Communities**: Reddit r/reactjs, Discord
- **Newsletters**: React Status, This Week in React
- **Conferences**: React Conf, Next.js Conf
- **Open source**: Contribute to projects
- **Practice**: Build projects, solve problems

---

## 📝 TIPS TRẢ LỜI

1. **Show code examples**: Luôn có ví dụ cụ thể
2. **Explain trade-offs**: Tại sao chọn approach này
3. **Discuss performance**: Luôn consider performance
4. **Mention best practices**: Security, accessibility, SEO
5. **Be honest**: Thành thật khi không biết

---

**Chúc bạn phỏng vấn thành công! 🚀**
