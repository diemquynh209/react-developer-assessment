## Rules

- Write answers **in your own words**. We read for your thinking process, not for perfect definitions.
- If you're unsure about something, say so — partial understanding is better than a copied answer.

## Section 1 — Read and Fix (Backend)

### Question 1.1 — Find the Bugs

This endpoint should create a new note and return it. Find bugs, explain what goes wrong from the user's perspective, and write the corrected code.

```javascript
app.post('/api/notes', (req, res) => {
  try {
    const note = Note.create({
      title: req.body.title,
      content: req.body.content,
    });
    res.status(200).json(note);
  } catch (error) {
    console.log(error);
  }
});
```

**Your answer format (for each bug):**

- The line with the problem
- What the user experiences because of it
- The fix

**Answer :**

1. Thiếu await tại dòng
```javascript
const note = Note.create({
```
Đây là một thao tác bất đồng bộ. Khi thiếu await res.status(200).json(note); sẽ chạy ngay lập tức khi note đang pending
Client nhận được một object rỗng hoặc cấu trúc Promise.
Fixed:
```javascript
app.post('/api/notes', async (req, res) => {
    try {
        const note = await Note.create({
```
2. Thiếu phản hồi lỗi tại
```javascript
} catch (error) {
    console.log(error);
}
```
Khi xảy ra request sẽ bị vì mới log ra console mà không gửi phản hồi HTTP về cho khiến giao diện bị 
Fixed:
```javascript
} catch (error) {
    console.log(error);
    res.status(500).({error: 'Create Failed'});
}
```

### Question 1.2 — What's Wrong With This Schema?

```javascript
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  age: String,
  createdAt: String,
});
```

List everything you would change and explain **why** for each change. There is no single right answer — we want to see your reasoning and what you prioritize.

**Answer :**

1. Kiểu dữ liệu của age đang là string, như thế sẽ không thể so sánh , tính toán hoặc validate giới hạn tuổi. Nên đổi string thành number
2. Kiểu dữ iệu của createAt để string sẽ khó khăn trong việc sắp xếp, lọc ngày tháng, nên chuyển thành { timestamps: true }
3. Cần ràng buộc lowercase: true và trim: true cho cả name và email  để chuẩn hóa dữ liệu, tránh lỗi phân biệt chữ hoa/thường hoặc khoảng trắng thừa
4. email cần ràng buộc thêm required: true vì email là thông tin bắt buộc để người dùng đăng nhập và xác thực tài khoản và unique: true để tránh trùng lặp tài khoản, validate regex định dạng email hợp lệ.

Mã sau khi sửa:
```javascript
const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      trim: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
      match: /^\S+@\S+\.\S+$/,
    },
    age: {
      type: Number,
    },
  },
  {timestamps: true,}
);
```

### Question 1.3 — This Query Is Slow

The notes collection has 500,000 documents. This endpoint takes 8 seconds to respond:

```javascript
app.get('/api/notes/search', async (req, res) => {
  const notes = await Note.find({});
  const results = notes.filter((n) => n.title.toLowerCase().includes(req.query.q.toLowerCase()));
  res.json(results);
});
```

1. Explain **why** it is slow.
2. Rewrite it to be faster. You don't need perfect syntax — show the approach.

**Answer :**
1. Truy vấn chậm vì
   Note.find({}) tải 500.000 bản ghi qua mạng (Network I/O), gây nghẽn băng thông và tiêu tốn RAM khổng lồ.
   Node.js lặp qua 500.000 phần tử trong bộ nhớ đơn luồng để chạy hàm .toLowerCase().includes(), làm block CPU và tăng độ trễ.
2. Cách tiếp cận:
   Dùng truy vấn trực tiếp trong Note.find(...) thay vì lọc ở Node.js.
   Tạo Text Index trong schema để tìm kiếm với $text
   Thêm Phân trang và iới hạn trường lấy ra (select / lean)
3. Viết lại:
```javascript
app.get('/api/notes/search', async (req, res) => {
    const { q ='', pageb= 1, limit = 20} = req.query;
    const query = q ? {title: {$regex: q, $options: 'i'}} : {};
    const results = await Note.find(query)
      .select('title createdAt')
      .skip((page - 1) * limit)
      .limit(Number(limit))
      .lean();
    res.json({ data: results, page: Number(page) });
});
```

## Section 2 — Read and Fix (Frontend)

### Question 2.1 — Fix This React Component

This component should show a list of notes and let the user delete one. Find bugs and fix each one.

