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

1. Thiếu await tại dòng const note = Note.create({
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

## Section 3 — Explain In Your Own Words

### Question 3.1 — The CORS Error

You are building a React + Express app. The React dev server runs on `localhost:3000` and Express runs on `localhost:5000`. You try to fetch data from Express and get this error in the browser console:

> Access to fetch at 'http://localhost:5000/api/notes' from origin 'http://localhost:3000' has been blocked by CORS policy

In **3–5 sentences**, explain what this error means to a teammate who has never seen it before. Then explain how you would fix it.

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

## Section 4 — Small Coding Tasks

### Question 4.1 — Write a Middleware

Write an Express middleware function that logs the HTTP method, URL, and the time it took to process the request. Example output:

```
POST /api/notes — 23ms
```

It should work for all routes. Show where you would place it in your app (write the `app.use(...)` line).

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
