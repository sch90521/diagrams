# diagrams
@startuml
'     ДИАГРАММА ПРЕЦЕДЕНТОВ 
left to right direction
actor "Покупатель" as Customer
actor "Администратор" as Admin
actor "Платежная система" as PaymentSystem

rectangle "Интернет-магазин" {
  usecase "Просмотр каталога" as UC1
  usecase "Поиск товаров" as UC2
  usecase "Регистрация" as UC3
  usecase "Вход в систему" as UC4
  usecase "Добавить товар в корзину" as UC5
  usecase "Оформить заказ" as UC6
  usecase "Оплатить заказ" as UC7
  usecase "Просмотр истории заказов" as UC8
  usecase "Управление товарами (CRUD)" as UC9
  usecase "Управление заказами" as UC10
  usecase "Просмотр статистики" as UC11
  usecase "Обработка платежа" as UC12
}

Customer --> UC1
Customer --> UC2
Customer --> UC3
Customer --> UC4
Customer --> UC5
Customer --> UC6
Customer --> UC8
Customer --> UC7

Admin --> UC9
Admin --> UC10
Admin --> UC11

UC7 ..> UC12 : <<include>>
UC12 <-- PaymentSystem
@enduml

@startuml
'    ДИАГРАММА КЛАССОВ 
class User {
  - id: int
  - name: string
  - email: string
  - password: string
  + login()
  + logout()
}

class Customer {
  - address: string
  - phone: string
  + addToCart()
  + makeOrder()
}

class Admin {
  + addProduct()
  + updateProduct()
  + deleteProduct()
  + manageOrders()
}

class Product {
  - id: int
  - name: string
  - price: double
  - stock: int
  - description: string
  + getInfo()
}

class Category {
  - id: int
  - name: string
  + getProducts()
}

class Cart {
  - id: int
  - createdDate: date
  + addItem()
  + removeItem()
  + calculateTotal()
}

class CartItem {
  - quantity: int
  - priceAtAdd: double
}

class Order {
  - id: int
  - orderDate: date
  - status: enum
  + calculateTotal()
  + updateStatus()
}

class OrderItem {
  - quantity: int
  - price: double
}

class Payment {
  - id: int
  - amount: double
  - status: string
  - paymentDate: date
  + processPayment()
}

User <|-- Customer
User <|-- Admin

Customer "1" -- "1" Cart
Cart "1" -- "*" CartItem
Product "1" -- "*" CartItem
Product "1" -- "*" OrderItem
Product "*" -- "*" Category

Order "1" -- "*" OrderItem
Order "1" -- "1" Customer
Order "1" -- "1" Payment
Payment "1" -- "1" Customer
@enduml

@startuml
'     ДИАГРАММА АКТИВНОСТЕЙ 
start
:Покупатель переходит в корзину;
:Нажимает "Оформить заказ";

if (Пользователь авторизован?) then (нет)
  :Предложить регистрацию/вход;
  :Авторизация;
else (да)
endif

:Заполняет данные доставки;
:Выбирает способ оплаты;

fork
  :Проверить наличие товаров на складе;
fork again
  :Рассчитать итоговую сумму;
end fork

if (Товары в наличии?) then (нет)
  :Уведомить о невозможности заказа;
  stop
else (да)
endif

:Создать заказ (статус "Ожидает оплаты");
:Зарезервировать товары;

:Перенаправить на платежный шлюз;
:Ввести данные карты;
:Обработать платеж;

if (Платеж успешен?) then (да)
  :Изменить статус заказа на "Оплачен";
  :Списать зарезервированные товары;
  :Отправить email-подтверждение покупателю;
  :Показать страницу "Заказ выполнен";
else (нет)
  :Изменить статус заказа на "Отменен";
  :Снять резерв с товаров;
  :Показать сообщение об ошибке оплаты;
endif

stop
@enduml

@startuml
'     ДИАГРАММА ПОСЛЕДОВАТЕЛЬНОСТИ 
actor "Покупатель" as Customer
participant "Веб-интерфейс" as Web
participant "Контроллер корзины" as CartController
participant "Сервис заказов" as OrderService
participant "Корзина" as Cart
participant "База данных" as DB
participant "Платежный шлюз" as PaymentGateway

Customer -> Web: Нажимает "Купить" на товаре
Web -> CartController: addToCart(productId, userId)
CartController -> Cart: addItem(productId)
Cart -> DB: getProduct(productId)
DB --> Cart: Product
Cart -> Cart: update total price
Cart --> CartController: success
CartController --> Web: OK

Customer -> Web: Нажимает "Оформить заказ"
Web -> OrderService: createOrder(userId)
OrderService -> Cart: getItems()
Cart --> OrderService: items[]
OrderService -> DB: saveOrder(items, status="NEW")
DB --> OrderService: orderId
OrderService -> PaymentGateway: processPayment(orderId, amount)

alt Payment Success
    PaymentGateway --> OrderService: success
    OrderService -> DB: updateOrderStatus(orderId, "PAID")
    OrderService --> Web: redirectToSuccess()
    Web --> Customer: "Заказ оплачен"
else Payment Failure
    PaymentGateway --> OrderService: failure
    OrderService -> DB: updateOrderStatus(orderId, "CANCELLED")
    OrderService --> Web: redirectToError()
    Web --> Customer: "Ошибка оплаты"
end
@enduml
