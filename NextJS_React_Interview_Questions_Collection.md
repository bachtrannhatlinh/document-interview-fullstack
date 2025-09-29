# NEXT.JS & REACT INTERVIEW QUESTIONS - COLLECTION

## 🚀 NEXT.JS QUESTIONS

### 1. **Development Server với Custom IP**
**Question:** You are developing a Next.js website on a laptop and want to preview how it will look on mobile. To do this, you need to make your Next.js app accessible via the local area network IP address 192.168.1.2. Which Next.js CLI command should you use to achieve this?

**Options:**
- npx next dev --hostname 192.168.1.2 ✅
- npx next dev --H 192.168.1.2
- npx next dev -hostname 192.168.1.2
- npx next dev -h 192.168.1.2

**Answer:** `npx next dev --hostname 192.168.1.2`

**Giải thích:** Để chạy Next.js development server trên một IP cụ thể trong mạng LAN (để truy cập từ mobile), bạn cần sử dụng flag `--hostname` với địa chỉ IP mong muốn.

---

### 2. **Data Fetching cho Blog SEO**
**Question:** You work for a startup trying to reach more users through blog articles that introduce its products to readers. You're developing the blog using Next.js, with content coming from a headless content management system (CMS). Your project manager wants the blog to be easily indexed by search engines and quickly accessed by users so there are no long loading times when moving from one article to another. Which data-fetching method should you use?

**Options:**
- getStaticProps ✅
- getServerSideProps
- getInitialProps
- getStaticPaths

**Answer:** `getStaticProps`

**Giải thích:** 
- **SEO quan trọng** (easily indexed by search engines)
- **Performance cao** (no long loading times)
- **Blog content** từ headless CMS
- **getStaticProps** pre-generate HTML tại build time → SEO xuất sắc và tốc độ load nhanh nhất

---

### 3. **Dynamic Routes cho Product Detail**
**Question:** You work for an ecommerce company that uses Next.js. You have completed the product list page and are developing a product detail page to display the details of each product by product ID. Which of the following file names should you use to create the page?

**Options:**
- id.js
- product.js
- product-id.js
- [product-id].js ✅

**Answer:** `[product-id].js`

**Giải thích:** Trong Next.js, để tạo **dynamic routes**, bạn cần sử dụng **square brackets** `[]` bao quanh tên parameter. `[product-id].js` sẽ tạo dynamic route với parameter `product-id`.

---

### 4. **Dynamic Import cho Code Splitting**
**Question:** Your website has a component called MobileNav, which appears when mobile users scroll. To improve initial loading performance, you plan to use code splitting with dynamic import. Which of the following codes should you use to import MobileNav dynamically?

**Options:**
- const MobileNav = dynamic() => import('../components/MobileNav'))
- const MobileNav = import(() => dynamicImport('../components/MobileNav'))
- const MobileNav = import(() => dynamic('../components/MobileNav'))
- const MobileNav = dynamic(() => import('../components/MobileNav')) ✅

**Answer:** `const MobileNav = dynamic(() => import('../components/MobileNav'))`

**Giải thích:** Đây là cú pháp đúng cho dynamic import với code splitting trong Next.js. Component sẽ chỉ được load khi cần thiết, không trong initial bundle.

---

### 5. **Runtime Config Access**
**Question:** You have a public runtime config defined in the next.config.js file. You need to import this configuration file into your code so you can access the data. What syntax should you use to accomplish this?

**Options:**
- import getConfig from 'next/config'; ✅
- return process.env.iconsFolder;
- import { publicRuntimeConfig } from '../next.config.js'
- import { iconsFolder } from 'next/config';

**Answer:** `import getConfig from 'next/config';`

**Giải thích:** 
```javascript
import getConfig from 'next/config';
const { publicRuntimeConfig } = getConfig();
const iconsFolder = publicRuntimeConfig.iconsFolder;
```

---

### 6. **Next.js Dependencies**
**Question:** You're setting up your company's profile website and plan to create a Next.js project from scratch. Before starting the project, you will need to install the necessary packages for Next.js to run. Which packages should you install?

**Options:**
- [x] create-next-app
- [x] next ✅
- [x] react-dom ✅
- [x] react-next
- [x] react ✅
- [x] next-app

**Answer:** `next`, `react`, `react-dom`

**Giải thích:** Đây là minimum requirements cho bất kỳ Next.js project nào. `create-next-app` là CLI tool, không phải dependency.

---

