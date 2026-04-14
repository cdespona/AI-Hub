---
description: 'Domain-Driven Design (DDD) modeling guidelines for building expressive, maintainable domain layers.'
applyTo: '**'
---

# ✅ Domain Modeling (DDD)

## Core Principle

Domain-Driven Design emphasizes that **business logic must be expressed in the domain model**, not scattered across service classes or infrastructure layers. The domain layer should be a precise reflection of the business domain, free from framework contamination.

---

## Building Blocks

### Aggregate Roots
- **Definition**: Top-level entities that own other entities and value objects. They enforce consistency boundaries and invariants.
- **Responsibility**: Contain and encapsulate all business logic related to the aggregate.
- **Invariants**: Aggregate Roots must validate and enforce all business rules before state changes.
- **Identity**: Have a persistent identity that doesn't change over the aggregate's lifetime.

**Example Pattern:**
```kotlin
// Good: Aggregate Root with embedded logic
data class Order(
    val id: OrderId,
    val items: List<OrderItem>,
    val customer: CustomerId,
    val status: OrderStatus
) {
    fun addItem(item: OrderItem) {
        require(status == OrderStatus.DRAFT) { "Cannot add items to a confirmed order" }
        // Business logic lives here
    }
}

// Bad: Anemic aggregate passed to a service
class Order(var id: OrderId, var items: MutableList<OrderItem>, ...)
class OrderService {
    fun addItem(order: Order, item: OrderItem) { // Wrong place for logic
        order.items.add(item)
    }
}
```

### Entities
- **Definition**: Objects with unique identity that persists over time.
- **Identity**: Equality is determined by ID, not by value.
- **Uniqueness**: Two entities with the same values are different if they have different IDs.
- **Mutability**: May be mutable, but changes are tracked and enforced by the aggregate root.

**Example Pattern:**
```kotlin
data class OrderItem(
    val id: OrderItemId,
    val productId: ProductId,
    val quantity: Int
) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other !is OrderItem) return false
        return id == other.id  // Equality by ID, not value
    }
    
    override fun hashCode(): Int = id.hashCode()
}
```

### Value Objects
- **Definition**: Objects without identity; equality is determined by their attributes.
- **Immutability**: Always immutable — create new instances instead of modifying.
- **No ID**: Do not have a unique identifier across time.
- **Interchangeability**: Two value objects with identical values are considered equal.

**Example Pattern:**
```kotlin
// Good: Immutable value object
data class Money(
    val amount: BigDecimal,
    val currency: Currency
) {
    init {
        require(amount >= BigDecimal.ZERO) { "Amount cannot be negative" }
    }
    
    fun add(other: Money): Money {
        require(currency == other.currency) { "Cannot add different currencies" }
        return Money(amount + other.amount, currency)
    }
}

// Bad: Mutable value object (defeats the purpose)
class Money(var amount: BigDecimal, var currency: Currency) {
    fun setAmount(newAmount: BigDecimal) { amount = newAmount } // Violates immutability
}
```

---

## Key Practices

### 1. Distinguish Building Blocks Clearly
Always explicitly define what you're modeling:
- Use distinct types (e.g., `UserId`, `Email`) instead of `String`.
- Apply immutability rules to value objects rigorously.
- Ensure aggregate roots own their entities and enforce invariants.

```kotlin
// Good: Explicit type distinction
data class Email(val value: String) {
    init {
        require(value.contains("@")) { "Invalid email format" }
    }
}

data class User(val id: UserId, val email: Email)

// Bad: Using primitives (loses domain semantics)
data class User(val id: String, val email: String)
```

### 2. Prevent Infrastructure Leakage
The domain layer must be **framework-agnostic**:
- No `@Entity`, `@Service`, `@Repository`, `@RestController`, etc. in the domain.
- No imports from `javax.persistence`, Spring, Quarkus, or any other framework.
- Dependencies flow inward (infrastructure depends on domain, never the reverse).

