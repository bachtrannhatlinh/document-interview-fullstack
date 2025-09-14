# STATE MANAGEMENT - REDUX & ALTERNATIVES

## 1. Redux Core Concepts

### What is Redux?
- **Predictable state container** cho JavaScript apps
- **Single source of truth** - một store duy nhất
- **State is read-only** - chỉ thay đổi qua actions
- **Pure functions** - reducers không có side effects

### Three Principles
1. **Single source of truth**: App state nằm trong một object tree trong một store
2. **State is read-only**: Chỉ có thể thay đổi state bằng cách dispatch actions
3. **Changes made with pure functions**: Reducers là pure functions nhận previous state và action, return new state

### Core Building Blocks

#### Actions
```javascript
// Action types
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';
const SET_FILTER = 'SET_FILTER';

// Action creators
const addTodo = (text) => ({
  type: ADD_TODO,
  payload: {
    id: Date.now(),
    text,
    completed: false
  }
});

const toggleTodo = (id) => ({
  type: TOGGLE_TODO,
  payload: { id }
});

// Async action creator with Redux Thunk
const fetchTodos = () => {
  return async (dispatch, getState) => {
    dispatch({ type: 'FETCH_TODOS_START' });
    
    try {
      const response = await api.getTodos();
      dispatch({
        type: 'FETCH_TODOS_SUCCESS',
        payload: response.data
      });
    } catch (error) {
      dispatch({
        type: 'FETCH_TODOS_ERROR',
        payload: error.message
      });
    }
  };
};
```

#### Reducers
```javascript
// Individual reducers
const todosReducer = (state = [], action) => {
  switch (action.type) {
    case ADD_TODO:
      return [...state, action.payload];
      
    case TOGGLE_TODO:
      return state.map(todo =>
        todo.id === action.payload.id
          ? { ...todo, completed: !todo.completed }
          : todo
      );
      
    case 'FETCH_TODOS_SUCCESS':
      return action.payload;
      
    default:
      return state;
  }
};

const filterReducer = (state = 'SHOW_ALL', action) => {
  switch (action.type) {
    case SET_FILTER:
      return action.payload;
    default:
      return state;
  }
};

// Root reducer
import { combineReducers } from 'redux';

const rootReducer = combineReducers({
  todos: todosReducer,
  filter: filterReducer,
  user: userReducer,
  ui: uiReducer
});
```

#### Store
```javascript
import { createStore, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';

// Create store with middleware
const store = createStore(
  rootReducer,
  compose(
    applyMiddleware(thunk),
    window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
  )
);

// Dispatch actions
store.dispatch(addTodo('Learn Redux'));
store.dispatch(fetchTodos());

// Subscribe to changes
const unsubscribe = store.subscribe(() => {
  console.log('State changed:', store.getState());
});

// Get current state
const currentState = store.getState();
```

### React Integration
```jsx
import { Provider, connect, useSelector, useDispatch } from 'react-redux';

// Provider wrapping
function App() {
  return (
    <Provider store={store}>
      <TodoApp />
    </Provider>
  );
}

// Using hooks (modern approach)
function TodoList() {
  const todos = useSelector(state => state.todos);
  const filter = useSelector(state => state.filter);
  const dispatch = useDispatch();
  
  const filteredTodos = useMemo(() => {
    switch (filter) {
      case 'SHOW_COMPLETED':
        return todos.filter(todo => todo.completed);
      case 'SHOW_ACTIVE':
        return todos.filter(todo => !todo.completed);
      default:
        return todos;
    }
  }, [todos, filter]);
  
  const handleAddTodo = (text) => {
    dispatch(addTodo(text));
  };
  
  const handleToggleTodo = (id) => {
    dispatch(toggleTodo(id));
  };
  
  return (
    <div>
      <AddTodo onAdd={handleAddTodo} />
      {filteredTodos.map(todo => (
        <Todo
          key={todo.id}
          todo={todo}
          onToggle={() => handleToggleTodo(todo.id)}
        />
      ))}
    </div>
  );
}

// Using connect (legacy approach)
const mapStateToProps = (state) => ({
  todos: state.todos,
  filter: state.filter
});

const mapDispatchToProps = {
  addTodo,
  toggleTodo
};

export default connect(mapStateToProps, mapDispatchToProps)(TodoList);
```

## 2. Redux Middleware

### What is Middleware?
- Sits between dispatching an action và reducer nhận action
- Có thể **intercept, modify, delay, hoặc stop** actions
- Chain of responsibility pattern

