```js
const categoryColors = {
	"эффективность": "#fff9f2", "разработка": "#ebffff", "edtech": "#f5fff7",
	"съёмка": "#f7f9ff", "художественное": "#fffdf0",
	"психология": "#faf1dd",
	"default": "#f5f5f5"
};

const getCategoryColor = category => {
    if (category in categoryColors) return categoryColors[category];
    else return categoryColors["default"];
}

const getBooks = () => dv.pages("").where(page => {
    if (page["Тип"] && page["Тип"].path == "types/Книга.md" && page.file.folder != "templates"  && page.file.folder != "templater" && !page["Закончил читать"] && page["Начал читать"]) {
      return page;
    }
}).sort(p => p["Начал читать"], "desc");

const formatDate = d => {
    if (!d) return '';
    return d.year === dv.date("now").year
        ? d.toFormat("d MMMM", { locale: "ru" })
        : d.toFormat("d MMMM yyyy", { locale: "ru" });
}

window.books = {getCategoryColor, getBooks, formatDate};
```