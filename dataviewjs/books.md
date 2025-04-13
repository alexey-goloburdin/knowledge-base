```js
const CATEGORY_COLORS = {
    "эффективность": "#fff9f2", "разработка": "#ebffff", "edtech": "#f5fff7",
    "съёмка": "#f7f9ff", "художественное": "#fffdf0",
    "психология": "#faf1dd",
    "default": "#f5f5f5"
};

const getCategoryColor = category => {
    return (category in CATEGORY_COLORS) ? CATEGORY_COLORS[category]: CATEGORY_COLORS["default"];
}

const getBooks = () => {
    const books = dv.pages("")
        .where(page => {
            if (page["Тип"] && page["Тип"].path == "types/Книга.md"
                    && page.file.folder != "templates"
                    && page.file.folder != "templater"){
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

const renderBook = (book, additionalHtml) => {
    additionalHtml = eval("`"+additionalHtml+"`");
    return `<div class="book">
        <a data-tooltip-position="top" data-href="${book.file.name}" href="${book.file.name}.md" class="internal-link" target="_blank" rel="noopener nofollow"><img src="${book.Обложка}" data-filename="${book.file.name}" /></a>
        ${book.Progress}
        ${additionalHtml}
        <div class="categories">${renderCategories(book)}</div>
        <div class="pages">${book.Страниц} стр.</div>
    </div>`;
}

const renderBooks = (books, additionalBookHtml) => {
    let booksHtml = [];
    for (const book of booksFiles) {
        booksHtml.push(renderBook(book, additionalBookHtml))
    }
	return booksHtml.join("");
}

window.books = {getBooks, formatDate, renderBooks};
```