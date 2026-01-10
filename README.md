# ind_proekt
# Программа для ведения учета личных финансов
transactions = [
    ["2024-03-25", "10:00", "приход", "зарплата", 50000, "ООО Работа"],
    ["2024-03-24", "19:30", "расход", "питание", 1500, "Пятерочка"],
    ["2024-03-24", "20:00", "расход", "развлечения", 2000, "Кинотеатр"],
    ["2024-03-23", "09:00", "расход", "транспорт", 100, "Метро"],
    ["2024-03-23", "12:00", "расход", "питание", 500, "Кафе"],
    ["2024-03-22", "18:00", "расход", "развлечения", 3000, "Концерт"],
    ["2024-03-22", "20:30", "расход", "питание", 1200, "Ресторан"],
    ["2024-03-21", "14:00", "приход", "аванс", 25000, "ООО Работа"],
    ["2024-03-21", "19:15", "расход", "развлечения", 1500, "Боулинг"],
    ["2024-03-20", "08:30", "расход", "транспорт", 150, "Такси"],
    ["2024-03-20", "21:00", "расход", "развлечения", 2500, "Театр"],
    ["2024-03-19", "17:45", "расход", "питание", 900, "Магазин"],
    ["2024-03-18", "19:20", "расход", "развлечения", 1800, "Караоке"],
    ["2024-03-17", "11:00", "приход", "подарок", 10000, "Родители"],
    ["2024-03-16", "16:30", "расход", "коммуналка", 5000, "ЖКХ"],
    ["2024-03-15", "13:00", "расход", "одежда", 8000, "Магазин одежды"],
    ["2024-03-14", "15:00", "расход", "медицина", 3000, "Аптека"],
    ["2024-03-13", "10:30", "приход", "премия", 15000, "ООО Работа"],
    ["2024-03-12", "14:20", "расход", "образование", 10000, "Курсы"],
    ["2024-03-11", "12:00", "расход", "кредит", 15000, "Банк"],
    ["2024-03-10", "19:40", "расход", "питание", 800, "Суши-бар"],
    ["2024-03-09", "20:10", "расход", "развлечения", 1200, "Бильярд"],
    ["2024-03-08", "09:15", "расход", "транспорт", 80, "Автобус"],
    ["2024-03-07", "15:45", "приход", "дивиденды", 8000, "Брокер"],
    ["2024-03-06", "18:30", "расход", "питание", 2000, "Ресторан"]
]


# ============================================
# 2. ПИРАМИДАЛЬНАЯ СОРТИРОВКА (HEAP SORT)
# ============================================

def heapify(arr, n, i, key_func):
    """Преобразует поддерево в кучу"""
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
    """Основная функция пирамидальной сортировки"""
    n = len(arr)

    # Строим максимальную кучу
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i, key_func)

    # Извлекаем элементы из кучи
    for i in range(n - 1, 0, -1):
        arr[i], arr[0] = arr[0], arr[i]
        heapify(arr, i, 0, key_func)

    return arr


# ============================================
# 3. ФУНКЦИИ ДЛЯ РАЗНЫХ ОТЧЕТОВ
# ============================================

def report1_all_income_last_n_days():
    """Отчет 1: Поступления за последние N дней"""
    print("\n" + "=" * 60)
    print("ОТЧЕТ 1: ПОСТУПЛЕНИЯ ЗА ПОСЛЕДНИЕ N ДНЕЙ")
    print("=" * 60)

    try:
        n = int(input("Введите количество дней (N): "))
    except:
        print("Ошибка! Введите число.")
        return

    # Фильтруем поступления
    income_list = [t for t in transactions if t[2] == "приход"]

    # Сортируем по дате (убывание) и сумме (убывание)
    # Сначала сортируем по сумме (вторичный ключ)
    income_list = heap_sort(income_list, lambda x: x[4])
    income_list.reverse()  # по убыванию суммы

    # Затем сортируем по дате (первичный ключ)
    income_list = heap_sort(income_list, lambda x: x[0])
    income_list.reverse()  # по убыванию даты

    print(f"\nПоступления за последние {n} дней:")
    print("-" * 60)
    print("Дата       | Время | Категория   | Сумма   | Контрагент")
    print("-" * 60)

    count = 0
    total = 0
    for t in income_list:
        print(f"{t[0]} | {t[1]} | {t[3]:11} | {t[4]:7} | {t[5]}")
        count += 1
        total += t[4]
        if count >= n * 2:
            break

    print("-" * 60)
    print(f"Всего: {count} записей, Сумма: {total} руб.")
    print("=" * 60)