### Redux Thunk
```javascript
// Redux Thunk allows action creators to return functions
const fetchUser = (userId) => {
  return async (dispatch, getState) => {
    dispatch({ type: 'FETCH_USER_START' });
    
    try {
      const response = await api.getUser(userId);
      dispatch({
        type: 'FETCH_USER_SUCCESS',
        payload: response.data
      });
    } catch (error) {
      dispatch({
        type: 'FETCH_USER_ERROR',
        payload: error.message
      });
    }
  };
};

// Conditional dispatching
const incrementIfOdd = () => (dispatch, getState) => {
  const { counter } = getState();
  
  if (counter % 2 !== 0) {
    dispatch({ type: 'INCREMENT' });
  }
};
```

### Redux Saga
```javascript
import { call, put, takeEvery, fork, select } from 'redux-saga/effects';

// Worker saga
function* fetchUserSaga(action) {
  try {
    yield put({ type: 'FETCH_USER_START' });
    
    const response = yield call(api.getUser, action.payload.userId);
    
    yield put({
      type: 'FETCH_USER_SUCCESS',
      payload: response.data
    });
  } catch (error) {
    yield put({
      type: 'FETCH_USER_ERROR',
      payload: error.message
    });
  }
}

// Watcher saga
function* watchFetchUser() {
  yield takeEvery('FETCH_USER_REQUEST', fetchUserSaga);
}

// Root saga
function* rootSaga() {
  yield fork(watchFetchUser);
  yield fork(watchOtherActions);
}

// Advanced saga patterns
function* handleUserFlow() {
  while (true) {
    const { username, password } = yield take('LOGIN_REQUEST');
    
    try {
      const user = yield call(authorize, username, password);
      yield put({ type: 'LOGIN_SUCCESS', user });
      
      // Wait for logout
      yield take('LOGOUT');
      yield call(clearSession);
      
    } catch (error) {
      yield put({ type: 'LOGIN_FAILURE', error });
    }
  }
}
```

### Custom Middleware
```javascript
// Logger middleware
const loggerMiddleware = (store) => (next) => (action) => {
  console.log('Dispatching:', action);
  const result = next(action);
  console.log('Next state:', store.getState());
  return result;
};

// Crash reporter middleware
const crashReporter = (store) => (next) => (action) => {
  try {
    return next(action);
  } catch (err) {
    console.error('Caught an exception!', err);
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    });
    throw err;
  }
};

// Apply middleware
const store = createStore(
  rootReducer,
  applyMiddleware(loggerMiddleware, crashReporter, thunk)
);
```

## 3. Redux Toolkit (RTK)

### Why Redux Toolkit?
- **Reduces boilerplate** code significantly
- **Built-in best practices** (Immer, Redux Thunk)
- **DevTools integration** by default
- **Simplified store setup**

### createSlice
```javascript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk
export const fetchTodos = createAsyncThunk(
  'todos/fetchTodos',
  async (_, { rejectWithValue }) => {
    try {
      const response = await api.getTodos();
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

// Slice
const todosSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    loading: false,
    error: null
  },
  reducers: {
    addTodo: (state, action) => {
      // Immer allows "mutating" logic
      state.items.push({
        id: Date.now(),
        text: action.payload,
        completed: false
      });
    },
    toggleTodo: (state, action) => {
      const todo = state.items.find(todo => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    removeTodo: (state, action) => {
      state.items = state.items.filter(todo => todo.id !== action.payload);
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export const { addTodo, toggleTodo, removeTodo } = todosSlice.actions;
export default todosSlice.reducer;
```

### configureStore
```javascript
import { configureStore } from '@reduxjs/toolkit';
import todosReducer from './features/todos/todosSlice';
import userReducer from './features/user/userSlice';

export const store = configureStore({
  reducer: {
    todos: todosReducer,
    user: userReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST']
      }
    }).concat(customMiddleware),
  devTools: process.env.NODE_ENV !== 'production'
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### RTK Query
```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

