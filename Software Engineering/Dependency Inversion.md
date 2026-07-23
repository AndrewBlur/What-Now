**High-level modules should not depend on low-level modules. Both should depend on abstractions.**

**Abstractions should not depend on details. Details should depend on abstractions.**

# Example 1: Payment System

## ❌ Without DIP

```java
class CreditCardPayment {
    void pay(double amount) {
        System.out.println("Paid using Credit Card");
    }
}

class Checkout {
    private CreditCardPayment payment = new CreditCardPayment();

    void checkout(double amount) {
        payment.pay(amount);
    }
}
```

Problem:

- Checkout only works with credit cards.
- To add UPI or PayPal, you must modify `Checkout`.

---

## ✅ With DIP

```java
interface PaymentMethod {
    void pay(double amount);
}
```

```java
class CreditCardPayment implements PaymentMethod {
    public void pay(double amount) {
        System.out.println("Paid using Credit Card");
    }
}

class UPIPayment implements PaymentMethod {
    public void pay(double amount) {
        System.out.println("Paid using UPI");
    }
}

class PayPalPayment implements PaymentMethod {
    public void pay(double amount) {
        System.out.println("Paid using PayPal");
    }
}
```

```java
class Checkout {
    private PaymentMethod paymentMethod;

    Checkout(PaymentMethod paymentMethod) {
        this.paymentMethod = paymentMethod;
    }

    void checkout(double amount) {
        paymentMethod.pay(amount);
    }
}
```

Usage

```java
Checkout c1 = new Checkout(new CreditCardPayment());
Checkout c2 = new Checkout(new UPIPayment());
Checkout c3 = new Checkout(new PayPalPayment());

c1.checkout(100);
c2.checkout(500);
c3.checkout(1000);
```

Notice that `Checkout` never changes.