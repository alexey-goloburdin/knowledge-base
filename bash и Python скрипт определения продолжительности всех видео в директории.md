# Bash

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

# Python

```python
import os
import subprocess
import re

def get_duration(filename):
    result = subprocess.run(
        ["ffmpeg", "-i", filename],
        stderr=subprocess.PIPE,
        stdout=subprocess.PIPE,
        text=True
    )
    duration_match = re.search(r'Duration: (\d+):(\d+):(\d+\.\d+)', result.stderr)
    if duration_match:
        hours, minutes, seconds = map(float, duration_match.groups())
        total_seconds = int(hours * 3600 + minutes * 60 + seconds)
        return total_seconds
    return 0

def convert_to_hms(seconds):
    hours = seconds // 3600
    minutes = (seconds % 3600) // 60
    seconds = seconds % 60
    return hours, minutes, seconds

def main():
    total_duration = 0
    for filename in os.listdir('.'):
        if os.path.isfile(filename):
            duration = get_duration(filename)
            if duration:
                print(f"Файл: {filename}, Длительность: {duration} секунд")
                total_duration += duration

    hours, minutes, seconds = convert_to_hms(total_duration)
    print(f"Общая длительность всех видео: {hours:02}:{minutes:02}:{seconds:02}")

if __name__ == "__main__":
    main()
```

[[Tools]]
[[codeblock]]