```jsx
function NoteList() {
  const [notes, setNotes] = useState();

  useEffect(async () => {
    const res = await fetch('/api/notes');
    const data = await res.json();
    setNotes(data);
  }, []);

  function handleDelete(id) {
    fetch(`/api/notes/${id}`, { method: 'DELETE' });
    setNotes(notes.filter((n) => n.id !== id));
  }

  return (
    <ul>
      {notes.map((note) => (
        <li>
          {note.title}
          <button onClick={handleDelete(note._id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

**Your answer format (for each problem):**

- Quote the line with the problem
- Explain what goes wrong
- Write the fix

**Answer :**

1. Dòng có vấn đề: const [notes, setNotes] = useState();
Khởi tạo không truyền giá trị mặc định khiến notes ban đầu là undefined. Khi component render lần đầu, gọi notes.map() sẽ gây crash ứng dụng
Sửa: const [notes, setNotes] = useState([]);

3. Dòng có vấn đề: useEffect(async () => { ... }, []);
Callback của useEffect không được là một hàm async vì hàm async luôn trả về một Promise, trong khi useEffect chỉ chấp nhận trả về undefined hoặc cleanup function
Sửa:
```javascript
useEffect(() => {
  const fetchNotes = async () => {
    const res = await fetch('/api/notes');
    const data = await res.json();
    setNotes(data);
  };
  fetchNotes();
}, []);
```

3.Dòng có vấn đề: <button onClick={handleDelete(note._id)}>Delete</button>
handleDelete(note._id) sẽ kích hoạt hàm ngay khi render thay vì đợi người dùng click dẫn đến việc gọi API xóa liên tục và cập nhật state gây lỗi re-render vô tận.
Sửa: <button onClick={() => handleDelete(note._id)}>Delete</button>

4.Dòng có vấn đề: setNotes(notes.filter((n) => n.id !== id));
Bên dưới giao diện sử dụng trường _id (note._id), nhưng hàm lọc lại so sánh với n.id (undefined !== id luôn trả về true), khiến danh sách trên UI không bao giờ được cập nhật đúng.
Sửa: setNotes((prevNotes) => prevNotes.filter((n) => n._id !== id));

## Section 3 — Explain In Your Own Words

### Question 3.1 — The CORS Error

You are building a React + Express app. The React dev server runs on `localhost:3000` and Express runs on `localhost:5000`. You try to fetch data from Express and get this error in the browser console:

> Access to fetch at 'http://localhost:5000/api/notes' from origin 'http://localhost:3000' has been blocked by CORS policy

In **3–5 sentences**, explain what this error means to a teammate who has never seen it before. Then explain how you would fix it.

**Answer :**

Giải thích: Lỗi CORS là cơ chế bảo mật của trình duyệt, không phải do backend bị sập hay code bị lỗi.Trình duyệt coi localhost:3000 (React) và localhost:5000 (Express) là hai nguồn xa lạ vì khác cổng kết nối.Khi React gửi request, Express vẫn xử lý bình thường, nhưng trình duyệt đứng ở giữa chặn lại không cho React đọc dữ liệu trả về.Lý do là vì server Express chưa gửi kèm giấy phép để xác nhận cho phép cổng 3000 truy cập
Cách khắc phục: cấu hình middleware cors trực tiếp trên server Express để cấp quyền cho React. 

### Question 3.2 — Two Ways to Fetch

Look at these two pieces of code:

```javascript
// Version A
function getUser(id) {
  return fetch(`/api/users/${id}`).then((res) => res.json());
}

// Version B
async function getUser(id) {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}
```

Are they doing the same thing? Explain the difference (if any) to someone who only knows Version A.

***Answer :***
1. Cả hai đang làm cùng một việc: Đều gửi request lấy thông tin người dùng theo id, parse dữ liệu về dạng JSON và trả về một Promise chứa dữ liệu đó.
2.Giải thích: Không cần chấm .then(): chỉ cần đặt chữ await phía trước fetch() từ await sẽ tạm dừng hàm tại dòng đó để đợi fetch chạy xong rồi kết quả biến res  
   
## Section 4 — Small Coding Tasks

### Question 4.1 — Write a Middleware

Write an Express middleware function that logs the HTTP method, URL, and the time it took to process the request. Example output:

```
POST /api/notes — 23ms
```

It should work for all routes. Show where you would place it in your app (write the `app.use(...)` line).

**Answer :***

1. Hàm Middleware
```javascript
const requestLogger = (req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.originalUrl || req.url} - ${duration}ms`);
  });
  next(); 
};
```
2. Vị trí: Đặt ngay sau app = express() và trước tất cả các định nghĩa tuyến đường (routes) để nó có thể ghi nhận mọi request

### Question 4.2 — Write a Utility Function

Write a function called `groupByTag` that takes an array of bookmark objects and returns an object where the keys are tags and the values are arrays of bookmarks with that tag. Bookmarks without a tag should be grouped under `"untagged"`.

**Input:**

```javascript
[
  { title: 'React docs', url: 'https://react.dev', tag: 'frontend' },
  { title: 'MDN', url: 'https://developer.mozilla.org', tag: 'frontend' },
  { title: 'Express guide', url: 'https://expressjs.com', tag: 'backend' },
  { title: 'Random link', url: 'https://example.com' },
];
```

**Expected output:**

```javascript
{
  frontend: [
    { title: "React docs", url: "https://react.dev", tag: "frontend" },
    { title: "MDN", url: "https://developer.mozilla.org", tag: "frontend" }
  ],
  backend: [
    { title: "Express guide", url: "https://expressjs.com", tag: "backend" }
  ],
  untagged: [
    { title: "Random link", url: "https://example.com" }
  ]
}
```

***Answer :***
```javascript
function groupByTag(bookmarks) {
  return bookmarks.reduce((acc, item) => {
    const tag = item.tag || 'untagged';
    if (!acc[tag]) {
      acc[tag] = [];
    }
    acc[tag].push(item);
    return acc;
  }, {});
}
```
