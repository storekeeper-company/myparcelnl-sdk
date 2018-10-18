# MyParcel SDK
This SDK connects to the MyParcel API using PHP.

## Contents

- [Contents](#contents)
- [Installation](#installation)
    - [Requirements](#requirements)
    - [Installation with Composer](#installation-with-composer)
    - [Installation without Composer](#installation-without-composer)
- [Quick start and examples](#quick-start-and-examples)
    - [Create a consignment](#create-a-consignment)
    - [Create multiple consignments](#create-multiple-consignments)
    - [Label format and position](#label-format-and-position)
    - [Package type and options](#package-type-and-options)
    - [Retrieve data from a consignment](#retrieve-data-from-a-consignment)
    - [Create and download label(s)](#create-and-download-label-s-)
- [List of classes and their methods](#list-of-classes-and-their-methods)
    - [Models](#models)
    - [Helpers](#helpers)
- [Contribute](#contribute)

## Installation

### Requirements
The MyParcel SDK works with PHP versions 5.4, 5.6 and 7.x. The [PHP cURL extension](http://php.net/manual/en/book.curl.php) needs to be installed.

### Installation with Composer
This SDK uses Composer. Composer is a tool for dependency management in PHP. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you. For more information on how to use/install composer, please visit https://getcomposer.org/

To install the MyParcel SDK into your project, simply use
```
$ composer require myparcelnl/sdk
```
	
### Installation without Composer
It's also possible to use the SDK without installing it with Composer.

You can download the zip on the project's [releases page](https://github.com/myparcelnl/sdk/releases).

1. Download the package (SDKvx.x.x.zip).
2. Extract the downloaded .zip file and upload the vendor directory to your server.
3. Require `src/AutoLoader.php`
4. You can now use the SDK in your project!

## Quick start and examples
Add the following lines to your project to import the SDK classes for creating shipments.

```php
use MyParcelNL\Sdk\src\Helper\MyParcelCollection;
use MyParcelNL\Sdk\src\Model\Repository\MyParcelConsignmentRepository;
```

### Create a consignment
This example uses only the required methods to create a shipment and download its label.

```php
$consignment = (new MyParcelConsignmentRepository())
    ->setApiKey('api_key_from_MyParcel_backoffice')
    ->setReferenceId('Order 146')
    ->setCountry('NL')
    ->setPerson('Piet Hier')
    ->setFullStreet('Plein 1945 55b')
    ->setPostalCode('2231JE')
    ->setCity('Amsterdam')
    ->setEmail('piet.hier@test.nl');
    
$myParcelCollection = (new MyParcelCollection())
    ->addConsignment($consignment)
    ->setPdfOfLabels()
    ->downloadPdfOfLabels();
```

### Create multiple consignments
This example creates multiple consignments by adding them to one ```MyParcelCollection()``` and then creates and downloads one PDF with all labels.

```php
$myParcelCollection = new MyParcelCollection(); // Note: Create the collection before the loop

// Loop through your shipments, adding each to the same MyParcelCollection()
foreach ($your_shipments as $your_shipment) {

    $consignment = (new MyParcelConsignmentRepository())
        ->setApiKey('api_key_from_MyParcel_backoffice')
        ->setReferenceId($your_shipment['reference_id']) // Note: Make sure every shipment gets a unique reference ID
        ->setPerson($your_shipment['name'])
        ->setPostalCode($your_shipment['postal_code'])
        ->setFullStreet($your_shipment['full_street']) 
        ->setCity($your_shipment['city']);
        
    $myParcelCollection
        ->setUserAgent('name_of_cms', '1.0')
        ->addConsignment($consignment);
}
```

### Label format and position
Choose to output the label as either A4 or A6 when creating a pdf or download link with the argument `$positions` of `setPdfOfLabels($positions)` and `setLinkOfLabels($positions)`.

Example values for `$positions`:
```
A4:            A6:
┏━━━━━┳━━━━━┓  ┏━━━━━┓
┃  1  ┃  2  ┃  ┃  x  ┃
┣━━━━━╋━━━━━┫  ┗━━━━━┛
┃  3  ┃  4  ┃
┗━━━━━┻━━━━━┛  
```
1. `1`: Default value. Outputs A4, starting at top left position.
1. `false`: Outputs at A6 format
1. `[1,4]`: Defines the position of labels on an A4 sheet. Only applies to the first page, subsequent pages will use the default positioning (1,2,3,4)

More information: https://myparcelnl.github.io/api/#6_F

### Package type and options
Set package type with `setPackageType($type)`. Retrieve it after with `getPackageType()`. For more details on the different types of packages: https://myparcelnl.github.io/api/#6_A_1

#### 1: Package
This is the default package type. It must be explicitly set to allow enabling of further shipment options. It's available for NL, EU and global shipments.

#### 2: Mailbox package
This package type is only available for NL shipments that fit into a mailbox. It does not support additional options.
Note: If you still make the request with additional options, bear in mind that you need to pay more than is necessary!

#### 3: Letter 
This package type is available for NL, EU and global shipments. The label for this shipment is unpaid meaning that you will need to pay the postal office/courier to send this letter/package. Therefore, it does not support additional options.

#### Package options
These options are only available for package type 1 (package).

Available options:
- only_recipient: Deliver the package only at address of the intended recipient. This option is required for Morning and Evening delivery types.
  - Set: `setOnlyRecipient(true)`
  - Get: `isOnlyRecipient()`
- signature: Recipient must sign for the package. This option is required for Pickup and Pickup express delivery types.
  - Set: `setSignature(true)`
  - Get: `isSignature()`
- return: Return the package to the sender when the recipient is not home.
  - Set: `setReturn(true)`
  - Get: `isReturn()`
- large_format: This option must be specified if the dimensions of the package are between 100 x 70 x 50 and 175 x 78 x 58 cm. If the scanned dimensions from the carrier indicate that this package is large format and it has not been specified then it will be added to the shipment in the billing process. This option is also available for EU shipments. 
  - Set: `setLargeFormat(true)`
  - Get: `isLargeFormat()`
- insurance: This option allows a shipment to be insured up to certain amount. NL shipments can be insured for 5000,- euros. EU shipments must be insured for 500,- euros. Global shipments must be insured for 200,- euros. The following shipment options are mandatory when insuring an NL shipment: only_recipient and signature.
  - Set: `setInsurance($amount)`
  - Get: `getInsurance()`

More information: https://myparcelnl.github.io/api/#6_A_3

### Retrieve data from a consignment
Most attributes that have a set...() method also have a get...() method to retrieve the data. View [all methods](#myparcelconsignment) for consignments here. 
```php
$consignment = new MyParcelConsignmentRepository();

echo $consignment->getFullStreet();
echo $consignment->getPerson();
echo $consignment->get();
echo $consignment->getStreet();
// etc...
```

### Create and download label(s)
Create and directly download PDF with `setPdfOfLabels($position)` where `$positions` is the [label position](#label-format-and-position) value. 
```php
$myParcelCollection
    ->setPdfOfLabels()
    ->downloadPdfOfLabels(false); // Opens pdf "inline" by default, pass false as argument to download file  
```

Create and echo download link to PDF with `setLinkOfLabels($position)` where `$positions` is the [label position](#label-format-and-position) value.
```php
echo $myParcelCollection 
    ->setLinkOfLabels($positions)
    ->getLinkOfLabels();
```

More information: https://myparcelnl.github.io/api/#6_F

## List of classes and their methods
This is a list of all the classes in this SDK and their available methods.

### Models
`\MyParcelNL\Sdk\src\Model`

#### MyParcelConsignment
IMPORTANT! Access these methods through `Repository\MyParcelConsignmentRepository`, which extends `MyParcelConsignment`.

```\MyParcelNL\Sdk\src\Model\MyParcelConsignment.php```
```php
$consignment = (new \MyParcelNL\Sdk\src\Model\Repository\MyParcelConsignmentRepository())
    ->setApiKey('api_key_from_MyParcel_backoffice')
    ->setReferenceId('Order 1203')
    
    // Recipient/address: https://myparcelnl.github.io/api/#7_B
    ->setPerson('Piet Hier')    // Name
    ->setEmail('test@test.nl')  // E-mail address
    ->setPhone('+31 612345678') // Phone number
    ->setCompany('Piet BV')     // Company
    
    ->setFullStreet('Plein 1945 55b')   // Street, number and suffix in one line 
    // OR send the street data separately:
    ->setStreet('Plein 1945')           // Street
    ->setNumber((string)55)             // Number
    ->setNumberSuffix('b')              // Suffix
    
    ->setCity('Amsterdam')      // City
    ->setPostalCode('2231JE')   // Postal code
    ->setCountry('NL')          // Country                
            
    // Available package types:
    // 1: Package (default)
    // 2: Mailbox package
    // 3: Letter
    ->setPackageType(1)

    // Options (https://myparcelnl.github.io/api/#6_A_3)
    ->setOnlyRecipient(false)   // Deliver the package only at address of the intended recipient. This option is required for Morning and Evening delivery types.
    ->setSignature(true)        // Recipient must sign for the package. This option is required for Pickup and Pickup express delivery types. 
    ->setReturn(true)           // Return the package to the sender when the recipient is not home.
    ->setLargeFormat(false)     // Must be specified if the dimensions of the package are between 100x70x50 and 175x78x58 cm. 
    ->setInsurance(250)         // Allows a shipment to be insured up to certain amount. Only packages (package type 1) can be insured. 
    
    ->setLabelDescription('Order 10034') // This description will appear on the shipment label for non-return shipments. 
        
    // Delivery: https://myparcelnl.github.io/api/#8
    ->setDeliveryType()
    ->setDeliveryDate()
    ->setDeliveryRemark()    
        
    // Set pickup location
    ->setPickupLocationName()
    ->setPickupStreet()
    ->setPickupNumber()
    ->setPickupPostalCode()
    ->setPickupCity()
   
    // Non-EU shipment attributes: see https://myparcelnl.github.io/api/#7_E
    ->setInvoice()
    ->setContents()
    ->addItem()
    
    // Other attributes
    ->setBarcode()
    ->setMyParcelConsignmentId()
    ->setShopId() 
    ->setStatus();

// Get attributes from consignment
$consignment
    ->getApiKey()
    ->getReferenceId()
    ->getBarcode() // Barcode is available after using setLinkOfLabels() or setPdfOfLabels() on the MyParcelCollection the consignment has been added to
    
    ->getLabelDescription()
    ->getMyParcelConsignmentId()
    ->getShopId()
    ->getStatus()
    
    // Recipient info
    ->getPerson()
    ->getEmail()    
    ->getPhone()
    ->getCompany()

    // It doesn't matter whether you used setFullStreet() or set all parts separately
    ->getStreet()
    ->getStreetAdditionalInfo()
    ->getNumber()
    ->getNumberSuffix()
    ->getPostalCode()
    ->getCity()
    ->getCountry()
        
    // Package type
    ->getPackageType()
    
    // Get value of options
    ->isOnlyRecipient()
    ->isSignature()
    ->isReturn()
    ->isLargeFormat()
    ->getInsurance()
        
    // Get pickup location info
    ->getPickupLocationName()
    ->getPickupStreet()
    ->getPickupNumber()
    ->getPickupPostalCode()
    ->getPickupCity()
    
    // Delivery
    ->getDeliveryDate()
    ->getDeliveryRemark()
    ->getDeliveryType()
    
    // Non-EU attributes
    ->getInvoice()
    ->getContents()
    ->getItems();
```

#### MyParcelCustomsItem
This object is embedded in the MyParcelConsignment object for global shipments and is mandatory for non-EU shipments.

```\MyParcelNL\Sdk\src\Model\MyParcelCustomsItem.php```
```php
  ->setAmount()
  ->setClassification()
  ->setCountry()
  ->setDescription()
  ->setItemValue()
  ->setWeight()
  
  ->getAmount()
  ->getClassification()
  ->getCountry()
  ->getDescription()
  ->getItemValue()
  ->getWeight()
  
  ->isFullyFilledItem()
```

#### MyParcelRequest
This model represents one request

```\MyParcelNL\Sdk\src\Model\MyParcelRequest.php```
```php
  ->sendRequest()
  ->setRequestParameters()
  ->setUserAgent()

  ->getError()
  ->getResult()
  ->getUserAgent()
  ->getUserAgentFromComposer()
```

#### Repository\MyParcelConsignmentRepository
The repository of a MyParcel consignment.

```\MyParcelNL\Sdk\src\Model\Repository\MyParcelConsignmentRepository.php```
```php
  ->setDeliveryDateFromCheckout()
  ->setFullStreet()
  ->setPickupAddressFromCheckout()

  ->apiDecode()
  ->apiEncode()
  ->encodeReturnShipment()
  
  ->getDeliveryTypeFromCheckout()
  ->getFullStreet()
  ->getTotalWeight()
  ->isCdCountry()
  ->isCorrectAddress()
  ->isEuCountry()
```

### Helpers

#### MyParcelCollection
Stores all data to communicate with the MyParcel API.

```\MyParcelNL\Sdk\src\Helper\MyParcelCollection.php```
```php
    ->addConsignment() // Add consignment to collection

    // Get consignments from the collection
    ->getConsignments()
    ->getConsignmentByApiId()
    ->getConsignmentByReferenceId()

    ->clearConsignmentsCollection() // Clear the collection
    
    ->createConcepts() // Create concept shipment in the MyParcel Backoffice
    ->deleteConcepts()
    
    ->getOneConsignment()
    ->getRequestBody()
    
    ->sendReturnLabelMails() // Send return label to customer. The customer can pay and download the label
    ->setLatestData() // Set id and run this function to update all the information about this shipment
    ->setLatestDataWithoutIds()
    
    ->setLinkOfLabels()
    ->getLinkOfLabels()

    // Refer to 
    ->setPdfOfLabels()
    ->downloadPdfOfLabels()
    
    // To give us insight into which CMS system you're connecting from, you should send a User-Agent. 
    // If you're using a known CMS system it's required. 
    // You must send the name of the CMS system followed by a version number.
    ->setUserAgent('name_of_cms', '1.0')
    ->getUserAgent()
```

#### MyParcelCurl
Curl to use in the MyParcel API.

```\MyParcelNL\Sdk\src\Helper\MyParcelCurl.php```
```php
  ->addOption()
  ->addOptions()
  ->close()
  ->connect()
  ->getErrno()
  ->getError()
  ->getInfo()
  ->multiRequest()
  ->read()
  ->setConfig()
  ->setOptions()
  ->setUserAgent()
  ->write()
```


## Contribute

1. Check for open issues or open a new issue to start a discussion around a bug or feature.
1. Fork the repository on GitHub to start making your changes.
1. Write one or more tests for the new feature or that expose the bug.
1. Make code changes to implement the feature or fix the bug.
1. Send a pull request to get your changes merged and published.
