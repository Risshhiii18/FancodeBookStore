# Module Explanation #
> <p>Created registration.php and module.xml file which is necessary for the Magento module registration</p>
> <p>Created "Setup/InstallData.php" for creating new Attribute and attribute set</p>
 ```
**Code Snippet for Creating new Attribute Set**
$categorySetup = $this->categorySetupFactory->create(['setup' => $setup]);
$attributeSet = $this->attributeSetFactory->create();
$entityTypeId = $categorySetup->getEntityTypeId(\Magento\Catalog\Model\Product::ENTITY);
$attributeSetId = $categorySetup->getDefaultAttributeSetId($entityTypeId);
$data = [
    'attribute_set_name' => 'Books', // new Attribute set books will be created 
    'entity_type_id' => $entityTypeId,
    'sort_order' => 100,
];
$attributeSet->setData($data);
$attributeSet->validate();
$attributeSet->save();
$attributeSet->initFromSkeleton($attributeSetId);
$attributeSet->save();
```
```
**Created Attribute Programmatically**
$eavSetup->addAttribute(
    \Magento\Catalog\Model\Product::ENTITY,
    'author',
    [
	'type' => 'varchar', // varchar so admin can add any name for the author
	'backend' => '',
	'frontend' => '',
	'label' => 'Author', // label will be visible on frontend
	'input' => 'text', 
	'class' => '',
	'source' => '',
	'global' => \Magento\Eav\Model\Entity\Attribute\ScopedAttributeInterface::SCOPE_GLOBAL, // Attribute Scope will be global so the attribute will be used in Website, Store, Store Front
	'visible' => true, 
	'required' => true, // Required will be true because the book author is required
	'user_defined' => false, 
	'default' => '',
	'searchable' => true, // as provided in the requirement the books should be searchable by the author name so this value is true.
	'filterable' => true, // Attribute can be filterable on the Category page
	'comparable' => false,
	'visible_on_front' => true,
	'used_in_product_listing' => true,
	'unique' => false,
	'apply_to' => '',
    ]
);
```
```
**Assign Custom Attribute to Attribute Set**
$eavSetup->addAttributeToGroup(
	\Magento\Catalog\Model\Product::ENTITY,
	$attributeSetId,
	'Product Details',
	'author',
	5
);
```
> Extended "view/frontend/layout/catalog_category_view.xml" File
```
// replaced the list.phtml file of "category.products.list" with our file which has the "author" attribute and "stock available" option display
    <block class="Magento\Catalog\Block\Product\ListProduct" name="category.products.list" as="product_list" template="Fancode_BookStore::list.phtml">
```
> Created "view/frontend/templates/list.phtml" File
```
// author is a product attribute so I can call the attribute via product collection which is by default provided by Magento so I have called the attribute by using getAuthor() method
// The below-following code will display the stock availability and also the author data. from PHTML file
    <?php if($_product->getAuthor() !== '' ): ?>
	<?=/* @noEscape */ $_helper->productAttribute($_product, $_product->getAuthor(), 'author')?>
    <?php endif;?>
    <?php if ($_product->isAvailable()) :?>
	<div class="stock available" title="<?= $block->escapeHtmlAttr(__('Availability')) ?>">
	    <span><?= $block->escapeHtml(__('In stock')) ?></span>
	</div>
    <?php else :?>
	<div class="stock unavailable" title="<?= $block->escapeHtmlAttr(__('Availability')) ?>">
	    <span><?= $block->escapeHtml(__('Out of stock')) ?></span>
	</div>
    <?php endif; ?>
```


> Note* Magento provide the graphql APIs for handling the listing, search, adding/removing cart, and placing order

