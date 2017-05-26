# (uns)Table of Contents
1. <a href="#alpha-entities-overview">Alpha Entities Overview</a>
	* <a href="#entities-index">Entities Index</a>
2. <a href="#basics-of-calling-the-api">Basics of Calling the API</a>
3. <a href="#api">API</a>
	* <a href="#getting-stores">Getting Stores</a>
	* <a href="#getting-the-menu">Getting the Menu</a>
	* <a href="#posting-an-order">Posting an Order</a>
	* <a href="#reading-all-orders">Reading all Orders</a>
	* <a href="#order-statuses">Order Statuses</a>
	* <a href="#accepting-an-order">Accepting an Order</a>
	* <a href="#completing-an-order">Completing an Order</a>
	* <a href="#cancelling-an-order">Cancelling an Order</a>
	* <a href="#querying-whether-a-customer-is-registered">Querying whether a Customer is Registered</a>
	* <a href="#register-a-customer">Register a Customer</a>
	* <a href="#update-a-customer">Update a Customer</a>
	* <a href="#delete-a-customer-address">Delete a Customer Address</a>
	* <a href="#get-customer-details">Get Customer Details</a>
	* <a href="#login-customer">Login Customer</a>
	* <a href="#logout-customer">Logout Customer</a>
	* <a href="#request-for-customer-password-change">Request for Customer Password Change</a>
	* <a href="#change-customer-password-after-request">Change Customer Password (after Request)</a>
	* <a href="#create-pin">Create PIN</a>
	* <a href="#get-eta">Get ETA</a>
	* <a href="#get-gps-coordinates-from-address">Get GPS Coordinates from Address</a>
	* <a href="#get-address-from-gps-coordinates">Get Address from GPS Coordinates</a>
	* <a href="#get-closest-store-from-address">Get Closest Store from Address</a>
	* <a href="#get-favourite-product-list-for-customer">Get Favourite Product List for Customer</a>
	* <a href="#add-a-favourite-product-for-customer">Add a Favourite Product for Customer</a>
	* <a href="#remove-a-favourite-product-for-customer">Remove a Favourite Product for Customer</a>
	* <a href="#getting-all-promotions">Getting all Promotions</a>
	* <a href="#testing-push-notifications">Testing Push Notifications</a>
4. <a href="#masterpass">Masterpass</a>
	* <a href="#testing">Testing</a>
	* <a href="#mpgs">MPGS</a>
	* <a href="#cub">CUB</a>
	* <a href="#wirecard">Wirecard</a>

# Alpha Entities Overview

* **Brand** - Alpha supports multiple Brands. Every other entity is tied to a Brand.
* **Store** - A place that you make an order to.
* **Product** - An item that can be sold on its own
* **Category** - A Product can belong to multiple Categories. A Category can have multiple child Categories.
* **Tag** - A Product can have zero or one or more Tags. A Tag is just a piece of information with an image.
* **Variant** - A Variant is some kind of variation of a Product that the customer must choose one. It has a price. Not all Products have Variants.
* **Modifier Group** - A Product can have multiple Modifier Groups. Each group has a set of Modifiers. This allows a wide range of choices to be presented to the customer
* **Modifier** - Something that modifies the final Product that the customer receives. Can have extra cost.
* **Availability** - A day + time range for when something is available

## Entities Index

* <a href="#address">Address</a>
* <a href="#availability">Availability</a>
* <a href="#brand">Brand</a>
* <a href="#banner">Banner</a>
* <a href="#category">Category</a>
* <a href="#customer">Customer</a>
* <a href="#coordinates">Coordinates</a>
* <a href="#discount">Discount</a>
* <a href="#extracharge">ExtraCharge</a>
* <a href="#product">Product</a>
* <a href="#store">Store</a>
* <a href="#table">Table</a>
* <a href="#tag">Tag</a>
* <a href="#modifier-group">Modifier Group</a>
* <a href="#modifier">Modifier</a>
* <a href="#modifieritem">ModifierItem</a>
* <a href="#order">Order</a>
* <a href="#orderitem">OrderItem</a>
* <a href="#payment">Payment</a>
* <a href="#pager">Pager</a>
* <a href="#promotion">Promotion</a>
* <a href="#rewards">Rewards</a>
* <a href="#variant">Variant</a>
* <a href="#voucher">Voucher</a>

# Basics of Calling the API

It will be at something like `/api/<version>/<service>/<brandCode>`

* GET for reading
* POST for modifying something
* Response is always a JSON
* If success, response will be `{ status: true [, code: <optional unique integer> ], property1: etc, property2: etc }`
* If failure, it will be `{ status: false, message: “Hopefully some reason why it failed” [, code: <optional unique integer> ] }`
* All entities will have the property `id`. It is a long value that is only unique for that entity
* Images will have the property `imageId`. The value will be something like “awblbnmbgwpbzpcc8sux” This is an ID to a resource on Cloudinary. It can also be null. The URL for the image will be `http://res.cloudinary.com/savantdegrees/image/upload/c_<crop-type>,h_<height>,w_<width/<id>` E.g <http://res.cloudinary.com/savantdegrees/image/upload/c_fit,h_400,w_300/awblbnmbgwpbzpcc8sux> There are many ways to customize the image, get different dimensions and weird transformations. See <http://cloudinary.com/documentation/image_transformations>
* For customer related APIs, an `authToken` is required (see Login Customer)
* Any customer related requests with an invalid authToken will return `{ success: false, code: -666, message: “Some message you can display to the user” }`. The user should be redirected to the login flow
* If the API Authentication Key is set at a brand, every API request to that brand must pass in that key. The key can be passed in as a request parameter `authorization` or a request header `Authorization`. E.g. <http://alpha.savantdegrees.com/api/1.0/menu/ALT?authorization=jO0fHEBl1C2d0WOGGIJt6amAwI84msle>
* It is recommended to set this key before you go live otherwise anyone can simply send orders to that brand

# API

## Getting Stores

`http://<server>/api/1.0/stores/<brandCode>`


### GET Parameters:
* lat: double, latitude of user's current location, optional
* lng: double, longitude of user's current location, optional
* line: string, human readable address of user's current location, optional

If `(lat and lng)` or `line` is passed in, the stores returned will be sorted in ascending order of their distances to the user's current location.

### Response:
<pre>
{
	success: true,
	stores: Array of Stores,
	serverTime: milliseconds since epochTime, current server time,
	timeZoneOffset: integer, UTC value, E.g. Singapore UTC+08:00 => 800, India UTC+05:30 => 530, Mexico City UTC-06:00 => -600
}
</pre>