```kotlin
// Good: Pure domain
package com.example.domain

data class Customer(val id: CustomerId, val name: String) {
    fun updateName(newName: String): Customer = copy(name = newName)
}

// Bad: Framework pollution
package com.example.domain

@Entity
@Table(name = "customers")
data class Customer(
    @Id val id: CustomerId,
    @Column val name: String
)
```

### 3. Encapsulate Domain Logic
Business rules belong in the domain, not in services:

```kotlin
// Good: Logic in the aggregate
data class ShoppingCart(
    val id: CartId,
    val items: List<CartItem>,
    val customerId: CustomerId
) {
    fun applyDiscount(discount: Discount): ShoppingCart {
        require(discount.isValid()) { "Discount has expired" }
        val newTotal = calculateTotal().minus(discount.amount)
        require(newTotal >= Money.ZERO) { "Discount exceeds cart total" }
        return copy(items = items.map { it.applyDiscount(discount) })
    }
    
    private fun calculateTotal(): Money = items.fold(Money.ZERO) { acc, item ->
        acc.add(item.subtotal)
    }
}

// Bad: Logic scattered in service
class ShoppingCartService {
    fun applyDiscount(cart: ShoppingCart, discount: Discount) {
        if (!discount.isValid()) return
        val newTotal = cart.items.sumOf { it.price }
        if (newTotal < discount.amount) return
        // Spread across multiple places
    }
}
```

### 4. Use Factory Methods and Constructors
Control creation logic and enforce invariants at construction time:

```kotlin
// Good: Factory method for controlled creation
data class Product(val id: ProductId, val name: String, val price: Money) {
    companion object {
        fun create(name: String, price: Money): Product {
            require(name.isNotBlank()) { "Product name cannot be empty" }
            require(price.amount > BigDecimal.ZERO) { "Price must be positive" }
            return Product(ProductId.generate(), name, price)
        }
    }
}

// Usage
val product = Product.create("Widget", Money(BigDecimal("19.99"), Currency.USD))

// Bad: No validation at creation time
class Product(val id: ProductId, val name: String, val price: Money)
val product = Product(ProductId.generate(), "", Money.NEGATIVE) // Invalid state created
```

### 5. Enforce Invariants

Invariants are business rules that must **always be true**. They should be enforced at two critical points:
1. **During creation** (via factory methods)
2. **During state transitions** (via aggregate methods)

```kotlin
// Good: Invariants enforced at creation and transition
data class Order(
    val id: OrderId,
    val items: List<OrderItem>,
    val status: OrderStatus
) {
    companion object {
        fun create(customerId: CustomerId): Order {
            // Enforce invariants at creation time
            return Order(
                id = OrderId.generate(),
                items = emptyList(),
                status = OrderStatus.DRAFT
            )
        }
    }

    fun addItem(item: OrderItem): Order {
        // Enforce invariants during state change
        require(status == OrderStatus.DRAFT) { "Cannot add items to a confirmed order" }
        require(!items.any { it.productId == item.productId }) { "Item already in order" }
        return copy(items = items + item)
    }

    fun confirm(): Order {
        require(items.isNotEmpty()) { "Cannot confirm an empty order" }
        require(status == OrderStatus.DRAFT) { "Only draft orders can be confirmed" }
        return copy(status = OrderStatus.CONFIRMED)
    }
}

// Bad: No invariant enforcement
class Order(val items: MutableList<OrderItem>, var status: OrderStatus) {
    // Can be mutated directly, invariants not enforced
}

// Bad: Partial enforcement (only at transitions, not creation)
data class Order(val items: List<OrderItem>, val status: OrderStatus) {
    fun confirm(): Order {
        require(items.isNotEmpty()) { "Cannot confirm an empty order" }
        return copy(status = OrderStatus.CONFIRMED)
    }
    // But invalid state can be created: Order(emptyList(), OrderStatus.CONFIRMED)
}
```