// API slice
export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: '/api/',
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    }
  }),
  tagTypes: ['Todo', 'User'],
  endpoints: (builder) => ({
    getTodos: builder.query<Todo[], void>({
      query: () => 'todos',
      providesTags: ['Todo']
    }),
    getTodo: builder.query<Todo, number>({
      query: (id) => `todos/${id}`,
      providesTags: (result, error, id) => [{ type: 'Todo', id }]
    }),
    addTodo: builder.mutation<Todo, Partial<Todo>>({
      query: (newTodo) => ({
        url: 'todos',
        method: 'POST',
        body: newTodo
      }),
      invalidatesTags: ['Todo']
    }),
    updateTodo: builder.mutation<Todo, Partial<Todo> & Pick<Todo, 'id'>>({
      query: ({ id, ...patch }) => ({
        url: `todos/${id}`,
        method: 'PATCH',
        body: patch
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Todo', id }]
    }),
    deleteTodo: builder.mutation<{ success: boolean; id: number }, number>({
      query: (id) => ({
        url: `todos/${id}`,
        method: 'DELETE'
      }),
      invalidatesTags: (result, error, id) => [{ type: 'Todo', id }]
    })
  })
});

export const {
  useGetTodosQuery,
  useGetTodoQuery,
  useAddTodoMutation,
  useUpdateTodoMutation,
  useDeleteTodoMutation
} = api;
```

### RTK Query in Components
```jsx
import { useGetTodosQuery, useAddTodoMutation } from './api';

function TodoList() {
  const {
    data: todos,
    error,
    isLoading,
    isFetching,
    refetch
  } = useGetTodosQuery();
  
  const [addTodo, {
    isLoading: isAdding,
    error: addError
  }] = useAddTodoMutation();
  
  const handleAddTodo = async (text: string) => {
    try {
      await addTodo({ text }).unwrap();
    } catch (error) {
      console.error('Failed to add todo:', error);
    }
  };
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      <button onClick={() => refetch()}>Refresh</button>
      {todos?.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </div>
  );
}
```

## 4. State Management Alternatives

### Zustand
```javascript
import { create } from 'zustand';
import { subscribeWithSelector } from 'zustand/middleware';

// Basic store
const useBearStore = create((set) => ({
  bears: 0,
  increase: () => set((state) => ({ bears: state.bears + 1 })),
  decrease: () => set((state) => ({ bears: state.bears - 1 })),
  reset: () => set({ bears: 0 })
}));

// With async actions
const useUserStore = create((set, get) => ({
  user: null,
  loading: false,
  
  fetchUser: async (id) => {
    set({ loading: true });
    try {
      const user = await api.getUser(id);
      set({ user, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
  
  updateUser: (updates) => set((state) => ({
    user: { ...state.user, ...updates }
  }))
}));

// Using in component
function BearCounter() {
  const bears = useBearStore((state) => state.bears);
  const increase = useBearStore((state) => state.increase);
  
  return (
    <div>
      <h1>{bears} around here...</h1>
      <button onClick={increase}>Add bear</button>
    </div>
  );
}

// Slices pattern
const createBearSlice = (set) => ({
  bears: 0,
  addBear: () => set((state) => ({ bears: state.bears + 1 })),
  eatFish: () => set((state) => ({ fishes: state.fishes - 1 }))
});

const createFishSlice = (set) => ({
  fishes: 0,
  addFish: () => set((state) => ({ fishes: state.fishes + 1 }))
});

const useStore = create((...args) => ({
  ...createBearSlice(...args),
  ...createFishSlice(...args)
}));
```

### Jotai (Atomic approach)
```javascript
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// Primitive atoms
const countAtom = atom(0);
const nameAtom = atom('John');

// Derived atoms
const doubleCountAtom = atom((get) => get(countAtom) * 2);

// Write-only atom
const decrementCountAtom = atom(
  null, // no read function
  (get, set) => set(countAtom, get(countAtom) - 1)
);

// Async atom
const fetchUserAtom = atom(async (get) => {
  const userId = get(userIdAtom);
  const response = await fetch(`/users/${userId}`);
  return response.json();
});

// Using atoms
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubleCount = useAtomValue(doubleCountAtom);
  const decrement = useSetAtom(decrementCountAtom);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Double: {doubleCount}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
      <button onClick={decrement}>-1</button>
    </div>
  );
}

// Atom families
const todosAtom = atom([]);
const todoAtomFamily = atomFamily((id) =>
  atom(
    (get) => get(todosAtom).find(todo => todo.id === id),
    (get, set, newTodo) => {
      set(todosAtom, (todos) =>
        todos.map(todo => todo.id === id ? { ...todo, ...newTodo } : todo)
      );
    }
  )
);
```

### Context API + useReducer
```jsx
import React, { createContext, useContext, useReducer } from 'react';

// Actions
const actionTypes = {
  ADD_TODO: 'ADD_TODO',
  TOGGLE_TODO: 'TOGGLE_TODO',
  SET_FILTER: 'SET_FILTER'
};

// Reducer
const todoReducer = (state, action) => {
  switch (action.type) {
    case actionTypes.ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, {
          id: Date.now(),
          text: action.payload,
          completed: false
        }]
      };
      
    case actionTypes.TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
      
    case actionTypes.SET_FILTER:
      return {
        ...state,
        filter: action.payload
      };
      
    default:
      return state;
  }
};

