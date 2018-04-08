# Open Invoice Data Exchange (Format)
The OIDE Format (Open Invoice Data Exchange) is a JSON structure specficiation developed to facilitate simpler sharing of invoice information across applications, both producers & consumers. 

## Open 
- Developed openly
- Discussed openly
- Exchanged openly

## Data
The data structure chosen for the purpose is JSON  simply due it's wide adoption, interoperability and support across a wide variety of systems. 

The recommended format is: 

```json
{
    "invoiceID": "bb94e6e8-99c4-4e97-ba1a-1fbfb2620ebf",
    "title": "",
    "number": "DZ-1819-0560",
    "timestamp": "2018-04-01T00:00:00+05:30",
    "due": "2018-04-15T23:59:59+05:30",
    "items": [{
        "title": "200g chocochip Cookies",
        "quantity": 2,
        "rate": {
            "value": 200.00,
            "unit": "currency",
            "code": "INR",
            "taxExclude": false
        }
    }, {
        "title": "500g oatmeal Cookies",
        "quantity": 1,
        "rate": {
            "value": 450.00,
            "unit": "currency",
            "code": "INR",
            "taxExclude": false
        }
    }, {
        "title": "Shipping & Handling",
        "quantity": 1,
        "rate": {
            "value": 50.00,
            "unit": "currency",
            "code": "INR",
            "taxExclude": true
        }
    }],
    "taxes": [{
        "title": "SGST",
        "rate": 2.5
    }, {
        "title": "CGST",
        "rate": 2.5
    }, {
        "title": "Friends & Family Discount",
        "rate": -15
    }],
    "payments": [{
        "value": 801.13,
        "unit": "currency",
        "code": "INR"
    }],
    "version": "1.0"
}
```

