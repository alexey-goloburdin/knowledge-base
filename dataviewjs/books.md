```js
const CATEGORY_COLORS = {
    "эффективность": "#fff9f2", "разработка": "#ebffff", "edtech": "#f5fff7",
    "съёмка": "#f7f9ff", "художественное": "#fffdf0",
    "психология": "#faf1dd",
    "архитектура ПО": "#fff3f3", "AI": "#dfe6ff",
    "default": "#f5f5f5"
};

const getCategoryColor = category => {
    return (category in CATEGORY_COLORS) ? CATEGORY_COLORS[category]: CATEGORY_COLORS["default"];
}

const getBooks = () => {
    const books = dv.pages("")
        .where(page => {
            if (page["Тип"] && page["Тип"].path == "types/Книга.md"
                    && !["templates", "templater"].includes(page.file.folder)){
                return page;
            }
        });
    return books;
};

const formatDate = d => {
    if (!d) return '';
    return d.year === dv.date("now").year
        ? d.toFormat("d MMMM", { locale: "ru" })
        : d.toFormat("d MMMM yyyy", { locale: "ru" });
}

const renderCategories = book => {
    return (book.Категории ? book.Категории : []).map(
        c => `<span class="category" style="background-color: ${getCategoryColor(c)}">${c}</span>`
    ).join(" ");
}

const renderBook = (book, additionalBookRenderFunction) => {
    return `<div class="book">
        <a data-tooltip-position="top" data-href="${book.file.name}" href="${book.file.name}.md" class="internal-link" target="_blank" rel="noopener nofollow"><img src="${book.Обложка}" data-filename="${book.file.name}" /></a>
        ${book.Progress}
        ${additionalBookRenderFunction(book)}
        <div class="categories">${renderCategories(book)}</div>
        <div class="pages">${book.Страниц} стр.</div>
    </div>`;
}

const renderBooks = (booksFiles, additionalBookRenderFunction) => {
    let booksHtml = [];
    for (const book of booksFiles) {
        booksHtml.push(renderBook(book, additionalBookRenderFunction))
    }
	return booksHtml.join("");
}

window.books = {getBooks, formatDate, renderBooks};
```