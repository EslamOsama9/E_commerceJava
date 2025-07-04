package com.mycompany.task_fawry;

import java.util.*;

interface Shippable {
    String getName();
    double getWeight();
}

abstract class Product {
    protected String name;
    protected double price;
    protected int quantity;

    public Product(String name, double price, int quantity) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
    }

    public String getName() { return name; }
    public double getPrice() { return price; }
    public int getQuantity() { return quantity; }

    public void reduceQuantity(int amount) {
        if (amount <= quantity) {
            quantity -= amount;
        }
    }

    public boolean isAvailable(int amount) {
        return quantity >= amount;
    }

    public boolean isExpired() {
        return false;
    }
}

class ExpirableProduct extends Product {
    private Date expiryDate;

    public ExpirableProduct(String name, double price, int quantity, Date expiryDate) {
        super(name, price, quantity);
        this.expiryDate = expiryDate;
    }

    @Override
    public boolean isExpired() {
        return new Date().after(expiryDate);
    }
}

class ShippableProduct extends Product implements Shippable {
    private double weight;

    public ShippableProduct(String name, double price, int quantity, double weight) {
        super(name, price, quantity);
        this.weight = weight;
    }

    public double getWeight() {
        return weight;
    }
}

class ExpirableShippableProduct extends ExpirableProduct implements Shippable {
    private double weight;

    public ExpirableShippableProduct(String name, double price, int quantity, Date expiryDate, double weight) {
        super(name, price, quantity, expiryDate);
        this.weight = weight;
    }

    public double getWeight() {
        return weight;
    }
}

class Customer {
    private String name;
    private double balance;

    public Customer(String name, double balance) {
        this.name = name;
        this.balance = balance;
    }

    public String getName() { return name; }
    public double getBalance() { return balance; }

    public void deductBalance(double amount) {
        balance -= amount;
    }
}

class CartItem {
    Product product;
    int quantity;

    public CartItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
}

class Cart {
    private List<CartItem> items = new ArrayList<>();

    public void add(Product product, int quantity) {
        if (!product.isAvailable(quantity)) {
            throw new RuntimeException("Not enough quantity for product: " + product.getName());
        }
        items.add(new CartItem(product, quantity));
    }

    public List<CartItem> getItems() {
        return items;
    }

    public boolean isEmpty() {
        return items.isEmpty();
    }
}

class ShippingService {
    public static void ship(List<Shippable> shippables) {
        System.out.println("** Shipping Notice **");
        double totalWeight = 0;
        Map<String, Integer> counts = new HashMap<>();

        for (Shippable item : shippables) {
            counts.put(item.getName(), counts.getOrDefault(item.getName(), 0) + 1);
            totalWeight += item.getWeight();
        }

        for (Map.Entry<String, Integer> entry : counts.entrySet()) {
            System.out.println(entry.getValue() + "x " + entry.getKey());
        }

        System.out.printf("Total weight: %.2f kg\n", totalWeight);
    }
}

public class Task_fawry {
    private static final double SHIPPING_FEE_PER_KG = 30.0;

    public static void checkout(Customer customer, Cart cart) {
        if (cart.isEmpty()) {
            System.out.println("Cart is empty.");
            return;
        }

        for (CartItem item : cart.getItems()) {
            Product p = item.product;
            if (!p.isAvailable(item.quantity)) {
                System.out.println("Product " + p.getName() + " is out of stock.");
                return;
            }
            if (p.isExpired()) {
                System.out.println("Product " + p.getName() + " is expired.");
                return;
            }
        }

        double subtotal = 0;
        double totalWeight = 0;
        List<Shippable> shippables = new ArrayList<>();

        for (CartItem item : cart.getItems()) {
            subtotal += item.product.getPrice() * item.quantity;

            if (item.product instanceof Shippable) {
                Shippable shipItem = (Shippable) item.product;
                for (int i = 0; i < item.quantity; i++) {
                    shippables.add(shipItem);
                }
                totalWeight += shipItem.getWeight() * item.quantity;
            }
        }

        double shippingCost = totalWeight * SHIPPING_FEE_PER_KG;
        double total = subtotal + shippingCost;

        if (customer.getBalance() < total) {
            System.out.println("Not enough balance.");
            return;
        }

        customer.deductBalance(total);

        for (CartItem item : cart.getItems()) {
            item.product.reduceQuantity(item.quantity);
        }

        if (!shippables.isEmpty()) {
            ShippingService.ship(shippables);
        }

        System.out.println("** Checkout Receipt **");
        Map<String, Integer> counts = new HashMap<>();
        Map<String, Double> prices = new HashMap<>();

        for (CartItem item : cart.getItems()) {
            counts.put(item.product.getName(),
                    counts.getOrDefault(item.product.getName(), 0) + item.quantity);
            prices.put(item.product.getName(), item.product.getPrice());
        }

        for (String productName : counts.keySet()) {
            System.out.println(counts.get(productName) + "x " + productName + " = " +
                    (prices.get(productName) * counts.get(productName)));
        }

        System.out.println("---------------------");
        System.out.println("Subtotal: " + subtotal);
        System.out.println("Shipping: " + shippingCost);
        System.out.println("Total: " + total);
        System.out.println("Balance after payment: " + customer.getBalance());
    }

    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        cal.add(Calendar.DAY_OF_MONTH, 1);
        Date expiry = cal.getTime();

        Product cheese = new ExpirableShippableProduct("Cheese", 100, 10, expiry, 0.4);
        Product biscuits = new ExpirableShippableProduct("Biscuits", 150, 5, expiry, 0.7);
        Product tv = new ShippableProduct("TV", 3000, 2, 8);
        Product scratchCard = new Product("Mobile Scratch Card", 50, 100) {};

        Customer customer = new Customer("Eslam", 5000);

        Cart cart = new Cart();
        cart.add(cheese, 2);
        cart.add(biscuits, 1);
        cart.add(tv, 1);
        cart.add(scratchCard, 1);

        checkout(customer, cart);
    }
}