- `invoiceID`: A UUID v4 based string. This should preferrably be generated using a crypto module on the system if one is available. 
- `title`: ***optional*** Title of the invoice. If this key is excluded or is an empty string, and the `number` key is included, it should be used instead. 
- `number`: ***optional*** The invoice number. If both the invoice number and title are excluded or are empty strings, the entire payload should be treated as invalid. 
- `timestamp`: An `ISO 8601` formatted date string of when the invoice was generated.
- `due`: ***optional*** The due date for the invoice. If one is not included, the invoice should be treated like an *open invoice*.
- `items`: An array of item objects. Should always contain atleast one item.
- `taxes`: ***optional*** An array of tax objects. If no taxes are applicable, this key can be skipped or be passed as an empty array. The array can also include discount items.
- `payments`: ***optional*** An array of payment objects. If no payments have been made or not applicable, this key can be skipped or be passed as an empty array. 
- `version`: The default value for this is the version number of this specification. Vendors can optionally choose to use their own version number as long as it follows the [semantic versioning](https://semver.org) system.

### Item Object
- `title`: The title of the item. 
- `quantity`: A Number indicating the quantity of the above item. Display providers can optionally choose to skip the quantity if the value is 1. 
- `rate`: A number or numerical object which determines the rate of the item. If a numerical object is provided, it should contain information regarding the `currency` and can optionally provide the `taxExclude` key (`false` by default) to include or exclude the item from the taxes computation. 

### Tax Object
- `title`: The title of the tax item
- `rate`: The rate of the item as a float. The value of this key is always assumed to be a percent. This can optionally be negative to denote a *discount*.

### Payment Object
- `value`: The value of the payment
- `unit`: ***optional*** Since the payment unit will always be that of currency, it can be excluded, however for semantic markup, it is recommended that you include it. 
- `code`: The currency code conforming to the ISO 4217 spec.

## Exchange?
The exchange is required so that a single producer can provide the same information to multiple consumers or vice-versa.   
**Example**  
**Producer**: A invoicing app  
**Consumer 1**: Taxation app  
**Conumser 2**: Accounting app  
**Consumer 3**: Consolidation app (automatically tracking receipt of payments with banks)

The reverse is also true. Missing information in the Producer (perhaps due to service denial for unavoidable reasons) can be provided by the consumers.   

*Treating **producer** and **consumer** as loose terms is highly recommended. The data provider is usually considered as a producer. Due to interoperability, either application can be a producer and an application can also simultanesouly be a producer and consumer*

## Data integrity
With exchange of financial information, data integrity validation is absolutely critical. Ergo, when any data exchange is happening, **Asymmetric Cryptographical** keys are employed. 

Any application participating in OIDE should generate it's Asymmetric keypair.

To generate a keypair using `openssl`:  

```sh
openssl genrsa -out private.pem 1024
openssl rsa -in private.pem -out public.pem -pubout
```

The privaye key of the producer should be used to sign all OIDE packages by the producer to generate a signature. The producer should make it's public key accessible to external consumers. The mechanisms to expose public keys are discussed here.   

The consumer should use the respective public key to verify the signature and then proceed to use the package provided to it.  

This enables both parties involved in the exchange to not only validate that the package isn't modified in-transit, but also to do so securely over established mechanisms. 

An example of the above transaction would look something like so:  

```sh  
# Producer  
openssl dgst -sha256 -sign private.pem -out signature package.txt

# Consumer 
openssl dgst -sha256 -verify public.pem -signature signature package.txt
```

In the above example, `package.txt` is the stringified version of a valid JSON structure conforming to this spec. 

## Print
The OIDE format also enables embedding Invoices in the print format by means of QR-Codes along with full formatted text representation. QR codes are easily printable, scannable and existing systems can easily adopt to this system. 

To generate a QR code for the above payload:   
1. Stringify the JSON payload
2. Generate the signature using your private key. 
3. Contact the two strings as follows: `oide::[signature]::[payload]`. From the above sample, the signature's base64 encoded format should be used.

The resulting string for the sample payload is as follows:  

```
oide::A9hh1eBFKEcLIc23rBy9S9AzOik0Aq/ErmcjmRdIV/kcwGtz80W2SHRU82rVhAoqBlWu2tM1VRTs34o3aaunAT9KhqLYGVRt3sb7v6IWQIjphwCm45jRA2FSNX0BI7H3SP1weqH6+jyIELJ8VGTTm6NNwUDxe2F8yYla49RmNgzZY8mXa3xHfjokPOKh6ZNux+xT3R8SIK/H9HUx2YiFGbQBvx20XS9XoJbxWxCTbzX4grb81FPMViE+66vPFnkzto4mLLPitX2/dzcCJKBdiH+GDBKb0z052xrZ/fyuoAU9Q47CbbZFOaM9ANYx/WuMdxWDbRNw5y0nVQF7+hixJDjZw+pfpqBH0uXMuWAKI1YC+tAJ+dRRKHTaoQFoGtZSYonSsfmAGgG9m8ot98X8EeYdtiqHIcL9PQtiaUV/zQ/4gZqaW1O8+wnH5AIZFIx8LCfn2jcxWnxusIUJ7BRYkX/yulSCTHmE42K403p1YYV2i6DXbqIBolk+5dAUfnyn+avgmGN+mR5JqqQaW22xtdrw9ukh1ZOu2tS3n0jP584+/JIh0AJJpIWSsZhnXK3aib8ynPY+7SRed3o5khf6w9h0fFQ9XWR08ektNGyj3OkJX3RQ0ANbNcM9w+OA0CpX8RE2rouvN8sHrB7RC2vgy1YThD77StCml2hVsXSWRJM=::{"invoiceID":"bb94e6e8-99c4-4e97-ba1a-1fbfb2620ebf","title":"","number":"DZ-1819-0560","timestamp":"2018-04-01T00:00:00+05:30","due":"2018-04-15T23:59:59+05:30","items":[{"title":"200g chocochip Cookies","quantity":2,"rate":{"value":200,"unit":"currency","code":"INR","taxExclude":false}},{"title":"500g oatmeal Cookies","quantity":1,"rate":{"value":450,"unit":"currency","code":"INR","taxExclude":false}},{"title":"Shipping & Handling","quantity":1,"rate":{"value":50,"unit":"currency","code":"INR","taxExclude":true}}],"taxes":[{"title":"SGST","rate":2.5},{"title":"CGST","rate":2.5},{"title":"Friends & Family Discount","rate":-15}],"payments":[{"value":801.13,"unit":"currency","code":"INR"}],"version":"1.0"}
```

The above will generate the following QR Code Image:  
<img src="http://api.qrserver.com/v1/create-qr-code/?color=000000&amp;bgcolor=FFFFFF&amp;data=oide%3A%3AA9hh1eBFKEcLIc23rBy9S9AzOik0Aq%2FErmcjmRdIV%2FkcwGtz80W2SHRU82rVhAoqBlWu2tM1VRTs34o3aaunAT9KhqLYGVRt3sb7v6IWQIjphwCm45jRA2FSNX0BI7H3SP1weqH6%2BjyIELJ8VGTTm6NNwUDxe2F8yYla49RmNgzZY8mXa3xHfjokPOKh6ZNux%2BxT3R8SIK%2FH9HUx2YiFGbQBvx20XS9XoJbxWxCTbzX4grb81FPMViE%2B66vPFnkzto4mLLPitX2%2FdzcCJKBdiH%2BGDBKb0z052xrZ%2FfyuoAU9Q47CbbZFOaM9ANYx%2FWuMdxWDbRNw5y0nVQF7%2BhixJDjZw%2BpfpqBH0uXMuWAKI1YC%2BtAJ%2BdRRKHTaoQFoGtZSYonSsfmAGgG9m8ot98X8EeYdtiqHIcL9PQtiaUV%2FzQ%2F4gZqaW1O8%2BwnH5AIZFIx8LCfn2jcxWnxusIUJ7BRYkX%2FyulSCTHmE42K403p1YYV2i6DXbqIBolk%2B5dAUfnyn%2BavgmGN%2BmR5JqqQaW22xtdrw9ukh1ZOu2tS3n0jP584%2B%2FJIh0AJJpIWSsZhnXK3aib8ynPY%2B7SRed3o5khf6w9h0fFQ9XWR08ektNGyj3OkJX3RQ0ANbNcM9w%2BOA0CpX8RE2rouvN8sHrB7RC2vgy1YThD77StCml2hVsXSWRJM%3D%3A%3A%7B%22invoiceID%22%3A%22bb94e6e8-99c4-4e97-ba1a-1fbfb2620ebf%22%2C%22title%22%3A%22%22%2C%22number%22%3A%22DZ-1819-0560%22%2C%22timestamp%22%3A%222018-04-01T00%3A00%3A00%2B05%3A30%22%2C%22due%22%3A%222018-04-15T23%3A59%3A59%2B05%3A30%22%2C%22items%22%3A%5B%7B%22title%22%3A%22200g+chocochip+Cookies%22%2C%22quantity%22%3A2%2C%22rate%22%3A%7B%22value%22%3A200%2C%22unit%22%3A%22currency%22%2C%22code%22%3A%22INR%22%2C%22taxExclude%22%3Afalse%7D%7D%2C%7B%22title%22%3A%22500g+oatmeal+Cookies%22%2C%22quantity%22%3A1%2C%22rate%22%3A%7B%22value%22%3A450%2C%22unit%22%3A%22currency%22%2C%22code%22%3A%22INR%22%2C%22taxExclude%22%3Afalse%7D%7D%2C%7B%22title%22%3A%22Shipping+%26+Handling%22%2C%22quantity%22%3A1%2C%22rate%22%3A%7B%22value%22%3A50%2C%22unit%22%3A%22currency%22%2C%22code%22%3A%22INR%22%2C%22taxExclude%22%3Atrue%7D%7D%5D%2C%22taxes%22%3A%5B%7B%22title%22%3A%22SGST%22%2C%22rate%22%3A2.5%7D%2C%7B%22title%22%3A%22CGST%22%2C%22rate%22%3A2.5%7D%2C%7B%22title%22%3A%22Friends+%26+Family+Discount%22%2C%22rate%22%3A-15%7D%5D%2C%22payments%22%3A%5B%7B%22value%22%3A801.13%2C%22unit%22%3A%22currency%22%2C%22code%22%3A%22INR%22%7D%5D%2C%22version%22%3A%221.0%22%7D&amp;qzone=1&amp;margin=0&amp;size=300x300&amp;ecc=L" alt="qr code" />

## Units
All units to be used in the JSON structures should be a recognized ISO format or a widely adopted system agreed upon by this specification. This is applicable but not limited to:  
- Dates ([ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html))  
- Units ([Metric System](https://en.wikipedia.org/wiki/International_System_of_Units))  
- Codes ([Countries: ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3), [Currencies ISO 4217](https://en.wikipedia.org/wiki/ISO_4217))

All units and codes should always be used in UPPERCASE, lowercase and MixEdCasE should be avoided completely. 

Further formatting and display of units is up to consideration of the consumer (assumption; see [note]()) and should be done in the end user's locale for optimum accessibility.  

## Numbers
All numbers should be unformatted, and preferabbly floats rounded up to 2 decimal places.   
- The rounding off to decimal places is optional, but should be utilized to produce smaller packages. This should be avoided for high accuracy systems.  
- All numerical values should be pure numbers. Strings should be treated as invalid values and therefore ignored completely.  
- If a numerical value has an associated unit, it should be wrapped as a object, keeping the numercial part strictly observing the above specifications. 
  
```json  
{  
    "value": 3201.38174,  
    "unit": "currency",  
    "code": "INR"  
}
```

## Notes
- All JSON data structures and value types are considered valid unless explicitly mentioned otherwise in the specificiation.  *No exclusions exist at the moment.*
- The RSA Keypair included in this repo should not be used in your application. They for demo purposes only and for validating information mentioned above in the spec. 

### Author
Nikhil Nigade (Dezine Zync Studios)

### Last Modified
2018-04-08T13:28:30+05:30