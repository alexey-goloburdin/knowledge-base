```js
const CATEGORY_BG_COLORS = {
    "эффективность": "#fff9f2", "разработка": "#ebffff", "edtech": "#f5fff7",
    "съёмка": "#f7f9ff", "художественное": "#fffdf0",
    "психология": "#faf1dd",
    "архитектура ПО": "#fff3f3", "AI": "#dfe6ff",
    //"фронтенд": "red",
    "Python": "#25912b",
    "менеджмент": "#f9fd95",
    "дизайн": "#d324ff",
    "default": "#f5f5f5"
};

const CATEGORY_FONT_COLORS = {
    "Python": "#fff",
    "дизайн": "#ffffec",
};

const getCategoryBgColor = category => {
    return (category in CATEGORY_BG_COLORS) ? CATEGORY_BG_COLORS[category]: CATEGORY_BG_COLORS["default"];
}

const getCategoryFontColor = category => {
    return (category in CATEGORY_FONT_COLORS) ? CATEGORY_FONT_COLORS[category]: CATEGORY_FONT_COLORS["default"];
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
        c => `<span class="category" style="background-color: ${getCategoryBgColor(c)}; color: ${getCategoryFontColor(c)};">${c}</span>`
    ).join(" ");
}

const renderBook = (book, additionalBookRenderFunction) => {
    return `<div class="book">
        <a href="${book.file.name}.md" class="internal-link" target="_blank" rel="noopener nofollow"><img src="${book.Обложка}" data-filename="${book.file.name}" /></a>
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