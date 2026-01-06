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
1. create assets/product-form.js
2. modify main-product.liquid

```liquid
{% if settings.cart_type == 'ajax' %}
	<script src="{{ 'product-form.js' | asset_url }}" defer></script>
{% endif %}
```
**cretae custom element & add event listener to addtocart btn**
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
		console.log('clicked')
	}
}

if (!customElements.get('product-form')) {
	customElements.define('product-form', ProductForm)
}
```

**get form data make API request**
```js
onSubmitHandler(e) {
	...
	const formData = new FormData(this.form)

	fetch(`${window.Shopify.routes.root}cart/add.js`, {
		method: 'POST',
		body: formData
	})
		.then(response => {
			console.log(response)
			return response.json()
		})
		.then(data => {
			console.log(data)
		})
		.catch(error => {
			console.error('AJAX Cart Error:', error)
		})

}
```

**cart count on header**
***-- modify header.liquid***
```liquid
<span class="translate-middle badge rounded-pill bg-danger cart-count">
	{{ cart.item_count }}
</span>
```

**update cart count: implement bundled section rendering**
***-- sections/cart-icon-bubble.liquid***
```liquid
<span>
	{% if cart == empty %}
		0
	{% else %}
		{{ cart.item_count }}
	{% endif %}
</span>
```
***-- append sections to the formdata***
```js
	...
	formData.append('sections', 'cart-drawer,cart-icon-bubble')
	...
```
***-- dispatch new custom event: cart:updated***
```js
if (data.sections) {
	document.dispatchEvent(
		new CustomEvent('cart:updated', {
			detail: {
				sections: data.sections
			}
		})
	)
}
```
***-- listen the event and update cart: global.js***
```js
document.addEventListener('cart:updated', e => {
	const sections = e.detail.sections
	console.log(sections)
})
```
***-- parsed content***
```js
const parsedHTML = new DOMParser().parseFromString(sections['cart-icon-bubble'], 'text/html')
const parsedContent = parsedHTML.querySelector('.shopify-section')
console.log(parsedContent.innerHTML)
```
**-- finally update cart count**
```js
const target = document.querySelector('.cart-count')
if (target) {
	target.innerHTML = parsedContent.innerHTML
}
```

**cart drawer**
***-- sections/cart-drawer***
```html
<cart-drwer>
	<div class="offcanvas offcanvas-end" tabindex="-1" id="cartDrawer" aria-labelledby="cartDrawerLabel">
		<div class="offcanvas-header">
			<h5 class="offcanvas-id" id="cartDrawerLabel">Your Cart</h5>
			<button type="button" class="btn-close text-reset" data-bs-dismiss="offcanvas" aria-label="Close"></button>
		</div>
		<div class="offcanvas-body">
		cart items
		</div>
	</div>
</cart-drwer>
```
***-- render at layout/theme.liquid & see***
```liquid
{% section 'cart-drawer' %}
```

***-- all cart items: snippets/cart-drawer***
```liquid
<div id="cart-items">
	{% if cart.item_count > 0 %}
		<form action="{{ routes.cart_url }}" method="post">
			<table class="table table-striped">
				<tbody>
					{% for item in cart.items %}
						<tr>
							<td>
								<img
									src="{{ item.image.src | image_url: width: 50 }}"
									alt="{{ item.image.alt }}"
									width="auto"
									height="auto"
									loading="lazy"
								>
							</td>
							<td>
								<h6>{{ item.product.title }}</h6>
								<p>{{ item.original_price | money }}</p>
								<div>
									{% if item.product.has_only_default_variant == false %}
										{% for option in item.options_with_values %}
											<p class="m-0">{{ option.name }}: {{ option.value }}</p>
										{% endfor %}
									{% endif %}
								</div>
								<p>
									Quantity:
									<input type="number" name="updates[]" value="{{ item.quantity }}">
								</p>
							</td>
							<td>
								<strong>
									{{ item.original_line_price | money }}
								</strong>
							</td>
						</tr>
					{% endfor %}
				</tbody>
			</table>
		</form>
	{% else %}
		<h2>Cart is Empty</h2>
	{% endif %}
