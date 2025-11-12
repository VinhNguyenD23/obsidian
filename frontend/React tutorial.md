## 📚 Hướng dẫn ReactJS: Từ Khởi Tạo đến Custom Hooks

Tài liệu này được thiết kế để dẫn dắt bạn qua hành trình học React, tập trung vào các khái niệm cốt lõi theo cách thực hành nhất. Chúng ta sẽ sử dụng **Functional Components** và **Hooks**, là tiêu chuẩn hiện đại của React.

### 📋 Yêu cầu tiên quyết

Trước khi bắt đầu, hãy đảm bảo bạn đã có kiến thức cơ bản về:

- **HTML và CSS:** Cấu trúc và tạo kiểu cho trang web.
    
- **JavaScript (ES6+):** Rất quan trọng. Bạn cần nắm rõ về biến (`let`, `const`), hàm mũi tên (`() => {}`), `map`, `filter`, `reduce`, destructuring (`{...}`), và modules (`import`/`export`).
    
- **Node.js và npm/yarn:** Đã cài đặt trên máy của bạn để quản lý thư viện và chạy dự án.
    

---

### Phần 1: 🚀 Khởi tạo Dự án (Cách Hiện Đại)

Chúng ta sẽ sử dụng **Vite** thay vì Create React App (CRA) vì tốc độ vượt trội của nó.

1. **Mở Terminal** của bạn và chạy lệnh sau:
    
    Bash
    
    ```bash
    npm create vite@latest my-react-app -- --template react
    ```
    
    (Nếu bạn dùng Yarn: `yarn create vite my-react-app --template react`)
    
2. **Di chuyển vào thư mục dự án** và cài đặt các thư viện:
    
    Bash
    
    ```bash
    cd my-react-app
    npm install
    ```
    
3. **Khởi động server phát triển:**
    
    Bash
    
    ```bash
    npm run dev
    ```
    
    Mở trình duyệt và truy cập `http://localhost:5173` (hoặc cổng mà Vite hiển thị). Bạn sẽ thấy trang React đầu tiên của mình!
    

**Cấu trúc thư mục (chỉ quan tâm `src`):**

- `public/`: Chứa các file tĩnh (như `favicon.ico`).
    
- `src/`: Đây là nơi bạn sẽ code 99% thời gian.
    
- `src/main.jsx`: Điểm vào (entry point) của ứng dụng.
    
- `src/App.jsx`: Component gốc của ứng dụng.
    

---

### Phần 2: 🧱 JSX và Components Căn bản

#### 1. JSX là gì?

JSX (JavaScript XML) là một phần mở rộng cú pháp cho JavaScript. Nó cho phép bạn viết code "giống-HTML" ngay bên trong file JavaScript.

JavaScript

```JavaScript
// Đây không phải HTML, đây là JSX!
const element = <h1>Xin chào, thế giới!</h1>;

// Nó được "biên dịch" thành JavaScript thuần:
const element = React.createElement('h1', null, 'Xin chào, thế giới!');
```

**Quy tắc JSX quan trọng:**

- **Chỉ một phần tử gốc:** Mọi component phải trả về một phần tử gốc duy nhất. Bạn có thể dùng `<div>...</div>` hoặc Fragment (`<>...</>`).
    
- **Thuộc tính:** Dùng `className` thay vì `class` (vì `class` là từ khóa của JS) và `htmlFor` thay vì `for`.
    
- **Biểu thức JavaScript:** Sử dụng dấu ngoặc nhọn `{}` để nhúng JavaScript.
    
    JavaScript
    
    ```JavaScript
    const name = "React";
    const element = <h1>Xin chào, {name}!</h1>; // Kết quả: <h1>Xin chào, React!</h1>
    ```
    

#### 2. Functional Components

Trong React hiện đại, component chỉ đơn giản là một hàm JavaScript trả về JSX.

Tạo file `src/components/Welcome.jsx`:

JavaScript

```JavaScript
// Tên component LUÔN viết hoa chữ cái đầu
function Welcome() {
  return <h1>Chào mừng bạn đến với React!</h1>;
}

// Đừng quên export để App.jsx có thể dùng
export default Welcome;
```

Sử dụng component này trong `src/App.jsx`:

JavaScript

```JavaScript
import Welcome from './components/Welcome'; // Import
import './App.css';

function App() {
  return (
    <> {/* Sử dụng Fragment */}
      <Welcome /> {/* Sử dụng component như một thẻ HTML */}
      <Welcome />
    </>
  );
}

export default App;
```