> below I'm Simulating the Process of placing an Order from the graphql APIs
```
1) listing page
//below API will be used for handling the listing page by using filters in the query we can fetch the product data 
//Here I'm passing category id 4 which is my book's category
{
  products(filter: {category_id: {eq: "4"}}) {
    aggregations {
      attribute_code
      count
      label
      options {
	label
	value
	count
      }
    }
    items {
      name
      sku
      price_range {
	minimum_price {
	  regular_price {
	    value
	    currency
	  }
	}
      }
    }
    page_info {
      page_size
    }
  }
}
```
```
2) Search API
// search query will take a parameter as "search" and "pageSize" for the result
// here I'm searching for search value = J.K. and pagesize = 2
{
  products(search: "J.K.", pageSize: 2) {
    total_count
    items {
      name
      sku
      price_range {
	minimum_price {
	  regular_price {
	    value
	    currency
	  }
	}
      }
    }
    page_info {
      page_size
      current_page
    }
  }
}
```
```
3) before we add or remove the product from the cart we need an empty cart for the guest by the following mutation it will be created
mutation {
  createEmptyCart
}
```
```
4) passing the cart id in the following mutation we can add the product to the cart also we have to pass the qty and sku for adding the product to the cart
mutation {
  addSimpleProductsToCart(
    input: {
      cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh"
      cart_items: [
	{
	  data: {
	    quantity: 2
	    sku: "BOOK008"
	  }
	}
      ]
    }
  ) {
    cart {
      items {
	id
	product {
	  sku
	  stock_status
	}
	quantity
      }
    }
  }
}
```
```
5) Passing the cart id and item id we can remove the item from the cart
mutation {
  removeItemFromCart(
    input: {
      cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh",
      cart_item_id: 25
    }
  )
 {
  cart {
    items {
      id
      product {
	name
      }
      quantity
    }
    prices {
      grand_total{
	value
	currency
      }
    }
  }
 }
}
```
```
6) set the Shipping method on the cart
mutation {
  setShippingAddressesOnCart(
    input: {
      cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh"
      shipping_addresses: {
        address: {
          firstname: "Rushabh"
          lastname: "Chudasama"
          street: ["Malad East", "Mumbai 400097"]
          city: "Mumbai"
          region: "MH"
          postcode: "400097"
          country_code: "IN"
          telephone: "8169208230"
          save_in_address_book: true
        }
      }
    }
  ) {
    cart {
      billing_address {
        firstname
        lastname
        company
        street
        city
        region{
          code
          label
        }
        postcode
        telephone
        country {
          code
          label
        }
      }
    }
  }
}
```
```
7) Set the Billing address on the cart
mutation {
  setBillingAddressOnCart(
    input: {
      cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh"
      billing_address: {
	address: {
	  firstname: "Rushabh"
	  lastname: "Chudasama"
	  street: ["Malad East", "Mumbai 400097"]
	  city: "Mumbai"
	  region: "MH"
	  postcode: "400097"
	  country_code: "IN"
	  telephone: "8169208230"
	  save_in_address_book: true
	}
      }
    }
  ) {
    cart {
      billing_address {
	firstname
	lastname
	company
	street
	city
	region{
	  code
	  label
	}
	postcode
	telephone
	country {
	  code
	  label
	}
      }
    }
  }
}
```
```
8) Set shipping method
mutation {
  setShippingMethodsOnCart(
    input: {
      cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh"
      shipping_methods: [
	{
	  carrier_code: "flatrate"
	  method_code: "flatrate"
	}
      ]
    }
  ){
    cart{
      shipping_addresses{
	selected_shipping_method{
	  carrier_code
	  carrier_title
	  method_code
	  method_title
	  amount{
	    value
	    currency
	  }
	}
      }
    }
  }
} 
```
```
9) set email for on cart
mutation {
  setGuestEmailOnCart(input: {
    cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh"
    email: "chudasamarushabh.1811@gmail.com"
  }) {
    cart {
      email
    }
  }
}
```
```
10) show available payment methods

query {
  cart(cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh") {
    available_payment_methods {
      code
      title
    }
  }
}
```
```
11) Set Payment Method on the cart
mutation {
  setPaymentMethodOnCart(input: {
      cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh"
      payment_method: {
	  code: "cashondelivery"
      }
  }) {
    cart {
      selected_payment_method {
	code
      }
    }
  }
}
```
```
12) Place Order 

mutation {
  placeOrder(input: {cart_id: "RUNQh1VN8mAbWTJbI7MwDrLWEJSXQUwh"}) {
    order {
      order_number
    }
  }
}
```
**Above I have provided the details regarding the API and Code which I have done to solve the problem provided in the assignment as per my understanding.**

- Q) Explain the process followed for database interactions through PDO or any other preferred method for security and to avoid SQL injection vulnerabilities.
- ans) Magento Uses ORM System to interact with the database. ORM system in Magento contains 3 major files they are Model, ResourceMode, and collection. all though there are also repositories that help encapsulate data while interacting with the database. ORM System provides a secure way to work with a database to prevent any SQL injection. using Model and ResourceModel we can interact with the magento database without writing direct SQL queries. magento denies using SQL query unless it is absolutely necessary, ORM uses query builders like addFiledToFilter, and addFiledToSelect to avoid writing raw queries we can avoid SQL injection.<br>
	- **Model** -> Model is basically setters and getters, it represents a single data entry in the table, also it can be used to adding and getting data from the tables. every model is associated with a resourceModel, model forwards the request to the Resouce model ad the actual database interaction happens in resourcemodel.<br>
 	- **Resorcemode** -> Resouce model is mainly responsible for the database operations such as insertion, updation, deletion, and selection of the data.<br>
 	- **Collection** -> It is worked as a group of models to work with multiple data from the table we can apply filter sorting over the collection. and get multiple entities according to our needs.<br>