#### Store:
<pre>
{
	id: ID,
	name: string,
	delivery: boolean, whether this store is available for delivery right now,
	takeAway: boolean,
	dineIn: boolean,
	presetDelivery: boolean, whether this store does delivery,
	presetTakeAway: boolean,
	presetDineIn: boolean,
	address: Address object,
	coord: Coordinates object,
	email: string,
	phone: string,
	table: Array of Table,
	openingHours: Array of Availabilities,
	dineInHours: Array of Availabilities,
	takeAwayHours: Array of Availabilities,
	deliveryHours: Array of Availabilities,
	openingHoursString: string, a nicely formatted human readable text from the 'openingHours' property,
	dineInHoursString: string, a nicely formatted human readable text from the 'dineInHours' property,
	takeAwayHoursString: string, a nicely formatted human readable text from the 'takeAwayHours' property,
	deliveryHoursString: string, a nicely formatted human readable text from the 'deliveryHours' property,
	imageId: string,
	deliveryMaxOrderValue: decimal number or null, apps should not allow a delivery order larger than this value,
	deliveryMinOrderValue: decimal number or null, apps should not allow a delivery order smaller than this value,
	sortIndex: integer, can be used for sorting Stores in ascending order,
}
</pre>


#### Address:
<pre>
{
	line1: string,
	line2: string,
	city: string,
	countryCode: string, 2 character country code,
	postalCode: string,
	string: string of the address in one line
}
</pre>


#### Coordinates:
<pre>
{
	lat: double, latitude
	lng: double, longitude
}
</pre>

#### Table:
<pre>
{
	id: ID,
	number: string, human readable number,
	sortIndex: integer, can be used for sorting Tables in ascending order,
}
</pre>

Sample: <http://alpha.savantdegrees.com/api/1.0/stores/ALT?authorization=jO0fHEBl1C2d0WOGGIJt6amAwI84msle>

## Getting the Menu

`http://<server>/api/1.0/menu/<brandCode>[/<storeId>]`

If `storeId` is passed in, only the available menu for that particular store is returned.

### GET Parameters:
* showAll: boolean, default is false, if true returns all products regardless whether they are available or not

### Response:
<pre>
{
	success: true,
	categories: Array of Categories,
	products: Array of Products,
	tags: Array of Tags,
	brand: Brand object
	banners: Array of Banners,
}
</pre>

#### Brand:
<pre>
{
	id: ID,
	imageId: string,
	name: string,
	apiCode: string,
	defaultImageId: string, default product image the client should use if any product does not have an image, beware - this can be null also
	enableProductImages: boolean,
	availabilityTimeIncrement: integer, number of minutes to use for incrementing time in a time picker
}
</pre>

#### Tag:
<pre>
{
	id: ID
	imageId: string,
	name: string,
	type: string
}
</pre>

#### Category:
<pre>
{
	id: ID,
	imageId: string,
	image2Id: string,
	image3Id: string,
	availabilities: Array of Availabilities, if empty, this Category is available the whole day,
	name: string,
	description: string,
	sortIndex: integer, can be used for sorting Categories in ascending order,
	children: Array of child Categories
}
</pre>

#### Product:
<pre>
{
	id: ID,
	imageId: string,
	image2Id: string,
	image3Id: string,
	name: string,
	description: string,
	sortIndex: integer, can be used for sorting Products in ascending order,
	price: decimal number,
	dineIn: boolean, if Store ID is passed in, whether this Product is available for dine-in right now. If Store ID is not passed in, whether ANY Stores have this available for dine-in,
	delivery: boolean, if Store ID is passed in, whether this Product is available for delivery right now. If Store ID is not passed in, whether ANY Stores have this available for delivery,
	takeAway: boolean, if Store ID is passed in, whether this Product is available for take away right now. If Store ID is not passed in, whether ANY Stores has this available for take-away,
	allDineIn: boolean, only appears when Store ID is not passed in, whether ALL Stores have this Product avaiable for dine-in right now,
	allTakeAway: boolean, only appears when Store ID is not passed in, whether ALL Stores have this Product avaiable for take-away right now,
	allDelivery: boolean, only appears when Store ID is not passed in, whether ALL Stores have this Product avaiable for delivery right now,
	categories: Array of Category IDs
	modifierGroups: Array of Modifier Groups,
	tags: Array of Tag IDs,
	variants: Array of Variants
}
</pre>

#### Modifier Group:
<pre>
{
	id: ID,
	name: string,
	description: string,
	sortIndex: integer, can be used for sorting Modifier Groups in ascending order,
	combo: boolean, whether this group contains a combo of other Products,
	minModifiers: integer, minimum number of Modifiers that must be selected for this group,
	maxModifiers: integer, maximum number of Modifiers that can be selected for this group,
	modifiers: Array of Modifiers
}
</pre>

#### Modifier:
<pre>
{
	id: ID,
	name: string,
	description: string,
	sortIndex: integer, can be used for sorting Modifiers in ascending order,
	price: decimal number,
	minQuantity: minimum number of this Modifier that must be selected,
	maxQuantity: maximum number of this Modifier that can be selected,
	product: ID of a Product if this Modifier belongs in a Modifier Group that is a combo, otherwise will be null
	preSelected: boolean, whether this Modifier should already be selected for the customer, of course the customer may end up selecting another Modifier
}
</pre>

#### Availability:
<pre>
{
	dayOfWeek: “MONDAY|TUESDAY|etc..”,
	startTime: string, format “HH:mm:ss.ms”,
	endTime: string, format same as above
}
</pre>

#### Banner:
<pre>
{
	id: ID,
	imageId: string,
	product: ID of product this banner is for, or null, apps should go directly to this product page if it is not null
	sortIndex: integer,
	link: URL string or null,
	name: string,
	startTime: milliseconds since epochTime, time to start showing this banner,
	endTime: milliseconds since epochTime, time to stop showing this banner,
	target: “PRODUCT|LINK|NONE”,
	position: ”TOP|BOTTOM” or null,
	description: string,
	type: "GENERAL|ABOUT_BRAND|ONBOARDING" or null
}
</pre>

#### Variant:
<pre>
{
	id: ID,
	price: decimal number,
	modifierGroups: Array of Modifier Groups,
	name: string
}
</pre>

#### Samples:
1. <http://alpha.savantdegrees.com/api/1.0/menu/ALT?authorization=jO0fHEBl1C2d0WOGGIJt6amAwI84msle>
2. <http://alpha.savantdegrees.com/api/1.0/menu/MLP?authorization=jO0fHEBl1C2d0WOGGIJt6amAwI84msle>
3. <http://alpha.savantdegrees.com/api/1.0/menu/MLP/19?authorization=jO0fHEBl1C2d0WOGGIJt6amAwI84msle>