---

### Phần 3: ➡️ Props (Truyền Dữ liệu)

**Props** (viết tắt của properties) là cách bạn truyền dữ liệu từ component cha xuống component con. Props là **read-only** (chỉ đọc).

Hãy cập nhật `Welcome.jsx` để nhận "props":

JavaScript

```JavaScript
// props là một object chứa tất cả dữ liệu được truyền vào
function Welcome(props) {
  // Bạn có thể truy cập props.name
  return <h1>Chào, {props.name}!</h1>;
}

export default Welcome;
```

Và cập nhật `App.jsx` để truyền "props" vào:

JavaScript

```JavaScript
import Welcome from './components/Welcome';
import './App.css';

function App() {
  return (
    <>
      <Welcome name="Alice" /> {/* Truyền prop 'name' */}
      <Welcome name="Bob" />
    </>
  );
}

export default App;
```

**Mẹo (Destructuring):** Bạn nên dùng destructuring để code sạch hơn:

JavaScript

```JavaScript
// Thay vì dùng (props)
function Welcome({ name, age }) {
  // Bạn có thể dùng trực tiếp 'name' và 'age'
  return <h1>Chào, {name}! Bạn {age} tuổi.</h1>;
}
```

---

### Phần 4: 🔄 State và Sự kiện (useState)

**State** là dữ liệu _nội tại_ của một component, và nó có thể thay đổi. Khi state thay đổi, React sẽ tự động render lại (re-render) component đó.

Chúng ta sử dụng Hook đầu tiên: `useState`.

#### 1. `useState` Hook

`useState` trả về một mảng gồm 2 phần tử: [giá trị state hiện tại, hàm để cập nhật state].

Hãy tạo một component counter (`src/components/Counter.jsx`):

JavaScript

```JavaScript
import { useState } from 'react'; // 1. Import useState

function Counter() {
  // 2. Khai báo state
  // 0 là giá trị ban đầu
  const [count, setCount] = useState(0);

  // 3. Hàm xử lý sự kiện
  const handleIncrease = () => {
    // 4. Cập nhật state
    setCount(count + 1);
  };

  return (
    <div>
      {/* 5. Hiển thị state */}
      <p>Số lượt click: {count}</p>
      
      {/* 6. Gắn sự kiện (camelCase) */}
      <button onClick={handleIncrease}>Click để tăng</button>
      
      <button onClick={() => setCount(count - 1)}>Click để giảm</button>
    </div>
  );
}

export default Counter;
```

> **Quan trọng:** Đừng bao giờ thay đổi state trực tiếp (ví dụ: `count = count + 1`). Luôn sử dụng hàm setter (`setCount`).

#### 2. Render có điều kiện và Lists

- **Render có điều kiện:**
    
    JavaScript
    
    ```JavaScript
    {isLoggedIn ? <AdminPanel /> : <LoginForm />}
    {unreadMessages.length > 0 && <h2>Bạn có {unreadMessages.length} tin nhắn.</h2>}
    ```
    
- Render Lists (dùng .map()):
    
    Luôn luôn cung cấp một key duy nhất cho mỗi phần tử trong danh sách.
    
    JavaScript
    
    ```JavaScript
    function TodoList({ todos }) {
      return (
        <ul>
          {todos.map((todo) => (
            <li key={todo.id}>{todo.text}</li>
          ))}
        </ul>
      );
    }
    ```
    

---

### Phần 5: ⚡ Tác vụ Phụ (useEffect)

Làm thế nào để fetch dữ liệu từ API khi component được render? Hoặc làm thế nào để dọn dẹp (cleanup) khi component bị gỡ bỏ?

Chúng ta dùng `useEffect`. Hook này dùng để xử lý các "tác vụ phụ" (side effects).

**Cú pháp:** `useEffect(callback, dependencyArray)`

1. `callback`: Hàm sẽ được chạy.
    
2. `dependencyArray` (Mảng phụ thuộc): Quyết định _khi nào_ `callback` được chạy.
    

**Các trường hợp của mảng phụ thuộc:**

- **`[]` (Mảng rỗng):** Chỉ chạy 1 lần duy nhất khi component được mount (gắn vào).
    
    - _Trường hợp sử dụng:_ Fetch dữ liệu API lần đầu.
        
    
    JavaScript
    
    ```JavaScript
    useEffect(() => {
      console.log("Component đã được mount!");
      // fetch('https://api.example.com/data')
    }, []); // <-- Mảng rỗng
    ```
    
