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
- *JSONtype*: notation for JSON syntax used in this document, for more information see https://github.com/launix-de/jsontype
