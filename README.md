const express = require("express");
const app = express();

app.use(express.json());

let products = [
  { id: 1, name: "Tomato", price: 30, farmer: "Ravi", location: "Trichy" },
  { id: 2, name: "Carrot", price: 50, farmer: "Suresh", location: "Madurai" },
  { id: 3, name: "Spinach", price: 20, farmer: "Muthu", location: "Salem" }
];

let orders = [];

app.get("/", (req, res) => {
  res.send(`
    <html>
    <head>
      <title>Local Farmer Connect</title>
      <style>
        body { font-family: Arial; background:#f0fff0; text-align:center; }
        h1 { color: green; }
        .product { border:1px solid #ccc; padding:10px; margin:10px; }
        button { padding:5px 10px; margin-top:5px; }
      </style>
    </head>
    <body>

    <h1>🌿 Local Farmer Connect</h1>

    <h2>Products</h2>
    <div id="products"></div>

    <h2>🛒 Cart</h2>
    <div id="cart"></div>

    <button onclick="checkout()">Checkout</button>

    <h2>📦 Orders</h2>
    <div id="orders"></div>

    <script>
      let cart = [];

      async function loadProducts() {
        const res = await fetch('/products');
        const data = await res.json();

        let html = "";
        data.forEach(p => {
          html += \`
            <div class="product">
              <h3>\${p.name}</h3>
              <p>Price: ₹\${p.price}</p>
              <p>Farmer: \${p.farmer}</p>
              <button onclick='addToCart(\${JSON.stringify(p)})'>Add to Cart</button>
            </div>
          \`;
        });

        document.getElementById("products").innerHTML = html;
      }

      function addToCart(p) {
        cart.push(p);
        displayCart();
      }

      function displayCart() {
        let total = 0;
        let html = "";

        cart.forEach(p => {
          total += p.price;
          html += \`<p>\${p.name} - ₹\${p.price}</p>\`;
        });

        html += \`<h3>Total: ₹\${total}</h3>\`;
        document.getElementById("cart").innerHTML = html;
      }

      async function checkout() {
        const res = await fetch('/order', {
          method: 'POST',
          headers: {'Content-Type':'application/json'},
          body: JSON.stringify({ items: cart })
        });

        const data = await res.json();
        alert(data.message);

        cart = [];
        displayCart();
        loadOrders();
      }

      async function loadOrders() {
        const res = await fetch('/orders');
        const data = await res.json();

        let html = "";
        data.forEach((o, i) => {
          html += \`<p>Order \${i+1}: \${o.items.length} items</p>\`;
        });

        document.getElementById("orders").innerHTML = html;
      }

      loadProducts();
      loadOrders();
    </script>

    </body>
    </html>
  `);
});

// APIs
app.get("/products", (req, res) => res.json(products));

app.post("/order", (req, res) => {
  orders.push(req.body);
  res.json({ message: "Order placed successfully!" });
});

app.get("/orders", (req, res) => res.json(orders));

// Start server
app.listen(3000, () => console.log("Server running at http://localhost:3000"));