Note that at store 19, only takeaway is available for that product.

## Posting an Order

`http://<server>/api/1.0/order/<brandCode>`

### POST body:
<pre>
{
	type: “DELIVERY|TAKE_AWAY|DINE_IN|TEST”,
	items: Array of OrderItems,
	payments: Array of Payments,
	discounts: Array of Discounts,
	store: ID of store that is receiving this order,
	table: ID of table for this order, can be NULL,
	authToken: authentication token of current customer (if null or missing, this is a guest order),
	address: Address object (only applicable for DELIVERY),
	specialRequest: string,
	origin: “IOS|ANDROID” or NULL,
	test: boolean, default false, if true, it’s as if type == “TEST” but order will behave correctly as the intended type,
	fulfillmentTime: milliseconds since epoch time, time the customer wants to be fulfilled, if null, means NOW
	customer: Guest object, optional,
	phone: string, optional, if passed in sets the phone number that the merchant can call specifically for that order,
	vouchers: Array of Voucher IDs to be applied to this order,
	promoCodes: Array of promo codes to be applied to this order,
	rewardPoints: integer,
	pax: integer, number of people eating in (only applicable for DINE_IN),
	masterpassPayment: object, optional to pay using masterpass,
	paymentIds: Array of Payment Ids used for making payments (for payments using web view inside mobile applications)
}
</pre>

If type == “TEST” the server will return a response as if this order is successfully placed. Useful for retrieving taxes, applying promotion codes, vouchers and so on.

**Apps should not pass in any taxes and/or discounts from promotions. Those will automatically calculated and applied by Alpha when the order is posted.**

See <a href="#getting-all-promotions">here</a> for more details on applying promotions.

#### Guest:
<pre>
{
	firstName: optional
	lastName: optional,
	phone: optional,
	email: optional
}
</pre>

#### OrderItem:
<pre>
{
	id: ID, ignored when POSTed to endpoint
	product: ID of product,
	productName: string, name of product, ignored when POSTed to endpoint
	quantity: integer,
	modifiers: Array of ModifierItems,
	total: decimal, ignored when POSTed to endpoint
	specialRequest: string, instructions provided by user when ordering,
	variant: ID of Variant that is chosen, can be null for Products without Variants
	variantName: string name of Variant that was chosen, ignored when POSTed to endpoint,
	variantPrice: decimal number, price of variant chosen, if no variant was chosen, this will be null, ignored when POSTed to endpoint
}
</pre>

#### ModifierItem:
<pre>
{
	id: ID, ignored when POSTed to endpoint
	modifierName: string, ignored when POSTed to endpoint
	modifierId: ID of Modifier this is referring to
	quantity: integer,
	modifiers: Array of ModifierItems (only applicable if this modifier is a combo product)
}
</pre>

#### Payment:
<pre>
{
	id: ID, ignored when POSTed to endpoint
	amount: decimal number,
	type: “CASH|CREDIT_CARD|PAYPAL|CREDITS”,
	transactionId: string, can be anything, this value will just be stored as information,
	paidTime: long, milliseconds since epoch time,
	status: “PAID|CANCELLED|REFUNDED”
}
</pre>

#### Address:
<pre>
{
	line1: string
	line2: string, optional
	city: string
	countryCode: 2 character string, ISO 3166-2 country code
	postalCode: string
}
</pre>

### Response:
<pre>
{
	success: true/false,
	order: Order object,
	vouchers: Array of VoucherErrors,
	promoCodes: Array of PromoCodeErrors

}
</pre>

When promotions are applied, success will be false if the any of the vouchers or promo codes applied are not valid. The `vouchers` property and `promoCodes` will contain details about why they are invalid.

#### VoucherError:
<pre>
{
	id: ID of Voucher that is invalid,
	message: string, a human readable string that explains why the voucher is invalid
}
</pre>

#### PromoCodeError:
<pre>
{
	code: string, the promo code that is invalid,
	message: string, a human readable string that explains why the promo code is invalid
}
</pre>

#### Order:
<pre>
{
	id: ID, ignored when POSTed to endpoint,
	type: “DELIVERY|TAKE_AWAY|DINE_IN|TEST”,
	number: string, human readable unique number of this order, ignored for TEST,
	items: Array of OrderItems,
	extraCharges: Array of ExtraCharges,
	payments: Array of Payments,
	discounts: Array of Discounts
	placeTime: long, milliseconds since epoch time, the time this order was placed,
	status: “PLACED|COMPLETED|CANCELLED|DELETED|READY”, see Order Status section,
	total: decimal, absolute final total, ignored when POSTed to endpoint,
	store: ID of store,
	table: ID of table,
	storeObject: Store object,
	tableObject: Table object,
	specialRequest: string,
	etaTime: long, milliseconds since epoch time, time the order will actually be fulfilled,
	fulfillmentTime: long, miliseconds, since epoch, time the customer wants the order to be fulfilled,
	phone: customer’s phone number, may be null,
	pax: number of people eating in
}
</pre>

#### ExtraCharge:
<pre>
{
	id: ID, ignored when POSTed to entpoint,
	percentage: decimal, can be NULL, for “20%”, this value will be = 20,
	type: “GST|SERVICE_CHARGE|DELIVERY_CHARGE|OTHERS”,
	comment: string,
	amount: decimal, absolute amount in dollars of this extra charge,
	name: name as specified by the admin if applicable,
	string: human friendly value to show to customers,
	tax: boolean, where this charge is a tax, apps may need to treat the display of taxes differently, also taxes should be displayed last,
	inclusive: boolean, whether this charge is already included in item price
}
</pre>

#### Discount:
<pre>
{
	id: ID, ignored when POSTed to endpoint,
	percentage: same idea from ExtraCharge’s percentage,
	amount: same,
	reason: string, why this discount is given,
	promotion: ID of the promotion used for creating this discount, ignored when POSTed to endpoint,
	voucher: ID of the voucher used for creating this discount, ignored when POSTed to endpoint
}
</pre>

#### Samples:

