# OBundle Test
[Store Preview](https://obundle-es.mybigcommerce.com/).

Preview Code: vbd6232eqb

## Tasks

* Create a product called Special Item which will be assigned to a new category called Special Items. Be sure to add at least 2 images during the product creation
* The Special Item should be the only item which shows in this category - create a feature that will show the product's second image when it is hovered on.
* Add a button at the top of the category page labeled Add All To Cart. When clicked, the product will be added to the cart. Notify the user that the product has been added.
* If the cart has an item in it - show a button next to the Add All To Cart button which says Remove All Items. When clicked it should clear the cart and notify the user.

## Bonus
If a customer is logged in - at the top of the category page show a banner that shows some customer details (i.e. name, email, phone, etc). This should utilize the data that is rendered via Handlebars on the Customer Object.


## Implementation
Occasionally you may need to use dynamic data from the template context within your client-side theme application code.

### Task 1
Completed at the storefront dashboard

### Task 2
Loads the default image and secondary image into separate data attributes using Handlebars helpers.<br/>
#### /templates/components/products/card.html 
```
<img class="card-image lazyload"
    data-sizes="auto"
    src="{{cdn 'img/loading.svg'}}"
    alt="{{image.alt}}" 
    title="{{image.alt}}"
    data-src="{{getImage image}}"
    data-hoverimage="{{#replace '{:size}' images.1.data}}500x659{{/replace}}{{getImageSrcset images.[1] img size (cdn default) use_default_sizes=true}}"
/>
```
New function for the hover effect with an event listener.<br/>
#### /assets/js/theme/category.js
```
hoverEffect() {
    const cards = document.querySelectorAll('.card-figure');
    cards.forEach(card => {
        const cardImage = card.querySelector('.card-image');
        card.addEventListener('mouseenter', () => {
            cardImage.src = cardImage.dataset.hoverimage;
        });
        card.addEventListener('mouseleave', () => {
            cardImage.src = cardImage.dataset.src;
        });
    });
}
```
### Task 3 & 4
Adding a button at the top of the category page labeled Add All To Cart.<br/>
#### /templates/pages/category.html 
```
<div class="category-cart-buttons">
    <button class="button button--primary" id="addAllToCart">Add All To Cart</button>    
    <button class="button button-hidden" id="removeAllFromCart">Remove All Items</button>
</div>
<p class="all-items-button-message"></p>
```
#### ./assets/js/theme/category.js
```
    createLineItems() {
        const products = document.querySelectorAll('.card');
        const cartItems = { lineItems: [] };
        products.forEach(product => {
            const lineItem = {
                quantity: 1,
                productId: parseInt(product.dataset.id),
            };
            cartItems.lineItems.push(lineItem);
        });
        return cartItems;
    }

    async createCart(url, cartItems) {
        try {
            const response = await fetch(url, {
                method: 'POST',
                credentials: 'same-origin',
                headers: {
                    Accept: 'application/json',
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(cartItems),
            });
            return await response.json();
        } catch (error) {
            // eslint-disable-next-line no-console
            console.error(error);
        }
    }

    async addItemsToCart(url, cartId, cartItems) {
        try {
            const response = await fetch(`${url}/${cartId}/items`, {
                method: 'POST',
                credentials: 'same-origin',
                headers: {
                    Accept: 'application/json',
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(cartItems),
            });
            return await response.json();
        } catch (error) {
            // eslint-disable-next-line no-console
            console.error(error);
        }
    }

    toggleRemoveAllFromCart(bool) {
        const removeAllButton = document.querySelector('#removeAllFromCart');
        if (bool) {
            removeAllButton.classList.remove('button-hidden');
        } else {
            removeAllButton.classList.add('button-hidden');
        }
    }

    showMessage(message) {
        const paragraph = document.querySelector('.all-items-button-message');
        paragraph.innerText = message;
        paragraph.style.opacity = 1;
        setTimeout(() => {
            paragraph.style.opacity = 0;
        }, 10000);
    }

    addAllToCart() {
        const addAllButton = document.querySelector('#addAllToCart');
        addAllButton.addEventListener('click', async () => {
            const cartItems = this.createLineItems();
            if (!this.cartId) {
                const createdCart = await this.createCart('/api/storefront/carts', cartItems);
                this.cartId = createdCart.id;
                this.toggleRemoveAllFromCart(true);
            } else {
                await this.addItemsToCart('/api/storefront/carts', this.cartId, cartItems);
            }
            this.showMessage('Added all items in this category to cart.');
            cartPreview(this.context.secureBaseUrl, this.cartId);
        });
    }
```
Implementing Remove All Items button<br/>
#### ./assets/js/theme/category.js
```
    async deleteCart(url, cartId) {
        try {
            await fetch(`${url}/${cartId}`, {
                method: 'DELETE',
                credentials: 'same-origin',
                headers: {
                    'Content-Type': 'application/json',
                },
            });
            return;
        } catch (error) {
            // eslint-disable-next-line no-console
            console.error(error);
        }
    }

    removeAllFromCart() {
        const removeAllButton = document.querySelector('#removeAllFromCart');
        if (removeAllButton) {
            removeAllButton.addEventListener('click', () => {
                this.deleteCart('/api/storefront/carts', this.cartId);
                this.cartId = null;
                this.showMessage('The cart has been emptied');
                cartPreview(this.context.secureBaseUrl, this.cartId);
                this.toggleRemoveAllFromCart(false);
            });
        }
    }
```
### Bonus
Showing information of a logged customer<br/>
#### /templates/pages/category.html
```
    {{#if customer}}
        <div>
            <p>Customer Name: {{customer.name}}<br/>
            Customer Email: {{customer.email}}</p>
            {{#if customer.phone}}
                <p>Customer Phone: {{customer.phone}}</p>
            {{/if}}
        </div>
    {{/if}}
```





