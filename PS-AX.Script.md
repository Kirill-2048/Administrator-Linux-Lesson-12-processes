#!/bin/bash

printf "%-6s %-6s %-6s %-6s %-8s %s\n" "PID" "TTY" "STAT" "TIME" "COMMAND"

for pid in $(ls -d /proc/[0-9]*/ | cut -d/ -f3 | sort -n); do
    if [ -d "/proc/$pid" ]; then
        #Получение информации о процессе
        stat=$(cat "/proc/$pid/stat" 2>/dev/null)
        if [ -z "$stat" ]; then
            continue
        fi

        #Извлечение данных из /proc/$pid/stat
        comm=$(echo "$stat" | awk '{gsub("[(]|[)]", "", $2); print $2}')
        state=$(echo "$stat" | awk '{print $3}')
        tty_nr=$(echo "$stat" | awk '{print $7}')
        utime=$(echo "$stat" | awk '{print $14}')
        stime=$(echo "$stat" | awk '{print $15}')
        
        #Вычисление времени CPU (сек)
        total_time=$((utime + stime))
        total_time_sec=$((total_time / $(getconf CLK_TCK)))
        minutes=$((total_time_sec / 60))
        seconds=$((total_time_sec % 60))
        time_str=$(printf "%02d:%02d" "$minutes" "$seconds")

        #Определение TTY
        if [ "$tty_nr" -eq 0 ]; then
            tty="?"
        else
            tty_major=$((tty_nr >> 8))
            tty_minor=$((tty_nr & 0xFF))
            tty="tty$((tty_major * 256 + tty_minor))"
        fi

        #Вывод информации
        printf "%-6s %-6s %-6s %-6s %-8s %s\n" "$pid" "$tty" "$state" "$time_str" "$comm"
    fi
done