##### Posting a test order: `http://alpha.savantdegrees.com/api/1.0/order/ALT?authorization=jO0fHEBl1C2d0WOGGIJt6amAwI84msle`
<pre>
{
	"type": "TEST",
	"items": [
		{
			"product": 31,
			"quantity": 1,
			"modifiers": [
				{
					"modifierId": 610,
					"quantity": 1
				},
				{
					"modifierId": 606,
					"quantity": 3
				}
			]
		},
		{
			"product": 4,
			"quantity": 1,
			"modifiers": [
				{
					"modifierId": 261,
					"quantity": 1
				},
				{
					"modifierId": 265,
					"quantity": 1
				},
				{
					"modifierId": 266,
					"quantity": 2
				}
			]
		},
		{
			"product": 273,
			"quantity": 1,
			"variant": 6
		}
	],
	"discounts": [
		{
			"amount": 2.99,
			"reason": "$2.99 off for using this cool app"
		}
	],
	"payments": [
		{
			"amount": 49.01,
			"type": "PAYPAL",
			"transactionId": "PAYPAL-123456",
			"status": "PAID",
			"paidTime": 1457273565769
		}
	],
	"store": 1
}
</pre>

## Reading all Orders

`http://<server>/api/1.0/orders/<brandCode>`


### GET Parameters:
* store: ID of Store, if missing Orders from all Stores are returned
* pageSize: maximum number of records to return per page, if missing = 20
* pageNumber: 0-based index of page to return, if missing = 0
* sortOrder: “ASC|DESC”, if missing = “ASC”
* sortProperty: “placeTime”, for now, if missing = “placeTime”
* status: “PLACED|COMPLETED|CANCELLED”, if missing, all orders will be returned
* placeTimeFrom: long, epoch time in millisecond, inclusive
* placeTimeTo: long, epoch time in millisecond, exclusive
* authToken: if passed in, will only return orders for this customer
* number: string, order number

### Response:
<pre>
{
	success: true,
	orders: Array of Order,
	pager: Pager object
}
</pre>

#### Pager:
<pre>
{
	count: integer, total number of records that match the query
	pageSize: integer, size (number of records) per page
	pageNumber: 0-based index of page
}
</pre>

## Order Statuses

When an order is submitted, its status is PLACED. The state transitions are as follows:

* PLACED -&gt; QUEUED | PREPARATION | CANCELLED
* QUEUED -&gt; PREPARATION | CANCELLED
* PREPARATION -&gt; READY | CANCELLED
* READY -&gt; COMPLETED |  CANCELLED
* COMPLETED -&gt; CANCELLED


The following are descriptions for each status (values in brackets are what is displayed to back-end users):

* **PLACED (Pending)**: Order is placed and pending acceptance
* **QUEUED (Pre-order)**: Order is queued to be prepared at a future time (for pre-orders, i.e. fulfillmentTime is not null). This happens when the order is accepted
* **PREPARATION (Preparing)**: Order is being prepared in the kitchen. This happens when the order is accepted (for ASAP orders), or when the order is close to fulfilment time (for pre-orders) i.e fulfilment time - delivery time - kitchen prep time
* **READY (Ready / In-Transit)**: Order is ready for serving or delivery. This happens automatically based on the kitchen preparation time set by the store administrator
* **COMPLETED**: Order is completed. This only happens when someone completes the order
* **CANCELLED**: Order is cancelled for some reason. An order can be cancelled at any time.

## Accepting an Order

`http://<server>/api/1.0/order/<brandCode>/accept/<order number>`

### Response:
<pre>
{
	success: true,
	order: Order object
}
</pre>

## Completing an Order

`http://<server>/api/1.0/order/<brandCode>/complete/<order number>`

### Response:
<pre>
{
	success: true,
	order: Order object
}
</pre>

## Cancelling an Order

`http://<server>/api/1.0/order/<brandCode>/cancel/<order number>`

### Response:
<pre>
{
	success: true,
	order: Order object
}
</pre>

## Querying whether a Customer is Registered

`http://<server>/api/1.0/customer/registered/<brandCode>`

### POST Parameters:
* email

### Response:
<pre>
{
	success: true,
	registered: boolean
}
</pre>

## Register a Customer

`http://<server>/api/1.0/customer/register/<brandCode>`

*Don’t pass in the parameter if not available*

### POST Parameters:
* firstName
* lastName
* email
* password
* gender: “MALE|FEMALE”
* origin: “ANDROID|IOS”
* dateOfBirth: long value, epoch time in milliseconds
* phone
* facebookId
* googlePlusId
* lineId: LINE mid
* accessToken: Required when facebookId, googlePlusId or lineId is passed in
* refreshToken: lineId refresh token
* addressName: name of address, free-form value e.g. “Home”, “Office”
* addressLine1
* addressLine2
* addressCity
* addressCountryCode: 2-character string ISO 3166-2
* addressPostalCode
* deviceToken: device token for push notifications
* deviceType: “ANDROID|IOS”
* nationalityCode: 2-character string ISO 3166-2 country code

### Response:
<pre>
{
	success: true,
	authToken: string,
	code: integer
}
</pre>

#### code values:
* -100 : Email already exists
* -101:  Email already exists and account is connected to Facebook
* -102: Email already exists and account is connected to Google Plus
* -106: Email already exists and account is connected to LINE

## Update a Customer

`http://<server>/api/1.0/customer/update/<brandCode>`

*Don’t pass in the parameter if not updating.*

### Additional POST Parameters:
* authToken: authentication token of the user
* addressId: ID of address to update, if not passed in when address data is passed in the system will create a new address for the customer
* currentPin: string, current 6 digit PIN, required when updating 6 digit PIN
* pin: string, new 6 digit PIN, required when updating 6 digit PIN

The rest are same as register.

## Delete a Customer Address

`http://<server>/api/1.0/customer/deleteAddress/<brandCode>`

### POST Parameters:
* authToken: string
* id: ID of Address to delete

### Response:
<pre>
{
	success: true
}
</pre>

## Get Customer Details

`http://<server>/api/1.0/customer/details/<brandCode>`

### GET Parameters:
* authToken: string

### Response:
<pre>
{
	success: true
	customer: Customer Object
}
</pre>

#### Customer:
<pre>
{
	id: ID
	firstName
	lastName
	email
	phone
	googlePlusId
	facebookId
	gender: “MALE|FEMALE” or null
	dateOfBirth: milliseconds since epoch time, or null
	addresses: List of Addresses
	rewards: Rewards object
	credits: decimal, number of credits the customer has left, should only be used for display
	countryCode: 2-character string country code,
	vouchers: Array of Vouchers this customer has in their account, this can be applied on an Order
	hasPin: boolean, whether the customer account has a pin
}
</pre>

#### Voucher:
<pre>
{
	id: ID,
	promotion: Promotion object,
	used: integer, number of times this voucher has been used, only applicable for vouchers that can be used multiple times per customer
}
</pre>


#### Rewards:
<pre>
{
	totalRewardPoints
	expiryDate: miliseconds since epoch time, when the reward points will expire
	earnPointAmount
	earnPoint
	redeemPointAmount
	redeemPoint
	description
	earnExclusionCategories: List of Categories
	redeemExclusionCategories: List of Categories
}
</pre>

