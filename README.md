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

## Quantity Error:
* If available **quantity exceeds** only automatically **adds the maximum available quantity**
* Excess quantity is **ignored**
* No error is thrown; cart reflects **in-stock limit**
  * exceeds quantity
  * entirely sold out
  * not sold out, but all of its stock is in the cart

## Steps
### Prepare data
#### Using Object
```js
let formData = {
 'items': [{
  'id': 36110175633573,
  'quantity': 2
  }]
};
```
or
#### Using FormData constructor
```js
let addToCartForm = document.querySelector('form[action$="/cart/add"]');
let formData = new FormData(addToCartForm);
```

### Make request
```js
fetch(window.Shopify.routes.root + 'cart/add.js', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(formData)
})
.then(response => {
  return response.json();
})
.catch((error) => {
  console.error('Error:', error);
});
```
or
```js
fetch(window.Shopify.routes.root + 'cart/add.js', {
  method: 'POST',
  body: formData
})
.then(response => {
  return response.json();
})
.catch((error) => {
  console.error('Error:', error);
});
```
