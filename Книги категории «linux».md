---
cssclasses:
  - wide-85
---
# Linux (`$= dv.pages("").where(page => {if (page["Тип"] && page["Тип"].path == "types/Книга.md" && !["templates", "templater"].includes(page.file.folder) && page.Категории.contains("linux")) {return page;}}).length`)

```dataviewjs
// import code
eval(
    (await app.vault.read(app.vault.getAbstractFileByPath("dataviewjs/books.md")))
    .replace("```js", "").replace("```", "")
);

const booksFiles = window.books.getBooks()
    .where(p => p["Категории"].contains("linux"))
    .sort(p => p["Добавлена в базу"], "desc");

dv.el("div", window.books.renderBooks(booksFiles, book => `добавлена ${window.books.formatDate(book["Добавлена в базу"])}<br>`), {cls: "books"});
```