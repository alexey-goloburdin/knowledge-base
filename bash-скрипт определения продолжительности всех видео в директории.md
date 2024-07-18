```bash
#!/bin/bash

# Проверьте, установлен ли ffmpeg
if ! command -v ffmpeg &> /dev/null
then
    echo "ffmpeg не установлен. Установите ffmpeg и попробуйте снова."
    exit 1
fi

# Инициализация общей продолжительности
total_duration=0

# Функция для конвертации времени в секунды
convert_to_seconds() {
    IFS=: read -r h m s <<< "$1"
    s=${s%.*} # Удаление дробной части
    echo "$((10#$h * 3600 + 10#$m * 60 + 10#$s))"
}

# Цикл по всем видеофайлам в текущей директории
for file in *; do
    if [[ -f "$file" ]]; then
        # Получение информации о видео
        duration=$(ffmpeg -i "$file" 2>&1 | grep Duration | awk '{print $2}' | tr -d ,)
        if [[ -n "$duration" ]]; then
            echo "Файл: $file, Длительность: $duration"
            seconds=$(convert_to_seconds "$duration")
            total_duration=$((total_duration + seconds))
        fi
    fi
done

# Конвертация общей продолжительности обратно в часы, минуты и секунды
hours=$((total_duration / 3600))
minutes=$(( (total_duration % 3600) / 60 ))
seconds=$((total_duration % 60))

# Вывод общей длительности
printf "Общая длительность всех видео: %02d:%02d:%02d\n" $hours $minutes $seconds
```

[[Tools]]