- **`[state, prop]` (Có giá trị):** Chạy lần đầu khi mount, và chạy lại _mỗi khi_ giá trị trong mảng (ví dụ: `state`, `prop`) thay đổi.
    
    - _Trường hợp sử dụng:_ Fetch dữ liệu mới khi `userId` thay đổi.
        
    
    JavaScript
    
    ```JavaScript
    useEffect(() => {
      console.log(`UserID đã thay đổi thành: ${userId}`);
      // fetch(`https://api.example.com/users/${userId}`)
    }, [userId]); // <-- Phụ thuộc vào userId
    ```
    
- **(Không có mảng):** Chạy _mỗi khi_ component re-render (dù là do state hay prop nào thay đổi). **Cẩn thận,** dễ gây lặp vô hạn!
    

Hàm Cleanup:

useEffect có thể trả về một hàm. Hàm này sẽ được gọi khi component bị unmount (gỡ bỏ) hoặc trước khi effect chạy lại.

JavaScript

```JavaScript
useEffect(() => {
  const timerId = setInterval(() => {
    console.log('Tick');
  }, 1000);

  // Hàm cleanup
  return () => {
    clearInterval(timerId); // Dọn dẹp interval khi component unmount
  };
}, []);
```

---

### Phần 6: 📦 Các Hooks Hữu ích khác

- **`useContext`**: Dùng để chia sẻ state xuyên suốt cây component mà không cần "truyền props" (prop drilling) qua nhiều cấp. Rất hữu ích cho state toàn cục (global state) như thông tin user, theme (sáng/tối).
    
- **`useReducer`**: Một giải pháp thay thế cho `useState` khi bạn có logic state phức tạp, nhiều hành động cập nhật, hoặc state tiếp theo phụ thuộc vào state trước đó.
    
- **`useRef`**: Dùng để truy cập trực tiếp một phần tử DOM (ví dụ: để focus vào input) hoặc để lưu một giá trị mà không làm component re-render khi nó thay đổi.
    
- **`useMemo` / `useCallback`**: Dùng để tối ưu hóa hiệu năng bằng cách "ghi nhớ" (memoize) một giá trị (useMemo) hoặc một hàm (useCallback), ngăn chúng bị tính toán hoặc tạo lại một cách không cần thiết.
    

---

### Phần 7: 🔧 Custom Hooks (Mục tiêu Cuối cùng)

Đây là phần tuyệt vời nhất của Hooks.

> **Custom Hook** là một hàm JavaScript có tên bắt đầu bằng `use`, cho phép bạn **tái sử dụng logic có trạng thái (stateful logic)**.

Nó chỉ đơn giản là một hàm, nhưng bên trong nó có thể gọi các Hooks khác (như `useState`, `useEffect`).

#### Ví dụ 1: `useToggle` (Đơn giản)

Hãy tạo một hook để quản lý logic "bật/tắt" (toggle).

Tạo file `src/hooks/useToggle.js`:

JavaScript

```JavaScript
import { useState } from 'react';

function useToggle(initialValue = false) {
  // 1. Dùng useState bên trong
  const [value, setValue] = useState(initialValue);

  // 2. Tạo logic để tái sử dụng
  const toggle = () => {
    setValue((prevValue) => !prevValue);
  };

  // 3. Trả về state và hàm điều khiển
  return [value, toggle];
}

export default useToggle;
```

**Sử dụng nó trong component:**

JavaScript

```JavaScript
import useToggle from '../hooks/useToggle';

function MyComponent() {
  const [isModalOpen, toggleModal] = useToggle(false); // Dùng như useState!
  const [isMenuOpen, toggleMenu] = useToggle(true);

  return (
    <div>
      <button onClick={toggleModal}>
        {isModalOpen ? 'Đóng Modal' : 'Mở Modal'}
      </button>
      {isModalOpen && (<div>Nội dung Modal</div>)}
      
      <button onClick={toggleMenu}>
        {isMenuOpen ? 'Đóng Menu' : 'Mở Menu'}
      </button>
      {isMenuOpen && (<ul><li>Menu Item</li></ul>)}
    </div>
  );
}
```

Bạn thấy không? Logic `(prev) => !prev` đã được đóng gói và tái sử dụng ở 2 nơi mà không cần lặp lại code.

#### Ví dụ 2: `useFetch` (Kinh điển)

Đây là custom hook mà hầu hết mọi dự án React đều cần: fetch dữ liệu.

Tạo file `src/hooks/useFetch.js`:

JavaScript

```JavaScript
import { useState, useEffect } from 'react';