// Context
const TodoContext = createContext();

// Provider component
export const TodoProvider = ({ children }) => {
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    filter: 'all'
  });
  
  const actions = {
    addTodo: (text) => dispatch({
      type: actionTypes.ADD_TODO,
      payload: text
    }),
    toggleTodo: (id) => dispatch({
      type: actionTypes.TOGGLE_TODO,
      payload: id
    }),
    setFilter: (filter) => dispatch({
      type: actionTypes.SET_FILTER,
      payload: filter
    })
  };
  
  return (
    <TodoContext.Provider value={{ state, actions }}>
      {children}
    </TodoContext.Provider>
  );
};

// Hook
export const useTodos = () => {
  const context = useContext(TodoContext);
  if (!context) {
    throw new Error('useTodos must be used within TodoProvider');
  }
  return context;
};

// Usage
function TodoList() {
  const { state, actions } = useTodos();
  
  return (
    <div>
      {state.todos.map(todo => (
        <div key={todo.id} onClick={() => actions.toggleTodo(todo.id)}>
          {todo.text}
        </div>
      ))}
    </div>
  );
}
```

## 5. Common Interview Questions

### 1. Redux vs Context API - Khi nào dùng gì?

**Redux:**
- **Pros**: DevTools, middleware, time-travel debugging, predictable updates
- **Cons**: Boilerplate code, learning curve
- **Khi dùng**: Complex state logic, need debugging tools, many components cần access state

**Context API:**
- **Pros**: Built-in React, simple setup, good for theme/auth
- **Cons**: No devtools, can cause unnecessary re-renders, no middleware
- **Khi dùng**: Simple state sharing, infrequent updates

```jsx
// Context API performance issue
const AppContext = createContext();

// ❌ This will re-render all consumers when any part of value changes
const AppProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  const value = { user, setUser, theme, setTheme }; // New object every render
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
};

// ✅ Better approach - split contexts or memoize value
const AppProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  const value = useMemo(() => ({
    user, setUser, theme, setTheme
  }), [user, theme]);
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
};
```

### 2. Redux middleware hoạt động như thế nào?

**Middleware signature:**
```javascript
const middleware = (store) => (next) => (action) => {
  // Before action reaches reducer
  console.log('Before:', store.getState());
  
  // Call next middleware in chain
  const result = next(action);
  
  // After reducer has processed action
  console.log('After:', store.getState());
  
  return result;
};
```

**Middleware chain execution:**
```javascript
// Multiple middleware
const store = createStore(
  reducer,
  applyMiddleware(middleware1, middleware2, middleware3)
);

// Execution flow:
// dispatch(action) -> middleware1 -> middleware2 -> middleware3 -> reducer -> store
```

### 3. Tại sao cần immutability trong Redux?

**Lý do:**
- **Predictable updates**: Mỗi state change tạo new object
- **Time-travel debugging**: Có thể quay lại previous states
- **Performance optimization**: Reference equality checks
- **Prevent bugs**: Tránh accidental mutations

```javascript
// ❌ Wrong - mutating state
const todosReducer = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      state.push(action.payload); // Mutation!
      return state;
      
    case 'TOGGLE_TODO':
      const todo = state.find(t => t.id === action.payload);
      todo.completed = !todo.completed; // Mutation!
      return state;
      
    default:
      return state;
  }
};

// ✅ Correct - immutable updates
const todosReducer = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload]; // New array
      
    case 'TOGGLE_TODO':
      return state.map(todo => // New array with new objects
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
      
    default:
      return state;
  }
};
```

### 4. Redux Toolkit vs Redux thuần - Ưu nhược điểm?

**Redux thuần:**
```javascript
// More boilerplate
const ADD_TODO = 'ADD_TODO';

const addTodo = (text) => ({
  type: ADD_TODO,
  payload: { id: Date.now(), text, completed: false }
});

const todosReducer = (state = [], action) => {
  switch (action.type) {
    case ADD_TODO:
      return [...state, action.payload];
    default:
      return state;
  }
};

const store = createStore(
  combineReducers({ todos: todosReducer }),
  applyMiddleware(thunk)
);
```

**Redux Toolkit:**
```javascript
// Less boilerplate, built-in best practices
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      state.push(action.payload); // Immer handles immutability
    }
  }
});

