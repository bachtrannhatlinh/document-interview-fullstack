# 2. Callback, Promise, Async/Await
- **Callback**: Là hàm được truyền vào một hàm khác như một đối số, sẽ được gọi lại (callback) khi tác vụ bất đồng bộ hoàn thành. Dễ gặp callback hell khi lồng nhiều callback.

Ví dụ:
```js
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

- **Promise**: Là một đối tượng đại diện cho một giá trị có thể có trong tương lai (thành công hoặc thất bại). Giúp tránh callback hell, dễ quản lý chuỗi tác vụ bất đồng bộ.

Ví dụ:
```js
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}
readFilePromise('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

- **Async/Await**: Cú pháp giúp viết code bất đồng bộ trông như đồng bộ, dễ đọc, dễ debug hơn. Dựa trên Promise.

Ví dụ:
```js
async function readFileAsync() {
  try {
    const data = await readFilePromise('file.txt');
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
readFileAsync();
```
