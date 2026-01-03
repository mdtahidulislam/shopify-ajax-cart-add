# AJAX Cart API: add

## Key points:

* Endpoint: **POST /{locale}/cart/add.js**
* Supports adding **one or multiple variants** at the same time.
* Each variant is passed as an **object** inside the <code>items</code> **array**.
* <code>id</code> = **variant ID**.
* <code>quantity</code> = number of units to add
* Multiple variants can be added by **adding more objects"" to <code>items</code>
* Request can be made using the **Fetch API**.
* Send payload as **JSON**
* Set header <code>Content-Type: application/json</code>.
* Response returns **added cart line items as JSON**
* If the item already exists, **quantity is updated to the new total**