### 7. **CSS Modules Implementation**
**Question:** You are developing a feature-rich Next.js application and want to ensure that the styles you apply to a specific component do not affect other components globally. To achieve this, you decide to use CSS Modules for component-specific styling. How should you accomplish this?

**Answer:** Create a .css file named after the component and import it directly into the component file using CSS Modules.

**Giải thích:**
1. Tạo file với extension `.module.css`
2. Import: `import styles from './Button.module.css'`
3. Sử dụng: `<button className={styles.primaryButton}>`

---

### 8. **TypeScript Migration**
**Question:** An established technology company has hired you to migrate their website from JavaScript to TypeScript without needing to recreate it from scratch. The current landing page was developed using Next.js and JavaScript. Which steps should you take before running the development server?

**Answer:** Create a tsconfig.json file.

**Giải thích:** Để migrate Next.js từ JS sang TS, bước đầu tiên là tạo `tsconfig.json` file. Next.js sẽ auto-detect TypeScript và show hướng dẫn install dependencies.

---

### 9. **ISR Configuration**
**Question:** You are developing a blog platform using Next.js, where articles are updated frequently based on editorial revisions. To optimize for performance, you decide to use incremental static regeneration (ISR). You need to configure your Next.js application to enable ISR for an individual blog post page.

**Answer:** `return { props: {}, revalidate: 10 }`

**Giải thích:**
```javascript
export async function getStaticProps() {
  return {
    props: { posts },
    revalidate: 10 // Regenerate page every 10 seconds
  };
}
```

---

### 10. **Custom 500 Error Page**
**Question:** You need to handle server-side exceptions in your Next.js application more effectively. To improve the user experience during unexpected failures, you decide to implement a custom error page specifically for HTTP 500 errors.

**Answer:** Create a 500.js file in the pages directory and add your custom error handling UI.

**Giải thích:** Next.js tự động detect và render custom error pages:
- `pages/404.js` → 404 errors
- `pages/500.js` → 500 errors
- `pages/_error.js` → all other errors

---

### 11. **Client-Side Rendering cho Social Media**
**Question:** You are developing a social media website's homepage. You plan to implement client-side rendering to fetch users' feeds.

**Answer:** Fetch in useEffect

**Giải thích:**
```javascript
useEffect(() => {
  async function fetchFeeds() {
    const response = await fetch('/api/user/feeds');
    const data = await response.json();
    setFeeds(data);
  }
  fetchFeeds();
}, []);
```

---

### 12. **Real-time Stock Prices Strategy**
**Question:** You are developing a feature for a Next.js application that displays real-time stock prices. The data updates every second and does not need to be prerendered for SEO purposes. To optimize for performance and user experience, you are considering the most appropriate rendering strategy.

**Answer:** Use client-side rendering to fetch stock prices directly in the browser using JavaScript.

**Giải thích:** Real-time data updates mỗi giây không cần SEO, client-side rendering với WebSocket/polling là tối ưu nhất.

---

### 13. **Error Handling Page**
**Question:** You are developing a content management system for a blog using Next.js. To handle any server errors, you need to create a page that can display the error message.

**Answer:** `_error.js`

**Giải thích:** `pages/_error.js` handle tất cả server và client errors trong Next.js.

---

### 14. **Layout Pattern Usage**
**Question:** You have a website with a layout component specifically designed for blog pages. However, you also want to apply the same layout to the news pages on your website.

**Answer:**
```jsx
<Layout>
  <p>News Page</p>
</Layout>
```

**Giải thích:** Content bên trong Layout component sẽ được pass vào như `children` prop.

---

### 15. **Custom 404 Page Configuration**
**Question:** You are implementing a custom 404 page in your Next.js application that displays whenever a user tries to access a nonexistent route. This custom 404 page should provide a friendly message and a link back to the homepage.

**Answer:** Create a pages/404.js file and define a custom React component.

**Giải thích:** Next.js tự động detect `pages/404.js` và render khi user truy cập undefined routes.

---

### 16. **Dynamic Layout per Page**
**Question:** You have a unique layout for various settings pages in your Next.js ecommerce content management system. Your _app.js manages these layouts using a dynamic layout function that defaults to DashboardLayout unless specified otherwise in page components. You are working on the store settings page, which should use a different layout named SettingsLayout.

**Answer:** Add a getLayout method to StoreSettings that returns the page wrapped in SettingsLayout.

