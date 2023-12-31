# **Feed V2 JS basics**
## *INTRODUCTION AND EXPLANATION*
### *INTRODUCTION*
The Hello Retail feed reader version 2 is the primary way for Hello Retail to receive, manipulate, and index a customers product data to later utilize said data in our solutions.

Once Hello Retail are in possession of a product feed that allows us to read the customers product data, the process of ensuring that the necessary data is stored in our database, begins.

The Hello Retail feed reader version 2 provides Hello Retail (and the customer) with an overview of the [raw source data](https://explain.helloretail.com/z8udz2x5) that was extracted from the customers product feed, which is displayed as JSON data.

Through the raw source data that are presented to us in the Hello Retail feed reader version 2, it is possible to start mapping the desired properties that we want to store on each product in our database, so that these properties can be utilized in our onsite solutions.

### *EXPLANATION*
By default, the Hello Retail feed reader version 2 is built to automatically map most of the properties that are found in the customers product feed data. This is done in order to reduce the time spent on mapping the properties in the product feed, as it is assumed that most of the provided data is going to be used.

The [JavaScript spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax), followed by the [root product parameter name](https://explain.helloretail.com/o0uK2zPd) *(...product)*, ensures that the Hello Retail feed reader version 2 starts automatically mapping the properties that it can.

While the Hello Retail feed reader version 2 is capable of mapping most properties automatically through the aforementioned functionality, there are situations in which the Hello Retail feed reader version 2 is not capable of automatically mapping properties.

To fully understand *why* certain properties are not automatically mapped, it becomes necessary to talk about JavaScript data types, and how some of these data types cannot be stored in the Hello Retail product database. This is, however, not crucial to understand the following parts of this guide, and so we will omit this for now. You can reach out to Mikkel for an indepth explanation if interested.

In this guide we will focus on mapping the properties that the Hello Retail feed reader version 2 does not automatically map.

This concludes the Introduction and Explanation segment.

## *Mapping properties in Feed v2*

### *Mapping a direct child property*

In this example data, the product url is delivered to us as *productLink*. Feed v2 might not recognize that *productLink* is in fact the url, and so it might not automatically map this property to our system. We are therefore going to manually map this property to url in our system.
```js
	{
		"type": "product_page",
		"id": "109560",
		"sku": "700179392",
		"productLink": "https://bolist-shop.5dev.se/produkt/snabbmaskering",

	}
```
Mapping *productLink* to url, from product data.
```js
function transform(product:any): TransformationResult {
	return {
		...product, /* feed v2 auto mapper.*/
		url: product.productLink /* manually mapped url. */
	};
}
```

### *Mapping a property that is nested within an object*
In this example data, the product prices are delivered to us in an object that exists at the root of the product data. The feed v2 auto mapper won't be able to read these prices due to the data structure. Furthermore, the feed v2 auto mapper most likely would not understand that *defaultPrice* should be mapped to our price variable, because the name is not similar enough to "price".

```js
	{
		"type": "product_page",
		"id": "109560",
		"sku": "700179392",
		"productLink": "https://bolist-shop.5dev.se/produkt/snabbmaskering",
		"prices":{
			"defaultPrice": 200,
			"originalPrice": 180,
		}

	}
```
Mapping *defaultPrice* to price, from product data.

Mapping *originalPrice* to oldPrice, from product data.
```js
function transform(product:any): TransformationResult {
	return {
		...product, /* feed v2 auto mapper.*/
		price: product.prices.defaultPrice, /* we mention "prices" before defaultPrice, because defaultPrice exists within the "prices" object in the product data. */
		oldPrice: product.prices.originalPrice
	};
}
```

### *Mapping a property as an extraData*
In this example data, the customer has provided us with a SKU value. SKU exists at the *root* of the product object, and so we can map it like we did with the *direct child property* in the very first example.

```js
	{
		"type": "product_page",
		"id": "109560",
		"sku": "700179392", /* String */
		"productLink": "https://bolist-shop.5dev.se/produkt/snabbmaskering",
		"prices":{
			"defaultPrice": 200,
			"originalPrice": 180,
		}
	}
```
SKU is not a native variable to Hello Retail, and so we cannot map it directly in the *return object* in the feed.

In order to map non-native variables to our database, we define a "container" named extraData to contain the mapped non-native variables.

Note that the value of SKU is a string. This will become important in the next example.
```js
function transform(product:any): TransformationResult {
	return {
		...product, /* feed v2 auto mapper.*/
		extraData:{
			sku: product.sku
		}
	};
}
```

### *Mapping a property as an extraDataList*
In this example data, the customer has provided us with a SKU value, just like in the previous example. This time, however, the SKU is provided as an Array containing a String.

```js
	{
		"type": "product_page",
		"id": "109560",
		"sku": ["700179392"], /* Array */
		"productLink": "https://bolist-shop.5dev.se/produkt/snabbmaskering",
		"prices":{
			"defaultPrice": 200,
			"originalPrice": 180,
		}
	}
```
This is where the importance of data types in JavaScript, and how our system handles data types, become relevant.

Because the SKU that the customer provided is an Array, it is no longer possible for us to assign it to ***extraData***. We must instead assign it to an ***extraDataList***.

Properties that contain Strings, Numbers, or Booleans, must be assigned as ***extraData***.

Properties that contain Arrays must be assigned as ***extraDataList***.
```js
function transform(product:any): TransformationResult {
	return {
		...product, /* feed v2 auto mapper.*/
		extraDataList:{
			sku: product.sku
		}
	};
}
```

### *Mapping a property from an array*
In this example data, the customer has provided us with an array of product sizes, named *productSizes*.

The customer has told us that the first value of the array will always contain the smallest product size.

The customer has asked us to make an extraData that contains only the smallest size that a product is available in.

```js
	{
		"type": "product_page",
		"id": "109560",
		"sku": ["700179392"],
		"productLink": "https://bolist-shop.5dev.se/produkt/snabbmaskering",
		"prices":{
			"defaultPrice": 200,
			"originalPrice": 180,
		},
		"productSizes": [34,36,38,40,42,44]
	}
```
JavaScript arrays consist of values that are comma separated within the array. Each comma separated value represents an individual position in the array.

We are able to select a specific position of the array by the infering the "position number" of the value we want to extract.

Note that position numbers (Array indexes) are zero based.
```js
function transform(product:any): TransformationResult {
	return {
		...product, /* feed v2 auto mapper.*/
		extraData: {
			productSizes: product.productSizes[0] /* selecting 34 from the productSizes array. */
		}
		
	};
}
```

### *Mapping a property from an array of objects (such as product variants)*
In this example data, the customer has provided us with an array of product variants, named *productVariants*.

*productVariants* contain 2 comma separated objects (product variants), meaning that this array has 2 "position numbers", starting from 0.

Furthermore, each of the objects contain a title and a size of the respective product variant.

The customer has asked us to make an extraData that contain the title of the last product variant.

```js
	{
		/* data from previous examples... */
		"productVariants": [
			{
				"title":"Green shoes",
				"size": 34
			},
			{
				"title":"Red shoes",
				"size": 42
			}
		]
	}
```
Selecting the title "Red shoes" from the last product variant in the array.

Note that position numbers (Array indexes) are zero based.
```js
function transform(product:any): TransformationResult {
	return {
		...product, /* feed v2 auto mapper.*/
		extraData: {
			variantTitle: product.productVariants[1].title
		}
	};
}
```

### *Mapping a property that has special characters or spaces*
In this example data, the customer has provided us with an attributes object containing properties with spaces and language specific letters.

This requires a specific approach when attempting to map.
```js
	{
		/* data from previous examples... */
		"attributes":{
			"sko mærke": "Adidas",
		}
	}
```
Selecting "sko mærke" so that the spaces and language specific letters are not causing issues.

This can look very similar to selecting values from an array, but should not be confused.
```js
function transform(product:any): TransformationResult {
	return {
		...product, /* feed v2 auto mapper.*/
		extraData: {
			shoeBrand: product.attributes["sko mærke"] /* square bracket notation is an alternative way of traversing objects. */
		}
	};
}
```





