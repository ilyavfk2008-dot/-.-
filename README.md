markdown
# Movie Library
**Автор:** Горбачев Илья
**Вариант:** №1 Movie Library
**Дата сдачи:** 15.05.2026
---
## Описание программы
Кратко: что делает программа, зачем она нужна, какие проблемы решает.
Пример:
*"Movie Library — это консольное приложение для ведения своей библиотеки фильмов. Позволяет
добавлять фильмы в свою библиотеку, оценивать их, сортировать по жанру и по году выпуска."*
---
## Требования для запуска
Что нужно установить на компьютер:
- Python 3.10 или выше
- Библиотеки: `pip install matplotlib requests`
---

import tkinter as tk
from tkinter import ttk, messagebox
import json
import os

class MovieLibrary:
    def __init__(self, root):
        self.root = root
        self.root.title("Movie Library")
        self.movies = []
        self.load_data()
        self.create_widgets()
        self.update_table()

    def create_widgets(self):
        # Поля ввода
        tk.Label(self.root, text="Название:").grid(row=0, column=0, sticky="w", padx=5, pady=5)
        self.title_entry = tk.Entry(self.root, width=30)
        self.title_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Жанр:").grid(row=1, column=0, sticky="w", padx=5, pady=5)
        self.genre_entry = tk.Entry(self.root, width=30)
        self.genre_entry.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Год выпуска:").grid(row=2, column=0, sticky="w", padx=5, pady=5)
        self.year_entry = tk.Entry(self.root, width=30)
        self.year_entry.grid(row=2, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Рейтинг (0-10):").grid(row=3, column=0, sticky="w", padx=5, pady=5)
        self.rating_entry = tk.Entry(self.root, width=30)
        self.rating_entry.grid(row=3, column=1, padx=5, pady=5)

        # Кнопка добавления
        self.add_button = tk.Button(self.root, text="Добавить фильм", command=self.add_movie)
        self.add_button.grid(row=4, column=0, columnspan=2, pady=10)

        # Фильтры
        tk.Label(self.root, text="Фильтр по жанру:").grid(row=5, column=0, sticky="w", padx=5, pady=5)
        self.genre_filter = ttk.Combobox(self.root, state="readonly")
        self.genre_filter.grid(row=5, column=1, padx=5, pady=5)
        self.genre_filter.bind("<<ComboboxSelected>>", self.apply_filters)

        tk.Label(self.root, text="Фильтр по году:").grid(row=6, column=0, sticky="w", padx=5, pady=5)
        self.year_filter = ttk.Combobox(self.root, state="readonly")
        self.year_filter.grid(row=6, column=1, padx=5, pady=5)
        self.year_filter.bind("<<ComboboxSelected>>", self.apply_filters)

        # Таблица
        columns = ("Название", "Жанр", "Год выпуска", "Рейтинг")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings", height=10)

        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=120)

        self.tree.grid(row=7, column=0, columnspan=2, padx=5, pady=10, sticky="nsew")

        # Полоса прокрутки
        scrollbar = ttk.Scrollbar(self.root, orient="vertical", command=self.tree.yview)
        scrollbar.grid(row=7, column=2, sticky="ns")
        self.tree.configure(yscrollcommand=scrollbar.set)

    def validate_input(self):
        """Проверка корректности ввода"""
        try:
            year = int(self.year_entry.get())
            if year < 1800 or year > 2030:
                messagebox.showerror("Ошибка", "Год должен быть между 1800 и 2030")
                return False
        except ValueError:
            messagebox.showerror("Ошибка", "Год должен быть числом")
            return False

        try:
            rating = float(self.rating_entry.get().replace(',', '.'))
            if rating < 0 or rating > 10:
                messagebox.showerror("Ошибка", "Рейтинг должен быть от 0 до 10")
                return False
        except ValueError:
            messagebox.showerror("Ошибка", "Рейтинг должен быть числом от 0 до 10")
            return False

        if not self.title_entry.get():
            messagebox.showerror("Ошибка", "Введите название фильма")
            return False

        return True

    def add_movie(self):
        """Добавление нового фильма"""
        if not self.validate_input():
            return

        movie = {
            "title": self.title_entry.get(),
            "genre": self.genre_entry.get(),
            "year": int(self.year_entry.get()),
            "rating": float(self.rating_entry.get().replace(',', '.'))
        }

        self.movies.append(movie)
        self.save_data()
        self.update_table()
        self.clear_entries()
        self.update_filters()

    def clear_entries(self):
        """Очистка полей ввода"""
        self.title_entry.delete(0, tk.END)
        self.genre_entry.delete(0, tk.END)
        self.year_entry.delete(0, tk.END)
        self.rating_entry.delete(0, tk.END)

    def load_data(self):
        """Загрузка данных из JSON"""
        if os.path.exists("movies.json"):
            try:
                with open("movies.json", "r", encoding="utf-8") as f:
                    self.movies = json.load(f)
            except (json.JSONDecodeError, IOError):
                self.movies = []
        else:
            self.movies = []

    def save_data(self):
        """Сохранение данных в JSON"""
        with open("movies.json", "w", encoding="utf-8") as f:
            json.dump(self.movies, f, ensure_ascii=False, indent=4)

    def update_table(self, filtered_movies=None):
        """Обновление таблицы"""
        for item in self.tree.get_children():
            self.tree.delete(item)

        movies_to_show = filtered_movies if filtered_movies is not None else self.movies

        for movie in movies_to_show:
            self.tree.insert("", "end", values=(
                movie["title"],
                movie["genre"],
                movie["year"],
                f"{movie['rating']:.1f}"
            ))

    def update_filters(self):
        """Обновление списков фильтров"""
        genres = sorted(set(movie["genre"] for movie in self.movies if movie["genre"]))
        years = sorted(set(str(movie["year"]) for movie in self.movies))

        self.genre_filter["values"] = ["Все"] + genres
        self.year_filter["values"] = ["Все"] + years

        self.genre_filter.set("Все")
        self.year_filter.set("Все")

    def apply_filters(self, event=None):
        """Применение фильтров"""
        selected_genre = self.genre_filter.get()
        selected_year = self.year_filter.get()

        filtered_movies = self.movies

        if selected_genre != "Все":
            filtered_movies = [m for m in filtered_movies if m["genre"] == selected_genre]

        if selected_year != "Все":
            filtered_movies = [m for m in filtered_movies if str(m["year"]) == selected_year]

        self.update_table(filtered_movies)

# Запуск приложения
if __name__ == "__main__":
    root = tk.Tk()
    app