**Giải thích:**
```javascript
StoreSettings.getLayout = function getLayout(page) {
  return <SettingsLayout>{page}</SettingsLayout>;
};
```

---

### 17. **Internationalization Setup**
**Question:** You are enhancing your Next.js application to support multiple languages, aiming to provide localized content based on the user's locale. To achieve this, you want to enable internationalized routing and define supported locales.

**Answer:** Add i18n: { defaultLocale: 'en', locales: ['en', 'es', 'fr'] } to next.config.js.

**Giải thích:** Next.js i18n configuration trong `next.config.js` enable automatic routing và locale detection.

---

### 18. **Middleware Configuration**
**Question:** You are working on a user profile page. You want to run middleware, which will only be implemented on the yoursite.com/profile page to check whether a user is authenticated.

**Answer:**
```javascript
export const config = {
  matcher: '/profile',
}
```

**Giải thích:** Middleware `matcher` property specify routes mà middleware sẽ chạy.

---

## ⚛️ REACT QUESTIONS

### 19. **useEffect Execution Count**
**Question:** Considering the code below, how many times will the 'Hello' message be displayed on the console?

```javascript
const App = (props) => {
  const [counter, setCounter] = useState(0);
  useEffect(
    () => {
      console.log('Hello');
      setCounter(1);
    },
    [props.visible]
  );
  return <div>{counter}</div>;
}
```

**Answer:** 1

**Giải thích:** useEffect chạy 1 lần khi component mount. `setCounter(1)` không trigger useEffect vì dependency là `[props.visible]`, không phải `counter`.

---

### 20. **Promise vs Data Comparison**
**Question:** Which statement describes the code below?

```javascript
const fetchData = () => new Promise(r => setTimeout(() => r(Date.now()), 100));

const MyComponent = () => {
  const [result, setResult] = React.useState();
  const data = fetchData().then(value => setResult(value));
  return (
    <div>
      {result === data.toString() ? (
        <div>hello</div>
      ) : (
        <div>good bye</div>
      )}
    </div>
  );
};
```

**Answer:** A 'good bye' message will be displayed.

**Giải thích:** 
- `data` là Promise object → `data.toString() = "[object Promise]"`
- `result` sau khi resolve là number (Date.now())
- `number === "[object Promise]"` → always false → "good bye"

---

### 21. **Component Hide/Show Logic**
**Question:** Which wrapper will hide its child component for four seconds?

**Answer:**
```javascript
const HiderWrapper = (props) => {
  const [visible, setVisible] = useState(false);
  useEffect(() => {
    setTimeout(() => {
      setVisible(true);
    }, 4000);
  }, []);
  if (visible) return props.children;
  else return null;
};
```

**Giải thích:** 
- Initial: `visible = false` → ẩn children
- Sau 4 giây: `setVisible(true)` → hiển thị children

---

### 22. **Component Unmounting Condition**
**Question:** Considering the code below, when will the MyChild component be unmounted?

```javascript
const MyParent = ({ value }) => {
  return <div>{value === 3 && <MyChild />}</div>;
}
```

**Answer:** When the value property is different from 3.

**Giải thích:** 
- `value === 3`: MyChild mounted
- `value !== 3`: MyChild unmounted (conditional rendering)

---

### 23. **React.memo + useCallback Behavior**
**Question:** Assume that the MyButton component is memoized using React.memo. What will happen when the MyParent component is re-rendered?

```javascript
const MyParent = ({ term }) => {
  const onItemClick = useCallback(event => {
    console.log('You clicked', event.currentTarget);
  }, [term]);
  return (
    <MyButton
      term={term}
      onItemClick={onItemClick}
    />
  );
};
```

**Answer:** The onItemClick function will be updated and the MyButton component will re-render.

**Giải thích:** 
- useCallback memoize function với dependency `[term]`
- Khi `term` thay đổi: `onItemClick` tạo function mới → MyButton re-render (mặc dù có memo)
- React.memo chỉ prevent re-render khi tất cả props giống nhau

---

## 📝 KEY CONCEPTS SUMMARY

### Next.js Core Features:
- **Rendering**: SSG, SSR, ISR, CSR
- **Routing**: File-based, dynamic routes, catch-all routes
- **Performance**: Code splitting, image optimization, caching
- **Configuration**: Dynamic imports, CSS modules, TypeScript
- **Error Handling**: Custom error pages, middleware
- **i18n**: Internationalization, locale routing