def report2_expenses_by_category():
    """Отчет 2: Затраты по категории"""
    print("\n" + "=" * 60)
    print("ОТЧЕТ 2: ЗАТРАТЫ ПО КАТЕГОРИИ")
    print("=" * 60)

    categories = ["питание", "транспорт", "развлечения", "коммуналка",
                  "одежда", "медицина", "образование", "кредит"]

    print("Доступные категории:")
    for i, cat in enumerate(categories, 1):
        print(f"{i}. {cat}")

    try:
        choice = int(input("\nВыберите категорию (1-8): "))
        if choice < 1 or choice > 8:
            print("Неверный выбор!")
            return
        selected_category = categories[choice - 1]
    except:
        print("Ошибка ввода!")
        return

    # Фильтруем расходы по выбранной категории
    expenses = [t for t in transactions if t[2] == "расход" and t[3] == selected_category]

    # 1. Сортируем по сумме (убывание)
    expenses = heap_sort(expenses, lambda x: x[4])
    expenses.reverse()

    # 2. Сортируем по контрагенту (возрастание)
    expenses = heap_sort(expenses, lambda x: x[5])

    # 3. Сортируем по дате (убывание)
    expenses = heap_sort(expenses, lambda x: x[0])
    expenses.reverse()

    print(f"\nЗатраты по категории '{selected_category}':")
    print("-" * 60)
    print("Дата       | Время | Сумма   | Контрагент")
    print("-" * 60)

    total = 0
    for t in expenses:
        print(f"{t[0]} | {t[1]} | {t[4]:7} | {t[5]}")
        total += t[4]

    print("-" * 60)
    print(f"Всего: {len(expenses)} записей, Сумма: {total} руб.")
    print("=" * 60)


def report3_expenses_by_time():
    """Отчет 3: Затраты в определенное время"""
    print("\n" + "=" * 60)
    print("ОТЧЕТ 3: ЗАТРАТЫ ВО ВРЕМЕННОМ ПРОМЕЖУТКЕ")
    print("=" * 60)

    try:
        start_hour = int(input("Начальный час (0-23): "))
        end_hour = int(input("Конечный час (0-23): "))

        if start_hour < 0 or start_hour > 23 or end_hour < 0 or end_hour > 23:
            print("Часы должны быть от 0 до 23!")
            return
        if start_hour >= end_hour:
            print("Начальный час должен быть меньше конечного!")
            return
    except:
        print("Ошибка ввода!")
        return

    # Фильтруем расходы по времени
    expenses = []
    for t in transactions:
        if t[2] == "расход":
            hour = int(t[1].split(":")[0])  # получаем час из времени
            if start_hour <= hour <= end_hour:
                expenses.append(t)

    # 1. Сортируем по контрагенту (возрастание)
    expenses = heap_sort(expenses, lambda x: x[5])

    # 2. Сортируем по сумме (убывание)
    expenses = heap_sort(expenses, lambda x: x[4])
    expenses.reverse()

    print(f"\nЗатраты с {start_hour}:00 до {end_hour}:00:")
    print("-" * 60)
    print("Дата       | Время | Категория   | Сумма   | Контрагент")
    print("-" * 60)

    total = 0
    for t in expenses:
        print(f"{t[0]} | {t[1]} | {t[3]:11} | {t[4]:7} | {t[5]}")
        total += t[4]

    print("-" * 60)
    print(f"Всего: {len(expenses)} записей, Сумма: {total} руб.")
    print("=" * 60)


def show_all_transactions():
    print("\n" + "=" * 70)
    print("ВСЕ ТРАНЗАКЦИИ")
    print("=" * 70)
    print("Дата       | Время | Направление | Категория   | Сумма   | Контрагент")
    print("-" * 70)

    total_income = 0
    total_expense = 0

    for t in transactions:
        direction_symbol = "+" if t[2] == "приход" else "-"
        print(f"{t[0]} | {t[1]} | {t[2]:11} | {t[3]:11} | {t[4]:7} | {t[5]}")

        if t[2] == "приход":
            total_income += t[4]
        else:
            total_expense += t[4]

    print("-" * 70)
    print(f"Приход: {total_income} руб.")
    print(f"Расход: {total_expense} руб.")
    print(f"Баланс: {total_income - total_expense} руб.")
    print("=" * 70)


# ============================================
# 4. ГЛАВНОЕ МЕНЮ
# ============================================

def main():
    while True:
        print("\n" + "=" * 50)
        print("ПРОГРАММА 'ПЕРСОНАЛЬНЫЙ БЮДЖЕТ'")
        print("=" * 50)
        print("1. Показать все транзакции")
        print("2. Отчет 1: Поступления за N дней")
        print("3. Отчет 2: Затраты по категории")
        print("4. Отчет 3: Затраты в определенное время")
        print("5. Выход")
        print("=" * 50)

        choice = input("Выберите действие (1-5): ")

        if choice == "1":
            show_all_transactions()
        elif choice == "2":
            report1_all_income_last_n_days()
        elif choice == "3":
            report2_expenses_by_category()
        elif choice == "4":
            report3_expenses_by_time()
        elif choice == "5":
            print("\nСпасибо за использование программы!")
            break
        else:
            print("Неверный выбор! Попробуйте еще раз.")

        input("\nНажмите Enter для продолжения...")


# ============================================
# ЗАПУСК ПРОГРАММЫ
# ============================================

if __name__ == "__main__":
    print("=" * 60)
    print("Всего записей в базе:", len(transactions))
    print("=" * 60)

    main()
