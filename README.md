# ind_proekt
# Программа для ведения учета личных финансов
import datetime
import os

# ==============================
# ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
# ==============================

def safe_input_int(prompt, min_val=None, max_val=None):
    while True:
        try:
            value = int(input(prompt))
            if min_val is not None and value < min_val:
                print(f"Значение должно быть не меньше {min_val}.")
                continue
            if max_val is not None and value > max_val:
                print(f"Значение должно быть не больше {max_val}.")
                continue
            return value
        except ValueError:
            print("Пожалуйста, введите целое число.")

def safe_input_time(prompt):
    while True:
        time_str = input(prompt).strip()
        try:
            datetime.datetime.strptime(time_str, "%H:%M")
            return time_str
        except ValueError:
            print("Неверный формат времени. Используйте ЧЧ:ММ (например, 18:30).")

def safe_input_date(prompt):
    while True:
        date_str = input(prompt).strip()
        try:
            datetime.datetime.strptime(date_str, "%Y-%m-%d")
            return date_str
        except ValueError:
            print("Неверный формат даты. Используйте ГГГГ-ММ-ДД (например, 2026-01-12).")

def parse_time(time_str):
    return datetime.datetime.strptime(time_str, "%H:%M").time()

def invert_string_for_desc_sort(s):
    """Преобразует строку в кортеж отрицательных кодов символов для инверсии порядка при сортировке по убыванию."""
    return tuple(-ord(c) for c in s)

# ==============================
# ПИРАМИДАЛЬНАЯ СОРТИРОВКА
# ==============================

def heapify(arr, n, i, key_func):
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2

    if left < n and key_func(arr[left]) > key_func(arr[largest]):
        largest = left
    if right < n and key_func(arr[right]) > key_func(arr[largest]):
        largest = right

    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest, key_func)

def heap_sort(arr, key_func):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i, key_func)
    for i in range(n - 1, 0, -1):
        arr[i], arr[0] = arr[0], arr[i]
        heapify(arr, i, 0, key_func)

# ==============================
# ЗАГРУЗКА ДАННЫХ
# ==============================

def load_transactions(filename="transactions.txt"):
    if not os.path.exists(filename):
        print(f"Ошибка: файл '{filename}' не найден.")
        return None

    transactions = []
    line_num = 0
    try:
        with open(filename, "r", encoding="utf-8") as f:
            for line in f:
                line_num += 1
                line = line.strip()
                if not line:
                    continue
                parts = line.split("|")
                if len(parts) != 6:
                    print(f"Предупреждение: пропущена строка {line_num} (ожидается 6 полей).")
                    continue
                date, time, direction, category, amount_str, counterparty = parts

                if direction not in ("приход", "расход"):
                    print(f"Предупреждение: строка {line_num} — неверное направление '{direction}'. Пропущено.")
                    continue

                try:
                    amount = int(amount_str)
                    if amount <= 0:
                        print(f"Предупреждение: строка {line_num} — сумма должна быть положительной. Пропущено.")
                        continue
                except ValueError:
                    print(f"Предупреждение: строка {line_num} — неверная сумма '{amount_str}'. Пропущено.")
                    continue

                try:
                    datetime.datetime.strptime(date, "%Y-%m-%d")
                    datetime.datetime.strptime(time, "%H:%M")
                except ValueError:
                    print(f"Предупреждение: строка {line_num} — неверный формат даты/времени. Пропущено.")
                    continue

                transactions.append({
                    "date": date,
                    "time": time,
                    "direction": direction,
                    "category": category,
                    "amount": amount,
                    "counterparty": counterparty
                })
    except Exception as e:
        print(f"Ошибка при чтении файла: {e}")
        return None
    return transactions

def save_transactions(transactions, filename="transactions.txt"):
    try:
        with open(filename, "w", encoding="utf-8") as f:
            for t in transactions:
                f.write(f"{t['date']}|{t['time']}|{t['direction']}|{t['category']}|{t['amount']}|{t['counterparty']}\n")
        print("Изменения сохранены в файл.")
    except Exception as e:
        print(f"Ошибка при сохранении: {e}")

# ==============================
# ОТЧЁТЫ (ПОЛНОСТЬЮ СООТВЕТСТВУЮЩИЕ УСЛОВИЮ)
# ==============================

def report_1(transactions, n_days):
    all_dates = [datetime.datetime.strptime(t["date"], "%Y-%m-%d").date() for t in transactions]
    today = max(all_dates) if all_dates else datetime.date.today()
    cutoff = today - datetime.timedelta(days=n_days)

    income = [t for t in transactions if t["direction"] == "приход" and
              datetime.datetime.strptime(t["date"], "%Y-%m-%d").date() >= cutoff]

    if not income:
        print("\nНет поступлений за указанный период.")
        return

    def key_func(t):
        dt = datetime.datetime.strptime(f"{t['date']} {t['time']}", "%Y-%m-%d %H:%M")
        return (dt, t["amount"])  # дата ↓, сумма ↓ → heap_sort (убывание) — всё верно

    heap_sort(income, key_func)

    print(f"\n=== Отчёт 1: Поступления за последние {n_days} дней ===")
    for t in income:
        print(f"{t['date']} {t['time']} | {t['category']:12} | {t['amount']:6} ₽ | {t['counterparty']}")