## Login Customer

`http://<server>/api/1.0/customer/login/<brandCode>`

### POST Parameters:
* email
* password
* facebookId
* googlePlusId
* lineId: LINE mid</li>
* accessToken: Facebook or Google Plus or LINE access token
* refreshToken: LINE refresh token
* deviceToken: device token for push notifications
* deviceType: “ANDROID|IOS”

(email and password) OR (facebookId and accessToken) OR (googlePlusId and accessToken) OR (lineId and accessToken) is required

### Response:
<pre>
{
	success: true,
	authToken: string
}
</pre>

## Logout Customer

`http://<server>/api/1.0/customer/logout/<brandCode>`

### POST Parameters:
* accessToken: Facebook or Google Plus access token
* authToken: string
* deviceToken: device push token so as to disable push notifications to this device

### Response:
<pre>
{
	success: true
}
</pre>

## Request for Customer Password Change

`http://<server>/api/1.0/customer/forgotPassword/<brandCode>`

### POST Parameters:
* email: customer email address

### Response:
<pre>
{
	success: true
}
</pre>

#### code values (if success: false)
* -103: Email not found

## Change Customer Password (after Request)

`http://<server>/api/1.0/customer/changePassword/<brandCode>`

### POST Parameters:
* email: customer email address
* pin: pin number that was sent to their email account
* password: new password

### Response:
<pre>
{
	success: true
}
</pre>

#### code values (if success: false)
* -103: Email not found
* -104: Pin is incorrect
* -105: Code has expired

## Create PIN

`http://<server>/api/1.0/customer/createPin/<brandCode>`

### POST parameters:

<ul><li>authToken: string</li>
<li>pin: string, 6 digit PIN used to authenticate Masterpass transactions</li></ul>

### Response

<pre>
{
	success: true
}
</pre>

## Get ETA

`http://<server>/api/1.0/eta/<brandCode>`

### GET Parameters:
* store: ID of store</li>
* orderType: “DELIVERY|TAKE_AWAY|DINE_IN”
* address: string, pass in either this or (postalCode and countryCode)
* postalCode: pass in (this and countryCode) or address
* countryCode: see postalCode
* subTotal: decimal number, defaults to 0 if not passed in

### Response:
<pre>
{
	success: true,
	eta: integer, estimated time of arrival in minutes
}
</pre>

## Get GPS Coordinates from Address

`http://<server>/api/1.0/location/coordinates`

### GET Parameters:
* line: string, any free-form string containing some kind of address

### Response:
<pre>
{
	success: true,
	lat: latitude,
	lng: longitude
}
</pre>

## Get Address from GPS Coordinates

`http://<server>/api/1.0/location/address`

### GET Parameters:
* lat: double, latitude
* lng: double, longitude

### Response:
<pre>
{
	success: true,
	address: Address object
}
</pre>

## Get Closest Store from Address

`http://<server>/api/1.0/location/closestStore/<brandCode>`

### GET Parameters:
* line: address string

### Response:
<pre>
{
	success: true,
	store: Store object
}
</pre>

## Get Favourite Product List for Customer

`http://<server>/api/1.0/customer/favourites`

### GET Parameters:
* authToken: string
* productData: boolean, if true returns product details in 'favourites' property of the response

### Response:
<pre>
{
	success: true,
	favourites: Array of Favourites
}
</pre>

#### Favourite:
<pre>
{
	product: ID of product,
	variant: ID of variant,
	time: milliseconds since epoch time, time this product was favourited,
	data: Product object, only sent when productData parameter is true,
	modifiers: array of Modifier objects
}
</pre>

####Modifier:
<pre>
{
	modifier: ID of modifier,
	quantity: integer
}
</pre>

## Add a Favourite Product for Customer

`http://<server>/api/1.0/customer/favourite`

### POST Parameters:
* authToken: string
* product: ID of product,
* variant: optional, ID of variant,
* modifiers: optional, array of modifier objects. Eg: modifiers[0].modifier=123&modifiers[0].quantity=1

### Response:
<pre>
{
	success: true
}
</pre>

## Remove a Favourite Product for Customer

`http://<server>/api/1.0/customer/unfavourite`

### POST Parameters:
* authToken: string
* product: ID of product,
* variant: optional, ID of variant,
* modifiers: optional, array of modifier objects. Eg: modifiers[0].modifier=123&modifiers[0].quantity=1

### Response:
<pre>
{
	success: true
}
</pre>

## Getting all Promotions

`http://<server>/api/1.0/promotions/<brandCode>`

Promotions are to be displayed only. There are two types of promotions - vouchers and promo codes. Vouchers are automatically given to customers in their account (see <a href="#customer">Customer</a> `vouchers` property). Customers can then choose which vouchers to apply into their orders. Promo codes on the other hand are free to be used at any time. They simply have to be entered into the order. Of course, terms and conditions still apply.

Apps should pass in vouchers and/or promo codes selected by the customer to Alpha with "test" = true, to query whether the promotion(s) can be applied. Alpha will return the order with discounts if the promotion(s) are valid. This should be presented to the customer before paying.

Note that payments will be not validated by Alpha. Users usually pay first before the app posts the order to Alpha. Therefore, apps **must** restrict the payment options when a promotion that requires only certain payments is applied.

### Response:
<pre>
{
	success: true,
	promotions: Array of Promotions
}
</pre>

