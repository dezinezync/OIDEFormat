# Calculations

Due to the nature of invoices and the number of items involved like taxes, discounts, different tax slabs, calculating the amount of an invoice can be confusing. This documentation aids in understanding the processes to employ when using the OIDE Format.

### Data
For all calculations mentioned below, the data from [this](https://github.com/dezinezync/OIDEFormat/blob/master/src/data.json) file is used. 

## Gross Total of items
To calculate the gross total of the items of the invoice, simply `SUM(value * amount)` of all the items. From our example, this comes to: **950**. Since the units of both items is `INR`, we can safely display `₹ 950.00`. 

However, say the first item was in USD and the second in INR, in this case, based on the user's locale you convert the non-conforming currency to the user locale's currency. 

In this example, we would convert the USD value to INR (using a standard currency exchange value) and perform the `SUM` computation. 

In our example, the `Shipping & Handling` item is excluded from taxes, so we skip it from the above `SUM` calculation. If it wasn't tax exempt, we would include it. 

## Tax calcuations
When one or multiple taxes are listed without a `taxIndex` key, all taxes are computed directly on the above total. For 2 taxes:

```
TAX_AMOUNT_A = TOTAL * TAX_A / 100
TAX_AMOUNT_B = TOTAL * TAX_B / 100

TOTAL_TAXES = TAX_AMOUNT_A + TAX_AMOUNT_B
```

In our example however, we have 2 tax slabs. Each item mentions which tax index it should use. So our calulation becomes:

```
TAX_AMOUNT_A = ITEM_A_AMOUNT * TAX_A / 100
TAX_AMOUNT_B = ITEM_B_AMOUNT * TAX_B / 100

TOTAL_TAXES = TAX_AMOUNT_A + TAX_AMOUNT_B
```

Using the above logic, we get:

```
TA_A = 400 * 5 / 100
TA_B = 450 * 15 / 100

TOTAL_TAXES = 20 + 67.5 = 87.50
```

## Discounts

There's also a discount included with the `beforeTaxes` key set to `false`. If this were true, you'd apply the discount first and then compute the taxes. This is explained further in [this](https://github.com/dezinezync/OIDEFormat/tree/master/calculations#simpler) example.

At this point, our total is **1037.50**. Let's compute the discount next. It's a direct percent. This can also be a currency with it's associated code. Then the amount can be directly subtracted. 

All negative taxes are to be treated as discounts.

```
DISCOUNT = TAXED_TOTAL * DISCOUNT / 100
``` 
Which gives us `881.875`. For display purposes, we can round it off to `881.88`, a palindrome. 

Finally, we go back to our items, and pick up all items which are to be procssed post taxes & discounts. 

## Finalising

So we add the *Shipping & Handling Fee* which gives us the total of: `931.88`. 

This is the printable total. So the full calculation looks like:

```js
const itemTotal = (200 * 2) + 450 = 850

const taxes = (itemATotal * (taxA / 100)) + (itemBTotal * (taxB / 100))

const taxedTotal = itemTotal + taxes

const discount = taxedTotal * (discountPercent/100)

const discountedTotal = taxedTotal - discount

const postTaxesTotal = shippingHandling

const total = discountedTotal + postTaxesTotal
```

# Simpler
The above example shows a lot of the permutations possible in the OIDE Format. Let's take a look at a simpler example:

```json
{
  "items": [
    {
      "title": "200g chocochip Cookies",
      "quantity": 2,
      "rate": {
        "value": 200,
        "unit": "currency",
        "code": "INR",
        "taxExclude": false
      }
    },
    {
      "title": "500g oatmeal Cookies",
      "quantity": 1,
      "rate": {
        "value": 450,
        "unit": "currency",
        "code": "INR",
        "taxExclude": false
      }
    },
    {
      "title": "Shipping & Handling",
      "quantity": 1,
      "rate": {
        "value": 50,
        "unit": "currency",
        "code": "INR"
      }
    }
  ],
  "taxes": [
    {
      "title": "GST",
      "rate": 5,
      "index": 1
    },
    {
      "title": "Friends & Family Discount",
      "rate": {
        "value": -120,
        "unit": "currency",
        "code": "INR"
      }
    }
  ]
}
```

***Note:*** The above isn't a valid OIDE Payload. The payload has been reduced for example purposes only. 

The above example is fairly straight forward. Most use cases should look like this. Let's compute the total for the above example. 

```js
const itemsTotal = 400 + 450 + 50 - 120 = 880

const taxes = (880 * 0.05)              = 44

const taxedTotal = itemsTotal + taxes   = 924

const total = taxedTotal                = 924

```

and that's about it. 
