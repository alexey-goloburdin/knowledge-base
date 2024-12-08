---
Добавлена в базу: 19.07.2024 в 13:47
Ссылка: https://incompetech.com/graphpaper/
Тип: "[[Ссылка на полезное]]"
---
Варианты: https://incompetech.com/graphpaper/

## Подходящий вариант — клетки

https://incompetech.com/graphpaper/lite/
Square size: 0.25 inches
Grid Line Weight: Fine
Number of Squares: 31 x 43
Color: dddddd

## Создать многостраничный PDF

```python
# pip install pypdf2
from PyPDF2 import PdfReader, PdfWriter

def duplicate_pdf_page(input_pdf_path, output_pdf_path, number_of_copies):
    reader = PdfReader(input_pdf_path)
    writer = PdfWriter()

    # Предполагаем, что дублировать нужно первую страницу
    page_to_duplicate = reader.pages[0]

    # Добавляем страницу нужное количество раз
    for _ in range(number_of_copies):
        writer.add_page(page_to_duplicate)

    # Сохраняем результат
    with open(output_pdf_path, 'wb') as output_file:
        writer.write(output_file)

# Параметры
input_pdf_path = 'squares.pdf'  # Укажите путь к исходному PDF
output_pdf_path = 'notebook.pdf'  # Укажите путь к выходному PDF
number_of_copies = 100  # Укажите количество копий

# Запуск функции
duplicate_pdf_page(input_pdf_path, output_pdf_path, number_of_copies)
```


[[Tools]]