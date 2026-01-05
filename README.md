# AJAX Cart API: add

## Key points:

* Endpoint: **POST /{locale}/cart/add.js**
* Supports adding **one or multiple variants** at the same time.
* Each variant is passed as an **object** inside the <code>items</code> **array**.
* <code>id</code> = **variant ID**.
* <code>quantity</code> = number of units to add
* Multiple variants can be added by **adding more objects** to <code>items</code>
* Request can be made using the **Fetch API**.
* Send payload as **JSON**
* Set header <code>Content-Type: application/json</code>.
* Response returns **added cart line items as JSON**
* If the item already exists, **quantity is updated to the new total**
* Can add **line item properties** & **selling plan**

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

### settings_schema.json
```json
{
  "name": "Cart",
  "settings": [
    {
      "type": "select",
      "id": "cart_type",
      "label": "Cart Type",
      "options": [
         {
             "value": "page",
             "label": "Page"
         },
         {
             "value": "ajax",
             "label": "AJAX"
         }
      ],
      "default": "page"
    }
  ]
}
```

### modify add_to_cart btn
``` liquid
{% if settings.cart_type == 'ajax' %}
  type="button"
{% else %}
  type="submit"
{% endif %}
```

### implement AJAX API
1. create assets/ajax-cart.js
2. modify main-product.liquid

```liquid
{% if settings.cart_type == 'ajax' %}
	<script src="{{ 'ajax-cart.js' | asset_url }}" defer></script>
{% endif %}
```
  
```js
class ProductForm extends HTMLElement {
	constructor() {
		super()
		this.form = this.querySelector('form')
		this.addButton = this.querySelector('#add-to-cart-btn')
		this.handleSubmit = this.onSubmitHandler.bind(this)
	}

	connectedCallback() {
		if (!this.form) return
		this.addButton.addEventListener('click', this.handleSubmit)
	}

	disconnectedCallback() {
		if (!this.form) return
		this.addButton.removeEventListener('click', this.handleSubmit)
	}

	onSubmitHandler(e) {
		e.preventDefault()
		const formData = new FormData(this.form)
		fetch(`${window.Shopify.routes.root}cart/add.js`, {
			method: 'POST',
			body: formData
		})
			.then(response => {
				return response.json()
			})
			.then(data => {
				console.log('AJAX Cart Success:', data)
			})
			.catch(error => {
				console.error('AJAX Cart Error:', error)
			})
	}
}

if (!customElements.get('product-form')) {
	customElements.define('product-form', ProductForm)
}
```

## Resources
* [add.js API](https://shopify.dev/docs/api/ajax/reference/cart#post--locale-cart-addjs)
* [FormData constructor](https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData)
* [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
