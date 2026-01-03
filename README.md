# AJAX Cart API: add

## Key points:

* Endpoint: **POST /{locale}/cart/add.js**
* Supports adding **one or multiple variants** at the same time.
* Each variant is passed as an **object** inside the items **array**.
* <mark>id</mark> refers to the **variant ID**.
* quantity specifies how many units of that variant to add.
* To add multiple variants, simply append more objects to the items array.
* The request can be made using the Fetch API.
* When building the payload in JavaScript, send data as JSON.
* Set the request header Content-Type: application/json.
* The response returns JSON data of the line items that were added.
* If a variant already exists in the cart, the returned quantity reflects the updated total quantity, not just the added amount.
