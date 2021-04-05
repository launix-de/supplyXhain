supplyXhain API Documentation
=============================

The supplyXhain API protocol is a REST/JSON based protocol that handles:

- providing catalog information such as prices, texts in multiple languages, configuration options
- asking for quotes and offers with prices, discounts and shipping costs for a shopping basket
- managing orders

Basic definitions:
------------------

- *Vendor*: A vendor is a person or organization that offers a supplyXhain endpoint
- *Customer*: A customer is a client that connects to a supplyXhain API endpoint
- *Article*: An article is a distinguishable buyable product or service with a common title / text and pictures. Articles can be further parameterized e.g. in size, color, print whereas it is still the same article.
- *Basket*: A basket is a collection of *Article*s with their article specific parameters as well as the amount ordered for each article
- *Quote*: A quote is a offer from the vendor that assigns a *price* to a *Basket* and a requested shipping method
- *Order*: An order is a quote that has been accepted and ordered by the customer
- *JSONtype*: notation for JSON syntax used in this document, for more information see https://github.com/launix-de/jsontype - you can generate your API JSON parsers and validators using the JSONtype library

Basic concepts of supplyXhain
-----------------------------

supplyXhain protocol is based on articles, quotes and orders. Before ordering an article, a price for the article and shipping has to be requested. Asking for a price is called a `Quote`. The customer can respond to a `Quote` with an `Order`.

A vendor can inform the customer about his articles in the `Catalog`. The Catalog will provide information like pictures, texts in multiple languages, prices and parameterization options.

When asking for a `Quote`, the customer must provide a `Basket` which is a collection of parameterized articles he wants to buy. The vendor will make a quote based on list prices, discounts or calculations based on provided parameters.

A `Quote` will not always reflect the `Basket` as requested, so be careful when implementing a purchase system. The vendor can leave out positions he can't deliver. He can also reduce the ordered amount due to limits or change parameters when out of range. The vendor can deny requests for conventional penalties or deny the promise of a requested delivery time. The customer's implementation has to check whether the returned `Quote` really fulfills his requrements or if he wants to switch vendors or manually discuss with the vendor about conventional penalities etc.

Once a `Quote` is ordered, the customer can provide notification URLs to get informed when a payment or delivery state changes.

Basic query schema
------------------

The API is a REST API which means it is built on HTTP/HTTPS and contains JSON data as payload.

GET requests mostly transfer parameters in GET parameters and return a `application/json` payload body.

POST requests will have at most one or two parameters encoded in the URL and GET parameters. The rest of the request payload is encoded in the POST body as JSON. The response again is a `application/json` payload.

supplyXhain URL / Primary Endpoint
----------------------------------

Every vendor will provide a supplyXhain URL. An example URL is `http://vendor.tld/products/api`.

When GETting the supplyXhain URL, basic vendor information about the vendor as well as further URLs to API endpoints are provided. The supplyXhain protocol does not prescribe a certain URL schema since every implementation will have it's own URL schema. This way, implementors can use both, URLs like `/order/:id` or `order.php?id=:id`. Querying the base URL will return the following format:

```
{
	vendor_name: string,
	version: string,
	currency: string,
	catalog_url: string,
	quote_url: string,
	categories: {*: string}
}
```

Catalog Endpoint
----------------

The `Catalog` is a GET endpoint. The following GET parameters are allowed:

- *search* (optional): search texts for this string
- *category* (optional): all items must match *category*
- *limit* (optional, default is 1024): returns at most *limit* items
- *page* (optional, starting at 0): offset results by page *page*

The query will return the following:

```
{
	limit: number,
	page: number,
	items: [article]
}
```

where

```
article := {
	id: string,
	name: i18nstring,
	detailsText?: i18nstring,
	detailsHTML?: i18nstring,
	pictures?: [picture], /* pictures */
	categories?: [string], /* category id list */
	detailURL?: string
}

i18nstring := {*: string}

picture := {
	url: string,
	thumbnailURL: string,
	title: i18nstring
}
```

Every article has an `id` - may it be EAN or any internal numbering. A article `id` is unique for that article and will be used in baskets.
i18nstrings are associative arrays assigning a text for each language. Languages are according to i18n naming conventions which means: `de_DE` is German in Germany whereas `de` is German. If an article does not provide a string for `de_AT`, `de` can be taken instead.
`detailURL` is a link to a human readable product info page from the vendor or a shop page.

Quote Endpoint
--------------

A request to a quote endpoint will provide a basket to the vendor where he can set prices for each item. The request payload is as follows:

```
{
	items: [requestItem],
	deliveryTime?: date,
	deliveryPenaltyPerDay?: number
}
```

where

```
date := /[0-9]{4}-[0-9]{2}-[0-9]{2}/

requestItem := {
	article: string /* article id here */,
	amount: number,
	parameters: {*: *}
}
```

With a quote, the customer can request a delivery time and back it with a penalty so the vendor will pay every day he is too late in delivery. Of course, the vendor can deny such penalty request but he would be better off raising the price to a certain amount to cover the risk.

The vendor will answer as follows:

```
{
	items: [quoteItem],
	deliveryTime: date,
	deliveryPenaltyPerDay: number,

	shipment: number,
	totalNet: number,
	totalGross: number
}
```

where

```
quoteItem := {
	article: string,
	amount: number,
	parameters: {*: *},
	singleListPrice?: number,
	tax: number,
	total: number,
}
```


Order Endpoint
--------------