#### Promotion:
<pre>
{
	id: ID,
	name: string,
	description: string,
	imageId: string,
	tnc: string, terms and conditions,
	benefitType: "FREE_ITEM|DOLLAR_DISCOUNT_ON_ITEM|DOLLAR_DISCOUNT_ON_CART|PERCENTAGE_DISCOUNT_ON_ITEM|PERCENTAGE_DISCOUNT_ON_CART", see Benefit Types below for more details,
	periodType: "LIMITED_PERIOD|FOREVER",
	periodFrom: millseconds since epochTime, when this promotion starts, only applicable when periodType == "LIMITED_PERIOD",
	periodTo: millseconds since epochTime, last day before this promotion ends, only applicable when periodType == "LIMITED_PERIOD",
	fulfillmentType: "ANY_TIME|CUSTOM", ANY_TIME means this can be redeemed anytime during the promotion period, CUSTOM means can only be redeemed during the fulfillment times,
	fulfillmentTimes: Array of Availabilities, can only be redeemed during these hours, only applicable when fulfillmentType == "CUSTOM",
	publishType: "VOUCHERS|PROMO_CODE",
	exclusive: boolean, if true this promotion can only be the only promotion used in an order,
	code: string, the promo code, only appicable when publsihType == "PROMO_CODE",
	freeType: "PRODUCT|MODIFIER", if PRODUCT the free item will be a specific product, if MODIFIER, it will be a specific modifier, only applicable when benefitType == "FREE_ITEM",
	freeProduct: ID of Product or null, only applicable when freeType == "PRODUCT",
	freeModifier: ID of Modifier or null, only applicable when freeType == "MODIFIER",
	takeAway: boolean, whether this promotion can be used for take away orders,
	dineIn: boolean, whether this promotion can be used for dine in orders,
	delivery: boolean, whether this promotion can be used for delivery orders,
	takeAwayStores: Array of Store IDs, only applicable when takeAway is true, and if empty, means ALL stores are applicable,
	dineInStores: Array of Store IDs, only applicable when dineIn is true, and if empty, means ALL stores are applicable,
	deliveryStores: Array of Store IDs, only applicable when delivery is true, and if empty, means ALL stores are applicable,
	emailDomains: Array of strings, e.g. of a string - "@eunoia.asia", only customers whose emails match any of these domains can apply this promotion,
	paymentTypes: Array of strings, currently only "CASH", "MASTERPASS", "PAYPAL", "STRIPE", "CREDIT_CARD", only orders paid with any of these payment methods are applicable, if empty all payment types are applicable,
	dollarDiscount: decimal number, dollars off a product or cart, only applicable when benefitType == "DOLLAR_DISCOUNT_ON_ITEM" || "DOLLARY_DISCOUNT_ON_CART",
	percentageDiscount: decimal number, x% off off a product or cart, only applicable when benefitType == "PERCENTAGE_DISCOUNT_ON_ITEM" || "PERCENTAGE_DISCOUNT_ON_CART",
	criteriaProducts: Array of Product IDs, products that will receive dollar or percentage discount, only applicable when benefitType == "DOLLAR_DISCOUNT_ON_ITEM" || "PERCENTAGE_DISCOUNT_ON_ITEM",
	minAmountSpent: decimal number, minimum amount that must be spent (sub-total) before this promotion can be applied,
	maxDiscountValue: decimal number, maximum amount of discount that will be capped when applying this promotion,
	stockCount: "LIMITED|UNLIMITED", if LIMITED, this promotion can be only be redeemed stockLimit number of times,
	stockLimit: number of times this promotion can be redeemed, only applicable when stockCount == "LIMITED",
	usageCount: "LIMITED|UNLIMITED", if LIMITED, this promotion can be only be redeemed usageLimit number of times per customer,
	usageLimit: number of times this promotion can be redeemed by a customer, only applicable when usageCount == "LIMITED",
	redeemed: number of times this promotion has been redeemed by any customers, apps should simply not allow this promotion to be applied when this value is equal (or greater) than stockLimit
}
</pre>

#### Benefit Types

The following are the values of this enum:

* **FREE\_ITEM**: The benefit of this promotion is a free product or modifier. Apps should remind the user the item is not added to their order if it is not there. Otherwise, the server will simply return an error anyway.
* **DOLLAR\_DISCOUNT\_ON\_ITEM**: Dollar discount on a product that is in the criteria list of products
* **DOLLAR\_DISCOUNT\_ON\_CART**: Dollar discount on a whole order
* **PERCENTAGE\_DISCOUNT\_ON\_ITEM**: Percentage discount on a product that is in the criteria list of products
* **PERCENTAGE\_DISCOUNT\_ON\_CART**: Percentage discount on a whole order

## Testing Push Notifications

`http://<server>/api/1.0/test/<brandCode>/push`

### POST Parameters:
* message: string
* deviceType: "ANDROID|IOS"
* title: string
* deviceId: string

Configuration to enable this to work is under Brand > Edit under the Brands menu.

The following fields have to be entered for GCM push notifications to work:

* Android Push Key

The following fields have to be entered for APN push notifications to work:

* Upload iOS Push Key File (.p12)
* iOS Push Key Password
* iOS Push Server Destination

# Masterpass

Payments using Masterpass Express Checkout has 4 main steps.

1. Pairing: Pairing can be completed in one of two ways - pairing outside of checkout (eg: during customer registration) or pairing during checkout. This process allows the app to pair with the customer's Masterpass wallet subsequently allowing the app to show available credit cards and process payment without having the user to login to Masterpass.
2. Precheckout: This API, when used after pairing is completed, allows the app to show the user's available credit cards and use them in express checkout payments. Precheckout data can be requested directly for all subsequent purchases without having to pair again. This data must not be stored in a device or server and should be requested each time the customer requires to perform Masterpass transactions.
3. Express checkout: This is the payment step, which can be done as part of Posting an Order API by using the optional masterpassPayment object with customer's selected values from precheckout response or pairing during checkout response.
4. Unpairing: This is an optional step to allow the user to unpair their Masterpass wallet from the app.

The following card types are supported for payment using a Masterpass wallet:

1. MasterCard
2. American Express
3. Diners
4. Discover
5. Maestro
6. Visa

Masterpass express checkout payments must be authenticated using either the mobile device touch authentication and/or a 6 digit PIN. Reference: <a href="#create-pin">Create PIN</a> & <a href="#update-a-customer">Update PIN</a>

## Testing