const store = configureStore({
  reducer: { todos: todosSlice.reducer }
}); // Thunk included by default
```

### 5. Server state vs Client state - Cách quản lý?

**Client State:**
- UI state (modals, forms, themes)
- User preferences
- Navigation state

**Server State:**
- Data from API
- Cache management
- Loading/error states
- Synchronization

```javascript
// ❌ Mixing server and client state
const appSlice = createSlice({
  name: 'app',
  initialState: {
    // Client state
    theme: 'light',
    sidebarOpen: false,
    
    // Server state (should be separated)
    users: [],
    posts: [],
    loading: false
  }
});

// ✅ Separate concerns
// Client state
const uiSlice = createSlice({
  name: 'ui',
  initialState: {
    theme: 'light',
    sidebarOpen: false
  }
});

// Server state with RTK Query
const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: (builder) => ({
    getUsers: builder.query({ query: () => 'users' }),
    getPosts: builder.query({ query: () => 'posts' })
  })
});
```

## 6. Best Practices & Patterns

### Normalized State Structure
```javascript
// ❌ Nested/denormalized state
const state = {
  posts: [
    {
      id: 1,
      title: 'Post 1',
      author: { id: 1, name: 'John' },
      comments: [
        { id: 1, text: 'Comment 1', author: { id: 2, name: 'Jane' } }
      ]
    }
  ]
};

// ✅ Normalized state
const state = {
  posts: {
    byId: {
      1: { id: 1, title: 'Post 1', authorId: 1, commentIds: [1] }
    },
    allIds: [1]
  },
  users: {
    byId: {
      1: { id: 1, name: 'John' },
      2: { id: 2, name: 'Jane' }
    },
    allIds: [1, 2]
  },
  comments: {
    byId: {
      1: { id: 1, text: 'Comment 1', authorId: 2, postId: 1 }
    },
    allIds: [1]
  }
};
```

### Selectors with Memoization
```javascript
import { createSelector } from '@reduxjs/toolkit';

// Basic selector
const getTodos = (state) => state.todos;
const getFilter = (state) => state.filter;

// Memoized selector
const getFilteredTodos = createSelector(
  [getTodos, getFilter],
  (todos, filter) => {
    switch (filter) {
      case 'COMPLETED':
        return todos.filter(todo => todo.completed);
      case 'ACTIVE':
        return todos.filter(todo => !todo.completed);
      default:
        return todos;
    }
  }
);

// Selector with parameters
const getTodoById = createSelector(
  [getTodos, (state, id) => id],
  (todos, id) => todos.find(todo => todo.id === id)
);

// Usage in component
function TodoList() {
  const filteredTodos = useSelector(getFilteredTodos);
  const todo = useSelector(state => getTodoById(state, todoId));
  
  // ...
}
```

### Error Handling Patterns
```javascript
// Global error handling
const errorMiddleware = (store) => (next) => (action) => {
  if (action.error) {
    console.error('Redux Error:', action.error);
    // Send to error reporting service
    errorReporter.captureException(action.error);
  }
  return next(action);
};

// Async error handling with RTK
const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: false,
    error: null
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});
```

### Testing Redux Code
```javascript
import { configureStore } from '@reduxjs/toolkit';
import { render } from '@testing-library/react';
import { Provider } from 'react-redux';

// Test store factory
const createTestStore = (initialState = {}) => {
  return configureStore({
    reducer: rootReducer,
    preloadedState: initialState
  });
};

// Test utilities
const renderWithRedux = (
  component,
  { initialState, store = createTestStore(initialState) } = {}
) => {
  return {
    ...render(
      <Provider store={store}>
        {component}
      </Provider>
    ),
    store
  };
};

// Testing reducers
describe('todosSlice', () => {
  it('should add todo', () => {
    const initialState = [];
    const action = addTodo('Test todo');
    const result = todosReducer(initialState, action);
    
    expect(result).toHaveLength(1);
    expect(result[0].text).toBe('Test todo');
  });
});

// Testing async thunks
describe('fetchTodos', () => {
  it('should fetch todos successfully', async () => {
    const mockResponse = [{ id: 1, text: 'Todo 1' }];
    api.getTodos.mockResolvedValue({ data: mockResponse });
    
    const store = createTestStore();
    await store.dispatch(fetchTodos());
    
    const state = store.getState();
    expect(state.todos.items).toEqual(mockResponse);
  });
});
```