**Key Points:**
- Use **private constructors** (or package-private) to force creation through factory methods
- Always validate **preconditions** before state transitions
- **Never allow invalid intermediate states** — invariants must hold at all times
- The factory method documents the **only valid way to create** an aggregate

### 6. Use Sealed Types for Variants
Model domain variants explicitly using sealed interfaces or classes:

```kotlin
// Good: Sealed type hierarchy
sealed interface PaymentMethod {
    data class CreditCard(val cardNumber: String, val expiryDate: String) : PaymentMethod
    data class BankTransfer(val accountNumber: String) : PaymentMethod
    data class PayPal(val email: String) : PaymentMethod
}

fun processPayment(method: PaymentMethod): Boolean = when (method) {
    is PaymentMethod.CreditCard -> validateCreditCard(method)
    is PaymentMethod.BankTransfer -> initiateTransfer(method)
    is PaymentMethod.PayPal -> processPayPalPayment(method)
}

// Bad: Using strings for variants (loses type safety)
data class Payment(val methodType: String, val details: String)
```

### 7. Constructor Injection in Domain Services
When domain objects need dependencies, inject them via constructors:

```kotlin
// Good: Constructor injection
class InventoryService(private val warehouse: Warehouse) {
    fun reserveStock(productId: ProductId, quantity: Int): Stock {
        return warehouse.reserve(productId, quantity)
    }
}

// Bad: Service locator or static access (hard to test and maintain)
class InventoryService {
    fun reserveStock(productId: ProductId, quantity: Int): Stock {
        return ServiceLocator.getWarehouse().reserve(productId, quantity)
    }
}
```

---

## Domain Services

Domain Services encapsulate business logic that doesn't naturally belong to a single aggregate. Use them **sparingly** and only when logic spans multiple aggregates or requires complex coordination.

### When to Use Domain Services

**Use domain services when:**
1. Business logic involves **multiple aggregates** and cannot be assigned to a single one
2. A coordination task is too complex to be handled by the aggregate root itself
3. **Kotlin only**: Heavy collection processing of aggregates → prefer **extension functions** instead

**Example Scenario:**
```kotlin
// Scenario: Transferring money between accounts
// Logic involves two aggregates (Account and Account), but belongs to neither individually
```

### Kotlin: Prefer Extension Functions

For Kotlin, prefer **extension functions** over domain services when processing collections of aggregates:

```kotlin
// Good (Kotlin): Extension function for aggregate collection processing
fun List<Order>.totalRevenue(): Money {
    return fold(Money.ZERO) { acc, order ->
        acc.add(order.calculateTotal())
    }
}

fun List<Order>.confirmedOrders(): List<Order> {
    return filter { it.status == OrderStatus.CONFIRMED }
}

fun List<Order>.groupByCustomer(): Map<CustomerId, List<Order>> {
    return groupBy { it.customer }
}

// Usage
val revenue = orders.totalRevenue()
val confirmed = orders.confirmedOrders()
```

### Java: Domain Services for Aggregate Collections

When working with Java (which lacks extension functions), use **focused domain services** for aggregate collection operations:

```java
// Good (Java): Focused domain service for aggregate operations
public class OrderCollectionService {
    public Money calculateTotalRevenue(List<Order> orders) {
        return orders.stream()
            .map(Order::calculateTotal)
            .reduce(Money.ZERO, Money::add);
    }

    public List<Order> findConfirmedOrders(List<Order> orders) {
        return orders.stream()
            .filter(order -> order.getStatus() == OrderStatus.CONFIRMED)
            .collect(toList());
    }

    public Map<CustomerId, List<Order>> groupByCustomer(List<Order> orders) {
        return orders.stream()
            .collect(groupingBy(Order::getCustomer));
    }
}

// Usage
Money revenue = collectionService.calculateTotalRevenue(orders);
List<Order> confirmed = collectionService.findConfirmedOrders(orders);
```

### Multi-Aggregate Domain Services