function useFetch(url) {
  // Quản lý cả 3 trạng thái của một lời gọi API
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Định nghĩa hàm fetch bên trong effect
    const fetchData = async () => {
      setLoading(true); // Bắt đầu loading
      setError(null);
      
      try {
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const jsonData = await response.json();
        setData(jsonData);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false); // Luôn tắt loading, dù thành công hay thất bại
      }
    };

    fetchData();

    // Chúng ta muốn effect này chạy lại nếu URL thay đổi
  }, [url]); 

  // Trả về một object chứa cả 3 trạng thái
  return { data, loading, error };
}

export default useFetch;
```

**Sử dụng nó trong component:**

JavaScript

```JavaScript
import useFetch from '../hooks/useFetch';

function UserProfile({ userId }) {
  // Chỉ một dòng để gọi API và quản lý state!
  const { data, loading, error } = useFetch(`https://api.example.com/users/${userId}`);

  if (loading) {
    return <div>Đang tải...</div>;
  }

  if (error) {
    return <div>Lỗi: {error.message}</div>;
  }

  // Khi không loading và không có lỗi, 'data' đã sẵn sàng
  return (
    <div>
      <h1>{data.name}</h1>
      <p>Email: {data.email}</p>
    </div>
  );
}
```

---
### Phần 8: 🧭 Thêm React Router (Điều hướng)

#### 1. React Router là gì?

Trong một SPA, bạn không tải lại toàn bộ trang web khi người dùng nhấp vào một liên kết. Thay vào đó, React Router sẽ "đánh chặn" các thay đổi URL và chỉ render các component React tương ứng với URL đó.

#### 2. Cài đặt

Trong terminal của dự án `my-react-app`, chạy lệnh:

Bash

```
npm install react-router-dom
```

#### 3. Thiết lập Cơ bản

Cách thiết lập phổ biến nhất là sử dụng `BrowserRouter`. Chúng ta sẽ cấu hình nó trong file `src/main.jsx`.

Cập nhật file `src/main.jsx`:

JavaScript

```
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom'; // 1. Import
import App from './App.jsx';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter> {/* 2. Bọc toàn bộ <App> bằng BrowserRouter */}
      <App />
    </BrowserRouter>
  </React.StrictMode>
);
```

Điều này cho phép toàn bộ ứng dụng của bạn nhận biết được các thay đổi về URL.

#### 4. Tạo các Trang (Pages)

Bây giờ, hãy tạo một vài component để làm "trang".

Tạo thư mục `src/pages`:

- `src/pages/HomePage.jsx`:
    
    JavaScript
    
    ```JavaScript
    function HomePage() {
      return (
        <div>
          <h1>Trang Chủ</h1>
          <p>Chào mừng bạn đến với trang chủ!</p>
        </div>
      );
    }
    export default HomePage;
    ```
    
- `src/pages/AboutPage.jsx`:
    
    JavaScript
    
    ```JavaScript
    function AboutPage() {
      return (
        <div>
          <h1>Trang Giới Thiệu</h1>
          <p>Đây là trang giới thiệu về chúng tôi.</p>
        </div>
      );
    }
    export default AboutPage;
    ```
    
- `src/pages/NotFoundPage.jsx` (Rất quan trọng):
    
    JavaScript
    
    ```JavaScript
    function NotFoundPage() {
      return <h1>404 - Không tìm thấy trang</h1>;
    }
    export default NotFoundPage;
    ```
    

#### 5. Định nghĩa các Tuyến đường (Routes)

Bây giờ, chúng ta sẽ cho React biết component nào sẽ hiển thị với URL nào. Chúng ta làm điều này trong `src/App.jsx`.

Cập nhật `src/App.jsx`:

JavaScript

```JavaScript
import { Routes, Route, Link } from 'react-router-dom'; // 1. Import
import HomePage from './pages/HomePage'; // 2. Import các trang
import AboutPage from './pages/AboutPage';
import NotFoundPage from './pages/NotFoundPage';
import './App.css';

function App() {
  return (
    <div>
      {/* 3. Tạo thanh điều hướng (Navigation) */}
      <nav>
        <ul>
          <li>
            {/* Dùng <Link> thay vì <a> để không tải lại trang */}
            <Link to="/">Trang Chủ</Link>
          </li>
          <li>
            <Link to="/about">Giới Thiệu</Link>
          </li>
        </ul>
      </nav>

      <hr />

      {/* 4. Nơi nội dung trang sẽ được render */}
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        
        {/* Route "bắt tất cả" cho trang 404 */}
        <Route path="*" element={<NotFoundPage />} /> 
      </Routes>
    </div>
  );
}