### React Patterns:
- **Hooks**: useState, useEffect, useCallback, useMemo
- **Performance**: React.memo, memoization strategies
- **Component Lifecycle**: Mount, unmount, conditional rendering
- **State Management**: Component state, prop drilling
- **Event Handling**: Synthetic events, callback patterns

---

### 24. **React.memo + useCallback Pattern**
**Question:** Assume that the MyButton component is memoized using React.memo. What will happen when the MyParent component is re-rendered?

```javascript
const MyParent = ({ term }) => {
  const onItemClick = useCallback(event => {
    console.log('You clicked', event.currentTarget);
  }, [term]);
  return (
    <MyButton
      term={term}
      onItemClick={onItemClick}
    />
  );
};
```

**Options:**
- The onItemClick function will be updated but MyButton memoization will stay intact
- The onItemClick function will remain the same and the MyButton component will re-render ✅
- The onItemClick function will be updated and the MyButton component will re-render
- The onItemClick function will remain the same and MyButton memoization will stay intact

**Answer:** The onItemClick function will remain the same and MyButton memoization will stay intact

**Giải thích:** Khi MyParent re-render nhưng `term` không thay đổi:
- useCallback với dependency `[term]` sẽ trả về cùng function reference
- MyButton được memoized và nhận same props → không re-render
- Đây là pattern tối ưu để prevent unnecessary re-renders

---

### 25. **React.forwardRef Pattern**
**Question:** What is the correct way to link a ref passed from a parent component to a child component?

**Options:**
- `const MyChild = ({ ref }) => { return <div ref={[ref]} />; };`
- `const MyChild = (props) => { return <div ref={[props.ref]} />; };`
- `const MyChild = (props) => { return <div ref={[MyChild.ref]} />; };`
- `const MyChild = React.forwardRef((props, ref) => { return <div ref={[ref]} />; });` ✅

**Answer:** `const MyChild = React.forwardRef((props, ref) => { return <div ref={[ref]} />; });`

**Giải thích:** 
- `ref` không phải là prop thông thường, không thể access trực tiếp
- `React.forwardRef` cho phép component nhận ref như argument thứ 2
- Syntax trong đáp án có lỗi nhỏ: should be `ref={ref}` not `ref={[ref]}`

---

### 26. **React Context Value**
**Question:** Considering the code below, which value will be displayed after the components are rendered?

```javascript
const Context = createContext('apple');
const MyChild = () => {
  const fruit = useContext(Context);
  return <div>{fruit}</div>;
};

const MyParent = () => {
  return (
    <Context.Provider value={'orange'}>
      <MyChild />
    </Context.Provider>
  );
};
```

**Options:**
- apple
- orange ✅  
- undefined
- { context: 'apple', value: 'orange' }

**Answer:** orange

**Giải thích:**
- Context có default value là 'apple'
- MyParent provide value 'orange' thông qua Provider
- MyChild consume context và nhận value 'orange' (không phải default)

---

### 27. **Async Function trong React Component**
**Question:** What will happen after the component below is rendered?

```javascript
const MyComponent = () => {
  const fetchData = async () => {
    return 'Hello';
  };
  return <div>{fetchData()}</div>;
};
```

**Options:**
- A Hello message is displayed
- An [Object Object] message is displayed ✅
- A 'fetchData()' message is displayed  
- The component will crash with the error: Objects are not valid as a React child

**Answer:** The component will crash with the error: Objects are not valid as a React child

**Giải thích:**
- `fetchData()` return một Promise object (vì là async function)
- React không thể render Promise objects → crash with error
- Cần sử dụng useEffect + state để handle async operations

---

### 28. **Higher-Order Component (HOC)**
**Question:** Considering the code below, which message is displayed after the MyApp component is rendered?

```javascript
const withTitle = (C, title, visible = false) => (p) =>
  visible ? (
    <>
      {title} <C {...p} />
    </>
  ) : (
    <>Test</>
  );

const Titled = withTitle(() => <span>morning</span>, 'Good');
const MyApp = () => <Titled />;
```

**Options:**
- Good morning
- morning  
- Test ✅
- Good

**Answer:** Test

**Giải thích:**
- `withTitle` là HOC với default parameter `visible = false`
- Khi `visible = false` → render `<>Test</>`
- Component `() => <span>morning</span>` và title 'Good' không được render

---

### 29. **Render Props Pattern**
**Question:** Considering the code below, which value will be displayed inside the button when the button is clicked for the first time?

