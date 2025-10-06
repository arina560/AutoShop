# AutoShop
МДиСУБД, 353501, Петрожицкая

## Описание
Проект представляет собой систему управления интернет-магазином автомобилей.  
Система включает функциональность для управления пользователями, автомобилями, брендами, моделями, заказами, корзинами, поставщиками и купонами.  
---

## Функциональные требования

1. **Авторизация/аутентификация пользователя**:
   - Регистрация нового пользователя с проверкой уникальности email.  
   - Вход в систему с использованием email и пароля.  
   - Выход из системы.  

2. **Управление пользователями (CRUD)**:
   - Администраторы могут создавать новых пользователей.  
   - Возможность просмотра списка пользователей с фильтрацией.  
   - Обновление информации о пользователях (пароль, имя, роль).  
   - Удаление учетных записей пользователей.  

3. **Система ролей**:
   - Клиент (customer).  
   - Менеджер (manager).  
   - Администратор (admin).  

4. **Управление каталогом автомобилей**:
   - Добавление, редактирование и удаление автомобилей.  
   - Хранение брендов и моделей.  
   - Указание характеристик автомобиля: год выпуска, цвет, цена, пробег, статус.  

5. **Система заказов**:
   - Добавление автомобилей в корзину.  
   - Оформление заказа с указанием адреса и купона.  
   - Возможность нескольких автомобилей в одном заказе.  
   - Статусы заказов: *pending*, *paid*, *delivered*, *cancelled*.  

6. **Корзина**:
   - Временное хранение выбранных автомобилей перед оформлением заказа.  
   - Реализована через таблицу `CartItem` для связи *многие ко многим* между `Cart` и `Car`.  

7. **Система купонов и скидок**:
   - Купоны на фиксированную скидку.  
   - Проверка срока действия купона и минимальной суммы заказа.  
   - Связь пользователей и купонов через `UserCoupon` (M:N).  

8. **Отзывы**:
   - Пользователь может оставить отзыв о купленном автомобиле.  
   - Оценка (1–5) и комментарий.  

---

## Описание таблиц

### 1. User (Пользователь)
**Назначение**: хранение информации о клиентах, менеджерах и администраторах.  
**Поля**:
- user_id: int (PK)  
- name: string  
- email: string (уникальный)  
- phone: string  
- role: enum (customer, manager, admin)  
- profile_picture: string (nullable)  

**Связи**:
- 1:N с `Address`, `Order`, `Review`, `Cart`, `UserCoupon`.  

---

### 2. Address (Адрес)
**Назначение**: хранение адресов доставки пользователей.  
**Поля**:
- address_id: int (PK)  
- user_id: int (FK → User)  
- city: string  
- street: string  
- house: string  
- apartment: string (nullable)  
- is_default: bool  

**Связи**:
- N:1 с `User`.  

---

### 3. Brand (Бренд)
**Назначение**: справочник брендов автомобилей.  
**Поля**:
- brand_id: int (PK)  
- name: string  
- country: string  

**Связи**:
- 1:N с `Model`.  

---

### 4. Model (Модель)
**Назначение**: справочник моделей автомобилей.  
**Поля**:
- model_id: int (PK)  
- brand_id: int (FK → Brand)  
- name: string  
- body_type: string  
- engine_type: string  

**Связи**:
- N:1 с `Brand`.  
- 1:N с `Car`.  

---

### 5. Supplier (Поставщик)
**Назначение**: хранение информации о поставщиках/дилерах автомобилей.  
**Поля**:
- supplier_id: int (PK)  
- name: string  
- contact_info: string  
- rating: decimal  

**Связи**:
- 1:N с `Car`.  

---

### 6. Car (Автомобиль)
**Назначение**: конкретный автомобиль, выставленный на продажу.  
**Поля**:
- car_id: int (PK)  
- model_id: int (FK → Model)  
- supplier_id: int (FK → Supplier)  
- year: int  
- color: string  
- price: decimal  
- mileage: int (nullable)  
- status: enum (available, reserved, sold)  

**Связи**:
- N:1 с `Model`, `Supplier`.  
- M:N с `Cart` через `CartItem`.  
- M:N с `Order` через `OrderItem`.  
- 1:N с `Review`.  

---

### 7. Cart (Корзина)
**Назначение**: временное хранилище выбранных автомобилей перед заказом.  
**Поля**:
- cart_id: int (PK)  
- user_id: int (FK → User)  
- created_at: datetime  

**Связи**:
- N:1 с `User`.  
- M:N с `Car` через `CartItem`.  
- 1:N с `Order`.  

---

### 8. CartItem (Связующая таблица Cart–Car)
**Назначение**: реализация связи *многие ко многим* между корзиной и автомобилями.  
**Поля**:
- cart_item_id: int (PK)  
- cart_id: int (FK → Cart)  
- car_id: int (FK → Car)  
- quantity: int  
- added_at: datetime  

**Связи**:
- N:1 с `Cart` и `Car`.  

---

### 9. Coupon (Купон)
**Назначение**: предоставление скидки на заказ.  
**Поля**:
- coupon_id: int (PK)  
- code: string (уникальный)  
- discount_amount: decimal  
- min_order_amount: decimal  
- valid_from: datetime  
- valid_until: datetime  
- is_active: bool  

**Связи**:
- 1:N с `Order`.  
- M:N с `User` через `UserCoupon`.  

---

### 10. UserCoupon (Связующая таблица User–Coupon)
**Назначение**: хранение данных о купонах, использованных пользователями.  
**Поля**:
- user_coupon_id: int (PK)  
- user_id: int (FK → User)  
- coupon_id: int (FK → Coupon)  
- used_at: datetime (nullable)  

**Связи**:
- N:1 с `User` и `Coupon`.  

---

### 11. Order (Заказ)
**Назначение**: оформление заказа.  
**Поля**:
- order_id: int (PK)  
- user_id: int (FK → User)  
- cart_id: int (FK → Cart, nullable)  
- coupon_id: int (FK → Coupon, nullable)  
- total_amount: decimal  
- status: enum (pending, paid, delivered, cancelled)  
- created_at: datetime  

**Связи**:
- N:1 с `User`, `Coupon`, `Cart`.  
- M:N с `Car` через `OrderItem`.  

---

### 12. OrderItem (Связующая таблица Order–Car)
**Назначение**: реализация связи *многие ко многим* между заказом и автомобилями.  
**Поля**:
- order_item_id: int (PK)  
- order_id: int (FK → Order)  
- car_id: int (FK → Car)  
- price: decimal  

**Связи**:
- N:1 с `Order` и `Car`.  

---

### 13. Review (Отзыв)
**Назначение**: отзывы пользователей об автомобилях.  
**Поля**:
- review_id: int (PK)  
- user_id: int (FK → User)  
- car_id: int (FK → Car)  
- rating: int (1–5)  
- comment: string  
- created_at: datetime  

**Связи**:
- N:1 с `User` и `Car`.  

---

## Структура связей (итог)

| Связь | Тип |
|-------|------|
| User – Address | 1:N |
| User – Cart | 1:N |
| User – Order | 1:N |
| User – Review | 1:N |
| User – Coupon | M:N через UserCoupon |
| Brand – Model | 1:N |
| Model – Car | 1:N |
| Supplier – Car | 1:N |
| Cart – Car | M:N через CartItem |
| Order – Car | M:N через OrderItem |
| Coupon – Order | 1:N |
