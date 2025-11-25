# Shopping List & Model API


---

REST API на FastAPI, включающий:

* Список покупок (CRUD-операции)
* Аналитику расходов по группам товаров
* Интеграцию с обучённой ML-моделью (Iris Logistic Regression)
* Демонстрационные запросы (`requests`) для покрытия всех методов API


---

#  **О проекте**

Проект демонстрирует реализацию полноценного REST API c использованием FastAPI в Google Colab. Он содержит:

###  **1. CRUD-API для списка покупок**

Каждый товар включает:

* название
* группу (категорию)
* цену
* единицу измерения
* количество

###  **2. Аналитика** `/expenses`

Возвращает:

* сумму расходов по каждой товарной группе
* общую сумму расходов

###  **3. Интеграция с ML-моделью** `/predict`

Модель Iris (LogisticRegression) обучается прямо в Colab и сохраняется в `model.joblib`.

Эндпоинт `/predict` принимает 4 числовых признака и возвращает предсказанный класс цветка и вероятности.

###  **4. Покрытие запросами**

Предоставлена отдельная ячейка с запросами `requests`, которая полностью покрывает:

* создание товара
* получение списка
* получение по ID
* обновление
* удаление
* аналитику `/expenses`
* предсказание `/predict`

---

#  **Установка и запуск (Colab)**

### **1. Установка зависимостей**

```bash
!pip install -q fastapi "uvicorn[standard]" nest_asyncio scikit-learn pandas requests joblib
```

### **2. Обучение модели (создаёт model.joblib)**

В ноутбуке есть ячейка, которая обучает Logistic Regression на данных Iris:

* сохраняет модель
* выводит качество

### **3. Запуск FastAPI сервера в фоне**

Используется:

* `nest_asyncio` для повторного запуска событийного цикла
* `uvicorn` в отдельном потоке

### **4. Демонстрационные запросы**

Отправляются через `requests`, включая CRUD и `/predict`.

---

#  **Документация OpenAPI / Swagger**

После запуска API доступны:

* Swagger UI: **[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)**
* ReDoc: **[http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)**

Метаданные API настроены через параметры FastAPI:

* title
* description
* version
* contact
* license

---

#  **Эндпоинты API**

Ниже приведён полный перечень реализованных методов.

---

## **Root**

### **GET /**

Возвращает краткое сообщение о сервисе.

Ответ:

```json
{
  "message": "Shopping List & Model API. See /docs for interactive documentation."
}
```

---

# 🛒 **Items (CRUD)**

## **POST /items/**

Создать товар.

* Request: `ItemCreate`
* Response: `Item` (201)

---

## **GET /items/**

Получить список всех товаров.

* Response: `List[Item]`

---

## **GET /items/{item_id}**

Получить товар по ID.

* 404 если товара нет

---

## **PUT /items/{item_id}**

Обновить товар по ID.

* Request: `ItemCreate`
* Response: `Item`
* Ошибки: 404 — не найден

---

## **DELETE /items/{item_id}**

Удалить товар.

* Response: 204 No Content
* Ошибки: 404

---

#  **Analytics**

## **GET /expenses**

Возвращает отчёт о расходах.

Response:

* `per_group`: список `{group, expense}`
* `total`: общая сумма

Документация оформлена через `response_description`, поэтому отображается в Swagger.

---

#  **ML Model Prediction**

## **POST /predict**

Прогнозирует класс цветка Iris.

Request:

```json
{"features": [5.1, 3.5, 1.4, 0.2]}
```

Response:

```json
{
  "predicted_class": "setosa",
  "predicted_index": 0,
  "probabilities": [0.97, 0.02, 0.01]
}
```

---

#  **Примеры запросов (curl / requests)**

## Создание товара (curl):

```bash
curl -X POST "http://127.0.0.1:8000/items/" -H "Content-Type: application/json" -d \
'{"name":"Milk","group":"Продовольствие","price":0.99,"unit":"L","quantity":2}'
```

## Прогноз модели (curl):

```bash
curl -X POST "http://127.0.0.1:8000/predict" -H "Content-Type: application/json" -d \
'{"features":[5.1,3.5,1.4,0.2]}'
```

## Python (`requests`):

```python
import requests
BASE="http://127.0.0.1:8000"
res = requests.post(BASE + "/items/", json={
    "name":"Milk", "group":"Продовольствие",
    "price":0.99, "unit":"L", "quantity":2
})
print(res.json())
```

---

#  **Схемы данных (Pydantic models)**

## **ItemCreate**

* `name: str`
* `group: str`
* `price: float` (gt=0)
* `unit: str`
* `quantity: float` (gt=0)

## **Item**

* `id: str`
* остальные поля из ItemCreate

## **ExpensePerGroup**

* `group: str`
* `expense: float`

## **ExpensesResponse**

* `per_group: List[ExpensePerGroup]`
* `total: float`

## **PredictRequest**

* `features: List[float]` (длина 4)

## **PredictResponse**

* `predicted_class: str`
* `predicted_index: int`
* `probabilities: List[float]`

---