Masterpass payments can be tested by using wallets [created](https://sandbox.wallet.masterpass.com/Wallet/masterpass/en-sg/) containing one of the following test credit cards.

<table>
	<thead>
		<tr>
			<th>Card number</th>
			<th>CVC</th>
			<th>Expiry date</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>5506900140100107</td>
			<td>Any</td>
			<td>Any</td>
		</tr>
		<tr>
			<td>5506900140100206</td>
			<td>Any</td>
			<td>Any</td>
		</tr>
		<tr>
			<td>5506900140100305</td>
			<td>Any</td>
			<td>Any</td>
		</tr>
		<tr>
			<td>5506900140100404</td>
			<td>Any</td>
			<td>Any</td>
		</tr>
		<tr>
			<td>5506900140100503</td>
			<td>Any</td>
			<td>Any</td>
		</tr>
		<tr>
			<td>5506900140200105</td>
			<td>Any</td>
			<td>Any</td>
		</tr>
	</tbody>
</table>

## MPGS

### Pairing request

`http://<server>/api/1.0/mpgs/pairingRequest/<brandCode>`

This is the pairing process outside of checkout that can be completed during user registration or outside of the checkout process.

#### POST Parameters:
<ul><li>authToken: string</li>
<li>deviceToken: string, unique device identifier</li></ul>

#### Response:

The url `http://<server>/masterpass` should be opened inside a WebView within the app with the injected JavaScript (Eg: see [here](https://facebook.github.io/react-native/docs/webview.html#injectedjavascript)) from the response. After the user pairs with Masterpass on the WebView, if the URL contains `pairingCancel` or `pairingFail`, then the pairing operation has not been completed and the app must navigate the user away from the WebView. If the URL contains `pairingSuccess`, then the pairing has been completed successfully.

<pre>
{
	success: true,
	injectedJavaScript: string, to be injected into the WebView on the app,
}
</pre>

### Pairing during checkout request

`http://<server>/api/1.0/mpgs/pairingCheckoutRequest/<brandCode>`

This is the pairing and checkout process that can be used when user wants to checkout a transaction, but has never paired before.

#### POST Parameters:
<ul><li>authToken: string,</li>
<li>deviceToken: string, unique device identifier,</li>
<li>amount: decimal, absolute amount in dollars for this payment</li></ul>

#### Response:

The same url for pairing request can be opened here inside a WebvIew within the app with the injected JavaScript from the response. After user completes authentication and pairing, if the URL contains `pairingCheckoutCancel` or `pairingCheckoutFail`, then the pairing during checkout operation has not been completed and the app must navigate the user away from the WebView. If the URL contains `pairingCheckoutSuccess`, then the pairing and checkout has been completed successfully. After this step, authentication in app must be completed (using either pin or fingerprint) and then order can be placed using the Posting an Order API using the resourceId from this response.

<pre>
{
	success: true,
	injectedJavaScript: string, to be injected into the WebView on the app,
	sessionId: string, value to be passed in masterpassPayment object when posting the order
}
</pre>

### Precheckout request

`http://<server>/api/1.0/mpgs/precheckoutRequest/<brandCode>`

#### POST Parameters:
<ul><li>authToken: string</li>
<li>deviceToken: string, unique device identifier</li></ul>

#### Response:

The response provides a base64 encoded wallet data (sample provided below) which can be used to display card options to the customer in the app. The selected information can be used in the place order request to pay using Masterpass. If the precheckout request fails, redirect the user to pair with their wallet again. The precheckout response cannot be used again (after using it in a transaction) or stored within the app - it should be requested each time before displaying the card details to customer.

<pre>
{
	success: true,
	sessionId: string,
	walletData: string, decode using base64
}
</pre>

#### Wallet data sample

<pre>
{
	"precheckoutData": {
		"billing": {
			"address": {
				"country": "SGP"
			}
		},
		"card": [
			{
				"brand": "BRAND",
				"default": false,
				"expiry": {
					"month": "MONTH",
					"year": "YEAR"
				},
				"masterpassCardId": "CARDID",
				"nameOnCard": "NAMEONCARD",
				"number": "CARDLAST4DIGITS"
			},
			{
				"brand": "BRAND",
				"default": false,
				"expiry": {
					"month": "MONTH",
					"year": "YEAR"
				},
				"masterpassCardId": "CARDID",
				"nameOnCard": "NAMEONCARD",
				"number": "CARDLAST4DIGITS"
			}
		],
		"consumerWalletId": "CONSUMERWALLETID",
		"customer": {
			"email": "EMAIL",
			"firstName": "FIRSTNAME",
			"lastName": "LASTNAME",
			"phone": "PHONE"
		},
		"precheckoutTransactionId": "PRECHECKOUTTRANSACTIONID",
		"walletName": "masterpass"
	}
}
</pre>

### Masterpass Express Checkout Payment

Apps should pass in `masterpassPayment` object in the Order JSON in the POST body of the Posting an Order API to process payment using Express Checkout. Passing in a test order will not process payment through this method. If this object is missing or null, Alpha considers the customer does not intend to pay with Masterpass and proceeds to process the order.

#### Masterpass Payment Object

<pre>
{
	pin: string, optional 6 digit pin for verification,
	deviceToken: string, unique device identifier,
	sessionId: string, session id from the precheckout or pairing and checkout response,
	transactionId: string, optional precheckout transaction id if precheckout response is available,
	cardId: string, optional id of selected card if precheckout response is available
}
</pre>

## CUB

### Pairing request

`http://<server>/api/1.0/cub/pairingRequest/<brandCode>`

This is the pairing process outside of checkout that can be completed during user registration or outside of the checkout process.

#### POST Parameters:
<ul><li>authToken: string</li>
<li>deviceToken: string, unique device identifier</li></ul>

#### Response:

The url `http://<server>/masterpass` should be opened inside a WebView within the app with the injected JavaScript (Eg: see [here](https://facebook.github.io/react-native/docs/webview.html#injectedjavascript)) from the response. After the user pairs with Masterpass on the WebView, if the URL contains `pairingCancel` or `pairingFail`, then the pairing operation has not been completed and the app must navigate the user away from the WebView. If the URL contains `pairingSuccess`, then the pairing has been completed successfully.

<pre>
{
	success: true,
	injectedJavaScript: string, to be injected into the WebView on the app,
}
</pre>

### Pairing during checkout request

`http://<server>/api/1.0/cub/pairingCheckoutRequest/<brandCode>`

This is the pairing and checkout process that can be used when user wants to checkout a transaction, but has never paired before. Redemption option is not available if this API is used for payment, however, subsequent payments by fetching precheckout data allows for redemption to be used.

#### POST Parameters:
<ul><li>authToken: string,</li>
<li>deviceToken: string, unique device identifier,</li>
<li>amount: decimal, absolute amount in dollars for this payment,</li>
<li>store: ID of the store where the order is being placed</li></ul>

#### Response:

The same url for pairing request can be opened here inside a WebvIew within the app with the injected JavaScript from the response. After user completes authentication and pairing, if the URL contains `pairingCheckoutCancel` or `pairingCheckoutFail`, then the pairing during checkout operation has not been completed and the app must navigate the user away from the WebView. If the URL contains `pairingCheckoutSuccess`, then the pairing and checkout has been completed successfully. After this step, authentication in app must be completed (using either pin or fingerprint) and then order can be placed using the Posting an Order API using the resourceId from this response.

<pre>
{
	success: true,
	injectedJavaScript: string, to be injected into the WebView on the app,
	resourceId: string, value to be passed in masterpassPayment object when posting the order
}
</pre>

### Precheckout request

`http://<server>/api/1.0/cub/precheckoutRequest/<brandCode>`

#### POST Parameters:
<ul><li>authToken: string</li>
<li>deviceToken: string, unique device identifier</li></ul>

#### Response:

The response provides a base64 encoded wallet data (sample provided below) which can be used to display card options and availability of redemption to the customer in the app. The selected information can be used in the place order request to pay using Masterpass. If the precheckout request fails, redirect the user to pair with their wallet again. The precheckout response cannot be used again (after using it in a transaction) or stored within the app - it should be requested each time before displaying the card details to customer.

<pre>
{
	success: true,
	resourceId: string,
	walletData: string, decode using base64,
	redeemAvailable: boolean, whether redemption is available for this wallet
}
</pre>

#### Wallet data sample

<pre>
&lt;PreCheckoutData xmlns:ns2="http://tempuri.org/MasterPassExpressCheckoutCommonType.xsd"&gt;
   	&lt;ns2:WALLETNAME&gt;WALLETNAME&lt;/ns2:WALLETNAME&gt;
   	&lt;ns2:PRECHECKTRANSACTIONID&gt;PRECHECKOUTTRANSACTIONID&lt;/ns2:PRECHECKTRANSACTIONID&gt;
   	&lt;ns2:CONSUMERWALLETID&gt;CONSUMERWALLETID&lt;/ns2:CONSUMERWALLETID&gt;
   	&lt;ns2:PROFILE&gt;
   		&lt;ns2:FIRSTNAME&gt;FIRSTNAME&lt;/ns2:FIRSTNAME&gt;
   		&lt;ns2:MIDDLENAME&gt;MIDDLENAME&lt;/MIDDLENAME&gt;
   		&lt;ns2:LASTNAME&gt;LASTNAME&lt;/ns2:LASTNAME&gt;
   		&lt;ns2:EMAILADDRESS&gt;EMAILADDRESS&lt;/ns2:EMAILADDRESS&gt;
   		&lt;ns2:PHONENUMBER&gt;PHONENUMBER&lt;/ns2:PHONENUMBER&gt;
   	&lt;/ns2:PROFILE&gt;
   	&lt;ns2:CARDS&gt;
   		&lt;ns2:CARD&gt;
   			&lt;ns2:CARDID&gt;123456&lt;/ns2:CARDID&gt;
   			&lt;ns2:BRANDID&gt;master&lt;/ns2:BRANDID&gt;
   			&lt;ns2:BRANDNAME&gt;MasterCard&lt;/ns2:BRANDNAME&gt;
   			&lt;ns2:LASTFOUR&gt;1234&lt;/ns2:LASTFOUR&gt;
   			&lt;ns2:SETDEFAULT&gt;true&lt;/ns2:SETDEFAULT&gt;
   		&lt;/ns2:CARD&gt;
   		&lt;ns2:CARD&gt;
   			&lt;ns2:CARDID&gt;123456&lt;/ns2:CARDID&gt;
   			&lt;ns2:BRANDID&gt;visa&lt;/ns2:BRANDID&gt;
   			&lt;ns2:BRANDNAME&gt;Visa&lt;/ns2:BRANDNAME&gt;
   			&lt;ns2:LASTFOUR&gt;1234&lt;/ns2:LASTFOUR&gt;
   		&lt;/ns2:CARD&gt;
   	&lt;/ns2:CARDS&gt;
&lt;/PreCheckoutData&gt;
</pre>

### Masterpass Express Checkout Payment

Apps should pass in `masterpassPayment` object in the Order JSON in the POST body of the Posting an Order API to process payment using Express Checkout. Passing in a test order will not process payment through this method. If this object is missing or null, Alpha considers the customer does not intend to pay with Masterpass and proceeds to process the order.

#### Masterpass Payment Object

<pre>
{
	pin: string, optional 6 digit pin for verification,
	deviceToken: string, unique device identifier,
	resourceId: string, resource id from the precheckout or pairing and checkout response,
	transactionId: string, optional precheckout transaction id if precheckout response is available,
	cardId: string, optional id of selected card if precheckout response is available,
	redeem: boolean, optional whether the payment uses Masterpass redemption feature, used only when precheckout response is available
}
</pre>

### Unpairing request

`http://<server>/api/1.0/cub/unpairingRequest/<brandCode>`

#### POST Parameters
<ul><li>authToken: string</li>
<li>deviceToken: string, unique device identifier</li></ul>

#### Response:

The reponse tells whether the customer has been unpaired from the Masterpass wallet for that device.

<pre>
{
	success: true
}
</pre>

## Wirecard

### Pairing request

`http://<server>/api/1.0/wirecard/pairingRequest/<brandCode>`

#### POST Parameters:
<ul><li>authToken: string</li>
<li>deviceToken: device token for push notifications</li></ul>

#### Response:

The url `http://<server>/masterpass` should be opened inside a WebView within the app with the injected JavaScript (Eg: see [here](https://facebook.github.io/react-native/docs/webview.html#injectedjavascript)) from the response. After the user pairs with Masterpass on the WebView, if the URL contains `pairingCancel` or `pairingFail`, then the pairing operation has not been completed and the app must navigate the user away from the WebView. If the URL contains `pairingSuccess`, then the pairing has been completed successfully.

<pre>
{
	success: true,
	injectedJavaScript: string, to be injected into the WebView on the app,
}
</pre>

### Precheckout request

`http://<server>/api/1.0/wirecard/precheckoutRequest/<brandCode>`

#### POST Parameters:
<ul><li>authToken: string</li>
<li>amount: decimal number</li>
<li>deviceToken: device token for push notifications</li></ul>

#### Response:

The response provides a base64 encoded wallet data (sample [here](http://docs.elastic-payments.com/?s=wirecardapac#alternative-payments-masterpass)) which can be used to display cards and shipping address options to the customer in the app. The selected information can be used in the place order request to pay using Masterpass. If the precheckout request fails, redirect the user to pair with their wallet again. The precheckout response cannot be used again (after using it in a transaction) or stored within the app - it should be requested each time before displaying the card details to customer.

<pre>
{
	success: true,
	transactionId: string,
	requestId: string,
	injectedJavaScript: string, to be injected into	the webview on the app,
	providerRef: string, use while placing order,
	walletData: string, decode using base64,
}
</pre>

### Masterpass Express Checkout Payment

Apps should pass in `masterpassPayment` object in the Order JSON in the POST body of the Posting an Order API to process payment using Express Checkout. Passing in a test order will not process payment through this method. If this object is missing or null, Alpha considers the customer does not intend to pay with Masterpass and proceeds to process the order.

#### Masterpass Payment Object

<pre>
{
	pin: string, optional 6 digit pin for verification,
	transactionId: string, transaction Id from the precheckout API response,
	cardId: string, id of selected card,
	addressId: string, id of selected address,
	providerRef: string, from the precheckout API response,
	cardType: string, card type information from the encoded wallet data (BrandName element),
	deviceToken: string, device token used for push notifications
}
</pre>