```javascript
const MyComponent = ({ render }) => {
  const [value, setValue] = useState('morning');
  return (
    <div>
      {render({
        onClick: () => setValue(v => v + 'good'),
        msg: value,
      })}
    </div>
  );
};

const Button = ({ msg, onClick }) => {
  return <button onClick={onClick}>{msg}</button>;
};

const MyApp = () => <MyComponent render={Button} />;
```

**Options:**
- undefined
- good morning
- morning good ✅
- morning good

**Answer:** morning good

**Giải thích:**
- Initial: `value = 'morning'` → button displays 'morning'  
- When clicked: `setValue(v => v + 'good')` → `'morning' + 'good' = 'morninggood'`
- Closest option is "morning good" (technical sẽ là 'morninggood' không có space)

---

### 30. **Component Selection Context**
**Question:** Which statement describes the results of the code below?

```javascript
const Context = React.createContext();

const Select = (props) => {
  const [current, setCurrent] = useState('');
  const onClick = useCallback(event => {
    console.log('You clicked', event.currentTarget);
  }, [term]);
  return (
    <Context.Provider value={{ onClick: setCurrent }}>
      <input value={current} />
      {props.children}
    </Context.Provider>
  );
};

Select.Button = ({ value }) => {
  const { onClick } = useContext(Context);
  return <button onClick={() => onClick(value)}>{value}</button>;
};

const MyApp = () => {
  return (
    <Select>
      <Select.Button value={'Option 1'} />
      <Select.Button value={'Option 2'} />
    </Select>
  );
};
```

**Answer:** When a button is clicked, the value of the button is displayed inside the input.

**Giải thích:**
- Select component cung cấp `setCurrent` qua Context
- Select.Button sử dụng context để call `setCurrent(value)` when clicked  
- Input hiển thị `current` state → shows button's value when clicked

---

### 31. **React State Update và Re-render**
**Question:** Considering the code below, which value will be displayed inside the button when the button is clicked for the first time?

```javascript
const MyComponent = ({ render }) => {
  const [value, setValue] = useState('morning');
  return (
    <div>
      {render({
        onClick: () => setValue(v => v + 'good'),
        msg: value,
      })}
    </div>
  );
};
```

**Options:**
- undefined
- good morning  
- morning good
- morning good ✅

**Answer:** morning good

**Giải thích:** 
- Trước khi click: button hiển thị 'morning'
- Sau khi click: `setValue('morning' + 'good')` → 'morninggood'
- Gần nhất với option "morning good"

---

### 32. **useEffect Error Handling**
**Question:** Considering the code below, what will happen when the component MyComponent is mounted?

```javascript
const fetchData = () =>
  new Promise(r => 
    setTimeout(() => {
      r(['one', 'two']);
    }, 1000)
  );

const MyComponent = () => {
  const [result, setResult] = useState();
  useEffect(() => {
    fetchData().then(value => setResult(value));
  }, []);
  return <div>{result.length}</div>;
};
```

**Options:**
- An error: Cannot read property 'length' of undefined ✅
- Only the value 2 will be displayed
- Only the value 0 will be displayed  
- The value 0 will be displayed for one second, then the value 2

**Answer:** An error: Cannot read property 'length' of undefined

**Giải thích:**
- Component render lần đầu: `result = undefined`
- `return <div>{result.length}</div>` → crash vì `undefined.length`  
- useEffect chạy sau render → không kịp để prevent error
- Cần conditional rendering: `{result?.length}` or `{result && result.length}`

---

### 33. **React Performance: Bandwidth Optimization** 
**Question:** Your cloud hosting charges you a lot of money for network traffic. You troubleshoot the issue and notice that very large JavaScript files take up most of the bandwidth. What strategies should you consider for using less bandwidth? (Select all that apply)

**Options:**
- [x] Deploy the package with npm run deploy
- [x] Strip comments and excess whitespace before deploying ✅
- [x] Implement type protections with the TypeScript compiler  
- [x] Serve the files over Apache instead of Nginx
- [x] Compress the file in transit with gzip ✅
- [x] Scan the code with a linter

**Answer:** Strip comments and excess whitespace before deploying + Compress the file in transit with gzip

**Giải thích:**
- **Minification** (strip comments/whitespace): Giảm file size significantly  
- **Gzip compression**: Giảm bandwidth 70-80% cho text files
- Other options không directly impact bandwidth usage

---

## 🛠️ ADVANCED PATTERNS & BEST PRACTICES