</div>
<div class="cart-drawer-footer border-top pt-3 mt-3">
	<div class="d-flex justify-content-between mb-3">
		<strong>Total:</strong>
		<span id="cart-drawer-total">{{ cart.total_price | money }}</span>
	</div>
	<div class="d-grid gap-2">
		<a href="{{ routes.cart_url }}" class="btn btn-outline-primary">View Cart</a>
		<form action="{{ routes.cart_url }}" method="post">
			<button type="submit" name="checkout" class="btn btn-primary w-100">Checkout</button>
		</form>
	</div>
</div>
```

***-- modify sections/cart-drawer.liquid***
```liquid
...
<div class="offcanvas-body">
	{% render 'cart-drawer' %}
</div>
...
```

**Open cart drawer: after ajax request & btn click**
***-- modify global.js***
```js
document.addEventListener('cart:updated', e => {
	...
	openCartDrawer()
})
function openCartDrawer() {
	const cartDrawer = document.getElementById('cartDrawer')
	if (cartDrawer && typeof bootstrap !== 'undefined') {
		const bsOffcanvas = bootstrap.Offcanvas.getOrCreateInstance(cartDrawer)
		bsOffcanvas.show()
	}
}
```
***-- modify header.liquid**
```liquid
<a
	class="nav-link position-relative {% if settings.cart_type == 'ajax' %}js-open-cart-drawer{% endif %}"
	href="{{ routes.cart_url }}"
>
```
***-- modify global.js***
```js
document.querySelector('.js-open-cart-drawer').addEventListener('click', e => {
	e.preventDefault()
	openCartDrawer()
})
```

**update cart drawer content: bundled section rendering**
***-- modify product-form.js***
```js
formData.append('sections', 'cart-drawer,cart-icon-bubble')
```
***-- refactor cart:updated event at global.js***
```js
Object.keys(sections).forEach(sectionId => {
	const target = document.getElementById(sectionId) || document.getElementById(`shopify-section-${sectionId}`)
	if (target) {
		console.log(`Updating section: ${sectionId}`)
		const parsedHTML = new DOMParser().parseFromString(sections[sectionId], 'text/html')
		const parsedContent = parsedHTML.querySelector('.shopify-section')
		if (parsedContent) {
			target.innerHTML = content.innerHTML
		} else {
			// Fallback if shopify-section wrapper isn't found
			target.innerHTML = sections[sectionId]
		}
	} else {
		console.warn(`Target not found for section: ${sectionId}`)
	}
})
```
***-- modify header.liquid***
```html
<span id="cart-icon-bubble" class="translate-middle badge rounded-pill bg-danger cart-count">
```

**fix maximum quantity error: add request headers**
```js
...
headers: {
 	'X-Requested-With': 'XMLHttpRequest'
}
...
```

**handle error message**
***--modify main-product.liquid***
```html
<div id="product-form-error" class="text-danger mt-2 d-none"></div>
```
***--create handleErrorMessage method: ajax-cart.liquid***
```js
this.errorContainer = this.querySelector('#product-form-error')

handleErrorMessage(message) {
	if (!this.errorContainer) return
	
	this.errorContainer.textContent = message
	this.errorContainer.classList.remove('d-none')
}

if (data.status === 422) {
	this.handleErrorMessage(data.description || data.message)
	return
}
```

***-- clear error message: handleErrorMessage(message)***
```js
setTimeout(() => {
	if (this.errorContainer) {
		this.errorContainer.classList.add('d-none')
		this.errorContainer.textContent = ''
	}
}, 3000)
```

## Resources
* [add.js API](https://shopify.dev/docs/api/ajax/reference/cart#post--locale-cart-addjs)
* [FormData constructor](https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData)
* [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