Use domain services **only when logic truly spans multiple aggregates** and cannot be pushed to a single one:

```kotlin
// Good: Domain service for multi-aggregate coordination
class MoneyTransferService(
    private val accountRepository: AccountRepository
) {
    fun transfer(
        fromAccountId: AccountId,
        toAccountId: AccountId,
        amount: Money
    ): Result<Unit> {
        // Retrieve both aggregates
        val fromAccount = accountRepository.findById(fromAccountId)
            ?: return Result.failure(AccountNotFound(fromAccountId))
        val toAccount = accountRepository.findById(toAccountId)
            ?: return Result.failure(AccountNotFound(toAccountId))

        // Execute business logic that spans both
        val updatedFrom = fromAccount.withdraw(amount)
            ?: return Result.failure(InsufficientFunds(fromAccount.id))
        val updatedTo = toAccount.deposit(amount)

        // Persist changes
        accountRepository.save(updatedFrom)
        accountRepository.save(updatedTo)

        return Result.success(Unit)
    }
}

// Bad: Domain service that could have been an aggregate method
class OrderTotalCalculationService {
    fun calculateTotal(order: Order): Money {
        return order.calculateTotal()  // Logic belongs in Order aggregate
    }
}
```

### Key Principles for Domain Services

- **Stateless**: Domain services should be stateless; state lives in aggregates
- **Single responsibility**: Each service handles one specific domain workflow
- **Named after behavior**: Use names like `MoneyTransferService`, not `OrderProcessor`
- **Operate on aggregates**: Work with aggregate roots, not primitive values
- **No business logic duplication**: Extract shared logic into the domain service, not services

---

## Infrastructure Services

Infrastructure Services (also called Application Services in IDDD) coordinate domain logic and infrastructure concerns. They sit at the boundary between the domain layer and the external world, translating between external requests and domain operations.

### Purpose and Responsibility

Infrastructure Services:
1. **Coordinate domain operations**: Orchestrate interactions between domain objects and repositories
2. **Handle cross-cutting concerns**: Transactions, logging, security, monitoring
3. **Translate between layers**: Convert external DTOs to domain objects and vice versa
4. **Manage persistence**: Retrieve aggregates, apply domain logic, and persist changes
5. **Enforce authorization**: Check permissions and apply security policies
6. **Manage transactions**: Define transaction boundaries for consistency

### Architectural Layers

```
┌─────────────────────────────────────────────┐
│ Interfaces (Controllers, REST, Messaging)    │
│ - HTTP endpoints, message handlers          │
│ - Convert requests to commands/queries       │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│ Application/Infrastructure Services          │
│ - Orchestrate domain operations             │
│ - Handle transactions and security          │
│ - Manage persistence lifecycle              │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│ Domain Layer                                 │
│ - Aggregates (business logic)               │
│ - Domain Services (multi-aggregate logic)   │
│ - Value Objects, Entities                   │
│ - Domain Ports (interfaces)                 │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│ Infrastructure Adapters                      │
│ - Repositories (implement domain ports)     │
│ - External service clients                  │
│ - Database access, message queues, etc.     │
└─────────────────────────────────────────────┘
```

### Application/Infrastructure Service Pattern