def report_2(transactions, category):
    expenses = [t for t in transactions if t["direction"] == "расход" and t["category"] == category]

    if not expenses:
        print(f"\nНет расходов по категории '{category}'.")
        return

    def key_func(t):
        dt = datetime.datetime.strptime(f"{t['date']} {t['time']}", "%Y-%m-%d %H:%M")
        contr_key = invert_string_for_desc_sort(t["counterparty"])  # для контрагента ↑ при сортировке ↓
        return (dt, contr_key, t["amount"])

    heap_sort(expenses, key_func)

    print(f"\n=== Отчёт 2: Расходы по категории '{category}' ===")
    for t in expenses:
        print(f"{t['date']} {t['time']} | {t['counterparty']:20} | {t['amount']:6} ₽")

def report_3(transactions, start_time_str, end_time_str):
    try:
        start_time = parse_time(start_time_str)
        end_time = parse_time(end_time_str)
    except:
        print("Неверный формат времени.")
        return

    if start_time > end_time:
        print("Начальное время не должно быть позже конечного.")
        return

    expenses = []
    for t in transactions:
        if t["direction"] != "расход":
            continue
        trans_time = parse_time(t["time"])
        if start_time <= trans_time <= end_time:
            expenses.append(t)

    if not expenses:
        print(f"\nНет расходов в период с {start_time_str} до {end_time_str}.")
        return

    def key_func(t):
        contr_key = invert_string_for_desc_sort(t["counterparty"])
        return (t["amount"], contr_key)  # сумма ↓, контрагент ↑

    heap_sort(expenses, key_func)

    print(f"\n=== Отчёт 3: Расходы с {start_time_str} до {end_time_str} ===")
    for t in expenses:
        print(f"{t['time']} | {t['counterparty']:20} | {t['amount']:6} ₽ | {t['category']}")

# ==============================
# МЕНЮ
# ==============================

def main():
    transactions = load_transactions()
    if transactions is None:
        print("Не удалось загрузить данные. Убедитесь, что файл 'transactions.txt' существует и имеет правильный формат.")
        return

    while True:
        print("\n" + "="*50)
        print("Персональный бюджет — Главное меню")
        print("="*50)
        print("1. Показать все транзакции")
        print("2. Отчёт 1: Поступления за N дней")
        print("3. Отчёт 2: Расходы по категории")
        print("4. Отчёт 3: Расходы в интервале времени")
        print("5. Добавить транзакцию")
        print("6. Сохранить изменения")
        print("0. Выход")
        choice = safe_input_int("Выберите действие (0-6): ", min_val=0, max_val=6)

        if choice == 0:
            print("До свидания!")
            break
        elif choice == 1:
            print("\n=== Все транзакции ===")
            for i, t in enumerate(transactions, 1):
                print(f"{i:2}. {t['date']} {t['time']} | {t['direction']:6} | {t['category']:12} | {t['amount']:6} ₽ | {t['counterparty']}")
        elif choice == 2:
            n = safe_input_int("Введите количество дней N: ", min_val=1)
            report_1(transactions, n)
        elif choice == 3:
            cat = input("Введите категорию (например, 'питание'): ").strip()
            if not cat:
                print("Категория не может быть пустой.")
            else:
                report_2(transactions, cat)
        elif choice == 4:
            start_t = safe_input_time("Введите начальное время (ЧЧ:ММ): ")
            end_t = safe_input_time("Введите конечное время (ЧЧ:ММ): ")
            report_3(transactions, start_t, end_t)
        elif choice == 5:
            print("\n--- Добавление новой транзакции ---")
            date = safe_input_date("Дата (ГГГГ-ММ-ДД): ")
            time = safe_input_time("Время (ЧЧ:ММ): ")
            direction = input("Направление (приход/расход): ").strip().lower()
            if direction not in ("приход", "расход"):
                print("Направление должно быть 'приход' или 'расход'.")
                continue
            category = input("Категория (например, зарплата, питание): ").strip()
            if not category:
                print("Категория не может быть пустой.")
                continue
            amount = safe_input_int("Сумма (положительное число): ", min_val=1)
            counterparty = input("Контрагент: ").strip()
            if not counterparty:
                print("Контрагент не может быть пустым.")
                continue
            transactions.append({
                "date": date, "time": time, "direction": direction,
                "category": category, "amount": amount, "counterparty": counterparty
            })
            print("Транзакция добавлена.")
        elif choice == 6:
            save_transactions(transactions)

if __name__ == "__main__":
    main()