export default App;
```

**Giải thích:**

- **`<Link to="...">`**: Đây là cách bạn tạo liên kết. Nó giống như thẻ `<a>`, nhưng nó ngăn trình duyệt tải lại trang và chỉ thay đổi URL, cho phép React Router xử lý phần còn lại.
    
- **`<Routes>`**: Bọc tất cả các định nghĩa tuyến đường của bạn.
    
- **`<Route path="..." element={...} />`**: Đây là phần cốt lõi.
    
    - `path="/"`: Khi URL là `/` (trang chủ), render `element` là `<HomePage />`.
        
    - `path="/about"`: Khi URL là `/about`, render `element` là `<AboutPage />`.
        
    - `path="*"`: Dấu `*` hoạt động như một "wildcard". Nếu không có `path` nào ở trên khớp, nó sẽ khớp với `path="*"` và render `<NotFoundPage />`.
        

Bây giờ, hãy chạy `npm run dev` và thử nhấp qua lại giữa "Trang Chủ" và "Giới Thiệu". Bạn sẽ thấy nội dung thay đổi ngay lập tức mà không cần tải lại trang!

---

### 💡 Mẹo Nâng cao: Layout Chung (Nested Routes)

Trong thực tế, bạn thường muốn thanh điều hướng (navbar) và chân trang (footer) xuất hiện trên _mọi_ trang. Chúng ta có thể dùng **Nested Routes** (Tuyến đường lồng nhau).

1. Tạo Layout Component:
    
    Tạo src/components/Layout.jsx:
    
    JavaScript
    
    ```JavaScript
    import { Outlet, Link } from 'react-router-dom';
    
    function Layout() {
      return (
        <div>
          {/* 1. Navbar cố định */}
          <nav>
            <ul>
              <li><Link to="/">Trang Chủ</Link></li>
              <li><Link to="/about">Giới Thiệu</Link></li>
            </ul>
          </nav>
    
          <hr />
    
          {/* 2. Đây là nơi các trang con (HomePage, AboutPage) sẽ được render */}
          <main>
            <Outlet /> 
          </main>
    
          {/* 3. Footer cố định (ví dụ) */}
          <footer>
            <p>© 2025 Bản quyền thuộc về tôi</p>
          </footer>
        </div>
      );
    }
    
    export default Layout;
    ```
    
    **`<Outlet />`** là một component đặc biệt của React Router, nó đóng vai trò là "chỗ giữ chỗ" cho các route con.
    
2. Cập nhật App.jsx:
    
    Bây giờ chúng ta lồng các route kia vào bên trong một route Layout.
    
    JavaScript
    
    ```JavaScript
    import { Routes, Route } from 'react-router-dom';
    import Layout from './components/Layout'; // 1. Import Layout
    import HomePage from './pages/HomePage';
    import AboutPage from './pages/AboutPage';
    import NotFoundPage from './pages/NotFoundPage';
    import './App.css';
    
    function App() {
      return (
        // Chỉ cần <Routes> ở đây
        <Routes>
          {/* 2. Tạo một route cha sử dụng Layout */}
          <Route path="/" element={<Layout />}>
            {/* 3. Các route con sẽ render vào <Outlet> của Layout */}
    
            {/* path="/" + index=true nghĩa là đây là component mặc định */}
            <Route index element={<HomePage />} /> 
    
            <Route path="about" element={<AboutPage />} />
    
            {/* Trang 404 cũng nên nằm trong Layout */}
            <Route path="*" element={<NotFoundPage />} /> 
          </Route>
    
          {/* (Nếu bạn có các trang không dùng Layout, ví dụ trang Login, 
               bạn có thể định nghĩa chúng bên ngoài) */}
          {/* <Route path="/login" element={<LoginPage />} /> */}
        </Routes>
      );
    }
    
    export default App;
    ```
    

**Giải thích:**

- `path="/" element={<Layout />}`: Route cha này nói rằng bất cứ URL nào bắt đầu bằng `/` (về cơ bản là mọi URL) sẽ sử dụng `Layout`.
    
- `<Route index ... />`: Thuộc tính `index` thay cho `path="/"`. Nó cho biết "Đây là component sẽ render khi URL khớp _chính xác_ với route cha (`/`)".
    
- `<Route path="about" ... />`: Lưu ý không có `/` ở trước. Path này được nối vào path của cha, thành `/about`.
    