```kotlin
// Good: Clear separation of concerns
@Service
class PlaceOrderService(
    private val orderRepository: OrderRepository,
    private val customerRepository: CustomerRepository,
    private val inventoryService: InventoryService,
    private val transactionManager: TransactionManager
) {
    @Transactional
    fun execute(command: PlaceOrderCommand): Result<OrderId> {
        // 1. Authorization (security concern)
        val customer = customerRepository.findById(command.customerId)
            ?: return Result.failure(CustomerNotFound(command.customerId))
        
        if (!customer.canPlaceOrder()) {
            return Result.failure(CustomerBlockedError())
        }

        // 2. Retrieve and validate inventory (domain logic via service)
        val items = command.items.map { item ->
            inventoryService.reserveStock(item.productId, item.quantity)
                .getOrElse { return Result.failure(it) }
        }

        // 3. Create aggregate using factory method (domain logic)
        val order = Order.create(
            customerId = command.customerId,
            items = items,
            deliveryAddress = command.address
        )

        // 4. Apply domain logic (multiple aggregates if needed)
        val updatedOrder = order.applyCustomerDiscount(customer.loyaltyTier())

        // 5. Persist (infrastructure concern)
        val savedOrder = orderRepository.save(updatedOrder)

        // 6. Return result to interface layer
        return Result.success(savedOrder.id)
    }
}

// Bad: Mixed concerns, anemic domain
@Service
class PlaceOrderService(private val db: Database) {
    @Transactional
    fun placeOrder(customerId: String, items: List<OrderItem>): String {
        // Mixing security, domain logic, and persistence
        val customer = db.query("SELECT * FROM customers WHERE id = ?", customerId)
        if (customer.status != "ACTIVE") throw Exception("Inactive customer")
        
        val order = Order(OrderId.generate(), customerId, items, "PENDING")
        db.insert("INSERT INTO orders ...", order)
        
        return order.id
    }
}
```

### Repository Pattern

Repositories implement domain ports and provide aggregate persistence:

```kotlin
// Domain Port (defined in domain layer)
interface OrderRepository {
    fun save(order: Order): Order
    fun findById(id: OrderId): Order?
    fun findByCustomer(customerId: CustomerId): List<Order>
}

// Infrastructure Adapter (implemented in infrastructure layer)
@Repository
class JpaOrderRepository(
    private val jpaOrderRepository: SpringDataOrderRepository
) : OrderRepository {
    override fun save(order: Order): Order {
        val entity = order.toEntity()  // Domain → persistence model
        val saved = jpaOrderRepository.save(entity)
        return saved.toDomain()  // Persistence model → domain
    }

    override fun findById(id: OrderId): Order? {
        return jpaOrderRepository.findById(id.value)
            ?.toDomain()
    }

    override fun findByCustomer(customerId: CustomerId): List<Order> {
        return jpaOrderRepository.findByCustomerId(customerId.value)
            .map { it.toDomain() }
    }
}

// Good: Repositories only expose aggregates, not database details
val order = orderRepository.findById(orderId)
order?.confirm()  // Use domain logic

// Bad: Leaking query logic to service layer
val orders = db.query("SELECT * FROM orders WHERE status = 'DRAFT'")
orders.forEach { it.status = "CONFIRMED" }  // Violates invariants
db.updateAll(orders)
```

### Key Principles for Infrastructure Services

| Principle | Good | Bad |
|-----------|------|-----|
| **Single Transaction** | One infrastructure service = one transaction boundary | Multiple nested transactions; unclear boundaries |
| **Aggregate-Centric** | Load one aggregate, apply domain logic, save it | Load multiple unrelated entities; manipulate data directly |
| **No Business Logic** | Orchestrate domain operations only | Implement business rules in infrastructure service |
| **Explicit Dependencies** | Constructor-injected ports and repositories | Service locator; static access; implicit dependencies |
| **Error Handling** | Return domain errors; let domain decide validity | Catch all exceptions; hide domain errors |
| **Naming** | `PlaceOrderService`, `UpdateCustomerService` | `OrderProcessor`, `DataService`, `Helper` |

### Infrastructure Service Checklist

Before implementing an infrastructure service, ask:

- ✅ **Does it orchestrate domain operations?** (If yes → infrastructure service)
- ✅ **Does it cross aggregate boundaries?** (If yes → use domain service or infrastructure service)
- ✅ **Does it manage persistence or transactions?** (If yes → infrastructure service)
- ❌ **Does it contain business logic?** (If yes → move to domain aggregate)
- ❌ **Does it mutate multiple unrelated entities?** (If yes → violates aggregate boundaries)
- ❌ **Is it doing what a single aggregate could do?** (If yes → it shouldn't exist)


````