### React Performance Optimization:
- **Memoization**: React.memo, useMemo, useCallback
- **Code Splitting**: React.lazy, dynamic imports  
- **Bundle Optimization**: Tree shaking, minification, compression

### Component Communication:
- **Props & Callbacks**: Parent-child communication
- **Context API**: Cross-component state sharing
- **Render Props**: Component logic reuse
- **Higher-Order Components**: Cross-cutting concerns

### Error Handling:
- **Error Boundaries**: Catch JavaScript errors
- **Conditional Rendering**: Handle undefined states
- **Async Error Handling**: Promise rejections, try-catch

### Modern React Patterns:
- **Custom Hooks**: Logic reuse and separation
- **Compound Components**: Flexible component APIs
- **forwardRef**: Ref forwarding to DOM elements

---

---

## 🌐 HTML FUNDAMENTALS

### 34. **Alternative Text for Disabled JavaScript**
**Question:** You can use the \_\_\_\_\_\_\_\_\_\_\_\_ element to display an alternative text for users who have disabled JavaScript.

**Answer:** `<noscript>`

**Giải thích:** Element `<noscript>` được sử dụng để hiển thị nội dung thay thế khi JavaScript bị tắt hoặc không được hỗ trợ trong browser. Nội dung bên trong `<noscript>` chỉ được render khi JavaScript không khả dụng.

```html
<noscript>
  <p>This website requires JavaScript to function properly.</p>
</noscript>
```

---

### 35. **Machine-Parsable Information Tags**
**Question:** Your colleague is building a webpage with HTML. They want to know what type of tag they should use to define machine parsable information that does not display on the webpage itself. What kind of tag do they need?

**Options:**
- meta ✅
- machine
- lang
- HTTP header

**Answer:** `meta`

**Giải thích:** Meta tags được sử dụng để cung cấp thông tin metadata có thể được máy đọc nhưng không hiển thị trên trang web, bao gồm:
- `<meta name="description" content="Page description">`
- `<meta name="keywords" content="keyword1, keyword2">`
- `<meta name="author" content="Author name">`
- `<meta name="viewport" content="width=device-width, initial-scale=1.0">`

---

### 36. **SEO Keywords Configuration**
**Question:** You can set keywords for SEO with the \_\_\_\_\_\_\_\_ tag.

**Answer:** `meta`

**Giải thích:** Keywords cho SEO được set bằng meta tag với attribute `name="keywords"`:

```html
<meta name="keywords" content="HTML, CSS, JavaScript, web development, SEO">
```

**Note:** Meta keywords ít được search engines sử dụng hiện tại, thay vào đó nên focus vào quality content và meta description.

---

### 37. **Security Token Transmission**
**Question:** How does a web developer pass a security token to the server when the user submits a form?

**Options:**
- `<input type="hidden" id="token" name="token" value="3245678">` ✅
- `<input type="secure" id="token" name="token" value="3245678">`
- `<input type="secret" id="token" name="token" value="3245678">`
- `<input type="protected" id="token" name="token" value="3245678">`

**Answer:** `<input type="hidden" id="token" name="token" value="3245678">`

**Giải thích:** 
- Input hidden field là cách chuẩn để truyền security tokens (như CSRF tokens) trong forms
- User không thể thấy hoặc modify hidden inputs 
- Security token sẽ được gửi cùng form data khi submit
- Commonly used cho CSRF protection và authentication

```html
<form method="POST" action="/submit">
  <input type="hidden" name="csrf_token" value="abc123xyz">
  <input type="text" name="username">
  <button type="submit">Submit</button>
</form>
```

---

### 38. **Form Method Attribute**
**Question:** The method attribute specifies how to send form-data. What value can the method attribute take? Choose only one value.

**Answer:** `POST`

**Giải thích:** Method attribute có thể nhận các giá trị:
- **GET**: Data được gửi qua URL parameters (visible, có limit length)
- **POST**: Data được gửi trong request body (hidden, no length limit)

```html
<!-- GET method -->
<form method="GET" action="/search">
  <input type="text" name="q">
</form>

<!-- POST method -->
<form method="POST" action="/login">
  <input type="text" name="username">
  <input type="password" name="password">
</form>
```

**Best Practices:**
- **GET**: Cho search, filtering, data retrieval
- **POST**: Cho form submissions, data modifications, sensitive data

---

*Collected from interview practice questions - Next.js & React fundamentals + Advanced patterns + HTML Fundamentals*
