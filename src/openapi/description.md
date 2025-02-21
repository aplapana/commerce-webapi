# Introduction

The Adobe Commerce and Magento Open Source web API framework provides integrators and developers the means to use web services that communicate with the application. Key features include:

*  Support for [GraphQL](../graphql/), [REST](https://developer.adobe.com/commerce/webapi/rest/) (Representational State Transfer) and [SOAP](soap-web-api-calls.md) (Simple Object Access Protocol).

*  Three types of [authentication](./authentication/index.md):
   *  Third-party applications authenticate with [OAuth 1.0a](./authentication/gs-authentication-oauth.md).
   *  Mobile applications authenticate using [tokens](./authentication/gs-authentication-token.md).
   *  Administrators and customers are authenticated with [login credentials](./authentication/gs-authentication-token.md).

*  All accounts and integrations are assigned resources that they have access to. The API framework checks that any call has the authorization to perform the request.

*  Any native or third-party service can be [configured as a web API](https://developer.adobe.com/commerce/php/development/components/web-api/services/) with a few lines of XML. To configure a web API, you define XML elements and attributes in a `webapi.xml` configuration file. If a service is not defined in a configuration file, it will not be exposed at all.

*  The framework is based on the CRUD (create, read, update, delete) & search model. The system does not currently support webhooks.

*  The framework supports field filtering of web API responses to conserve mobile bandwidth.

*  Integration style web APIs enable a single web API call to run multiple services at once for a more efficient integration.  An example of this behavior can be seen in the Catalog where one web API call can create a product. If your payload includes the `stock_item` and `media_gallery_entries` objects, then the framework will also create the product's inventory & media in that one API call.

## What can I do with the web APIs?

The APIs can be used to perform a wide array of tasks. For example:

*  Create a shopping app. This can be a traditional app that a user downloads on a mobile device. You could also create an app that an employee uses on a showroom floor to help customers make purchases.

*  Integrate with CRM (Customer Relationship Management) or ERP (Enterprise Resource Planning) backend systems, such as Salesforce or Xero.

*  Integrate with a CMS (Content Management System). Currently, content tagging is not supported.

*  Create JavaScript widgets in the storefront or on the Admin panel. The widget makes Ajax calls to access services.

## How do I get started?

You must register a web service on Admin. Use the following general steps to set up Magento to enable web services.

1. If you are using token-based authentication, create a web services user on Admin by selecting **System** > Permission > **All Users** > Add New User. (If you are using session-based or OAuth authentication, you do not need to create the new user in the Admin.)
1. Create a new integration on Admin. To create an integration, click **System** > Extensions > **Integration** > Add New Integration**. Be sure to restrict which resources the integration can access.
1. Use a REST or SOAP client to configure authentication.

See the User Guide for more information.

# Authentication

This topic describes the different forms of authentication that are available in the Commerce API, and how to use them.

## Manage API keys

To create or manage API keys, select one of the following ...

# Status codes and responses

---
title: Status Codes and Responses
description: A list of status codes and REST responses that can be returned from the APIs.
keywords:
  - GraphQL
  - REST
---

# Status codes and REST responses

Each web API call returns a HTTP status code and a response payload. When an error occurs, the response body also returns an error message.

## HTTP status codes

Each web API call returns an HTTP status code that reflects the result of a request:

HTTP code | Meaning | Description
--- | --- | ---
200 | Success | The framework returns HTTP 200 to the caller upon success.
400 | Bad Request | If service implementation throws either `Magento_Service_Exception` or its derivative, the framework returns a HTTP 400 with a error response including the service-specific error code and message. This error code could indicate a problem such as a missing required parameter or the supplied data didn't pass validation.
401 | Unauthorized | The caller was not authorized to perform the request. For example, the request included an invalid token or a user with customer permissions attempted to access an object that requires administrator permissions.
403 | Forbidden | Access is not allowed for reasons that are not covered by error code 401.
404 | Not found | The specified REST endpoint does not exist. The caller can try again.
405 | Not allowed | A request was made of a resource using a method that is not supported by that resource. For example, using GET on a form which requires data to be presented via POST, or using PUT on a read-only resource.
406 | Not acceptable | The requested resource is only capable of generating content that is not acceptable according to the Accept headers sent in the request.
500 | System Errors | If service implementation throws any other exception  like network errors, database communication, framework returns HTTP 500.

## Response payload

POST, PUT, and GET web API calls return a response payload. This payload is a JSON- or XML-formatted response body. The `Accept: application/<FORMAT>` header in the request determines the format of the response body, where `FORMAT` is either `json` or `xml`.

A successful DELETE call returns `true`. An unsuccessful DELETE call returns a payload similar to the other calls.

The response payload depends on the call.
For example, a `GET /V1/customers/:customerId` call returns the following payload:

```json
{
    "customers": {
        "customer": {
            "email": "user@example.com",
            "firstname": "John",
            "lastname": "Doe"
        },
        "addresses": [
            {
                "defaultShipping": true,
                "defaultBilling": true,
                "firstname": "John",
                "lastname": "Doe",
                "region": {
                    "regionCode": "CA",
                    "region": "California",
                    "regionId": 12
                },
                "postcode": "90001",
                "street": ["Zoe Ave"],
                "city": "Los Angeles",
                "telephone": "555-000-00-00",
                "countryId": "US"
            }
        ]
    }
}
```

This JSON-formatted response body includes a `customer` object with the customer email, first name, and last name, and customer address information. The information in this response body shows account information for the specified customer.

## Error format

When an error occurs, the response body contains an error code, error message, and optional parameters.

Part | Description
--- | --- | ---
`code` | The status code representing the error.
`message` | The message explaining the error.
`parameters` | Optional. An array of attributes used to generate a different and/or localized error message for the client.

As an example, the application returns a `code` of `400` and the following `message` when an invalid `sku` value is specified in the call `PUT V1/products/:sku`.

```json
{
  "message": "Invalid product data: %1",
  "parameters": [
    "Invalid attribute set entity type"
  ]
}
```

# Security

The following security options are available:

## Rate limiting

In a carding attack, an attacker tries to determine which credit card numbers are valid, usually in batches of thousands. Attackers can use similar techniques to brute force missing details, such as the expiration date. Adobe Commerce merchants can be targeted by this attack type through their shops and integrations with third-party payment gateways.

As of Adobe Commerce 2.4.7, you can configure rate limiting for the payment information transmitted using REST and GraphQL. This added layer of protection allows merchants to prevent and decrease the volume of carding attacks that test many credit card numbers at once.

The rate limiting functionality affects the following entry points:

REST:

- `<base_url>/rest/V1/<store_code>/guest-carts/<cart_id>/payment-information`
- `<base_url>/rest/V1/<store_code>/guest-carts/<cart_id>/order`
- `<base_url>/rest/V1/<store_code>/carts/mine/payment-information`
- `<base_url>/rest/V1/<store_code>/carts/mine/order`

GraphQL:

- `<base_url>/graphql`

InstantPurchase module:

- `magento/module-instant-purchase`

Setting up rate limiting has two discrete steps:

1. Add a configuration that connects to the service where the request logs will be stored. By default, the connection is configured for a Redis server. Most merchants must configure a cloud or local installation of Redis to implement the rate limiting feature. However, merchants who have [instances hosted on Amazon EC2](#use-aws-elasticache-with-your-ec2-instance) use an AWS ElastiCache instead of a cloud or local Redis instance.

  **Note:** The data, including request time and identifier, is temporarily stored in Redis. Registered users are identified by their user ID. Non-registered users are identified by their external IP address.

1. Configure Commerce to set restrictions on guest users and authenticated customers. This step is the same for all merchants.

### Set the Redis service connection (most merchants)

The [Configure Redis](https://experienceleague.adobe.com/docs/commerce-operations/configuration-guide/cache/redis/config-redis.html) topic in the _Commerce Configuration Guide_ describes basic installation and configuration information.

Commerce provides command-line options to configure the connection to the backpressure logger. Although you can configure the backpressure logger by editing the `<Commerce-install-dir>/app/etc/env.php` file, using the command line is the recommended method, especially for initial configurations. The command line provides validation, ensuring the configuration is syntactically correct.

By default, the connection is configured for a Redis server. Use the following `bin/magento setup:config:set` commands to configure the Redis service connection:

Command line parameter | Value | Description | Default
--- | --- | --- | ---
`--backpressure-logger` | `redis` | Specifies the backpressure logger handler | This value must be set to `redis`.
`--backpressure-logger-id-prefix` | String | ID prefix for keys | None
`--backpressure-logger-redis-db` | Database number | Required if you use Redis for both the default and full-page cache. You must specify the database number of one of the caches; the other cache uses 0 by default.<br/><br/>**Important**: If you use Redis for more than one type of caching, the database numbers must be different. It is recommended that you assign the default caching database number to 0, the page-caching database number to 1, and the session storage database number to 2. And as a result, the number of the database for storing the backpressure log is 3. | `3`
`--backpressure-logger-redis-password` | Password | Configuring a Redis password enables one of its built-in security features: the `auth` command, which requires clients to authenticate to access the database. The password is configured directly in Redis' configuration file: `/etc/redis/redis.conf` | None
`--backpressure-logger-redis-persistent` | String | Unique string to enable persistent connections | None
`--backpressure-logger-redis-port` | Port | Redis server listen port | `6379`
`--backpressure-logger-redis-server` | Server | Fully qualified hostname, IP address, or an absolute path to a UNIX socket. The default value of 127.0.0.1 indicates Redis is installed on the Commerce server. | `127.0.0.1`
`--backpressure-logger-redis-timeout` | Timeout | Redis server timeout, in seconds | 2.5
`--backpressure-logger-redis-user` | User ID | Redis server user | None

The following example creates a new connection to a Redis server with the following properties:

- Host: 195.34.23.5
- Port: 9345
- Password: s0M3StR0NgP@SsW0Rd
- User: SomeUser

```bash
$ bin/magento setup:config:set \
    --backpressure-logger=redis \
    --backpressure-logger-redis-server=195.34.23.5 \
    --backpressure-logger-redis-port=9345 \
    --backpressure-logger-redis-timeout=1 \
    --backpressure-logger-redis-persistent=persistent \
    --backpressure-logger-redis-db=3 \
    --backpressure-logger-redis-password=s0M3StR0NgP@SsW0Rd\
    --backpressure-logger-redis-user=SomeUser \
    --backpressure-logger-id-prefix=some_pref
```

After the command is executed, the following configuration is added to the `app/etc/env.php` file.

```php
[
//...
    'backpressure' => [
        'logger' => [
            'type' => 'redis',
            'options' => [
                'server' => '195.34.23.5',
                'port' => 9345,
                'timeout' => 1,
                'persistent' => 'persistent',
                'db' => '3',
                'password' => 's0meStr0ngPassw0rd',
                'user' => 'SomeUser'
            ],
            'id-prefix' => 'some_pref'
        ]
    ]
//...
];
```

Continue to [Configure rate limiting](#configure-rate-limiting)

### Use AWS ElastiCache with your EC2 instance

As of Commerce 2.4.3, instances hosted on Amazon EC2 can use an AWS ElastiCache in place of a local Redis instance.

<InlineAlert variant="warning" slots="text" />

This section applies to Commerce instances running on Amazon EC2 VPCs. It does not work for standard cloud or on-premises installations.

#### Configure a Redis cluster

After [setting up a Redis cluster on AWS](https://aws.amazon.com/getting-started/hands-on/setting-up-a-redis-cluster-with-amazon-elasticache/), configure the EC2 instance to use the ElastiCache.

1. [Create an ElastiCache Cluster](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/set-up.html) in the same region and VPC of the EC2 instance.

1. Verify the connection.

    - Open an SSH connection to your EC2 instance.

    - On the EC2 instance, install the Redis client:

       ```bash
       sudo apt-get install redis
       ```

    - Add an inbound rule to the EC2 security group: Type `- Custom TCP, port - 6379, Source - 0.0.0.0/0`

    - Add an inbound rule to the ElastiCache Cluster security group: Type `- Custom TCP, port - 6379, Source - 0.0.0.0/0`

    - Connect to the Redis CLI:

       ```bash
       redis-cli -h <ElastiCache Primary Endpoint host> -p <ElastiCache Primary Endpoint port>
       ```

#### Configure Commerce to use the cluster

Run `setup` commands to specify the Redis endpoints.
To configure Commerce for the backpressure logger connection:

```bash
bin/magento setup:config:set --backpressure-logger=redis --backpressure-logger-redis-server=<ElastiCache Primary Endpoint host> --backpressure-logger-redis-port=<ElastiCache Primary Endpoint port> --backpressure-logger-redis-db=3
```

### Configure rate limiting

After the Redis server connection has been configured, you can run several `bin/magento config:set` commands that define how rate limiting is implemented for guest users and authenticated customers. Rate limiting is disabled by default.

| Parameter                      | Description                               |
|--------------------------------|-------------------------------------------|
| sales/backpressure/enabled     | Enable rate limiting for placing orders.  |
| sales/backpressure/disabled    | Disable the rate limiting feature.        |
| sales/backpressure/guest_limit | Requests limit per guest.                 |
| sales/backpressure/limit       | Requests limit per authenticated customer.|
| sales/backpressure/period      | The number of seconds to wait until resetting the counter. |

You can also enable and configure rate limiting from the Admin by selecting **Stores** > **Configuration** > **Sales** > **Sales** > **Rate Limiting**.

![Rate limiting section](../_images/api-security-rate-limiting.png)

Use the following commands to enable and configure rate limiting:

1. Enable (`1`) or disable (`0`) rate limiting for placing orders:

    ```bash
    $ bin/magento config:set sales/backpressure/enabled 1
    ```

1. Set the request limit per guest (IP address).

    ```bash
    $ bin/magento config:set sales/backpressure/guest_limit 50
    ```
  
1. Set the request limit for authenticated customers:

    ```bash
    $ bin/magento config:set sales/backpressure/limit 10
    ```

1. Set the period of time (in seconds) for the request limit. Supported values `60`, `3600`, `86400` seconds. This time period is multiplied by three to calculate the timeout period:

    ```bash
    $ bin/magento config:set sales/backpressure/period 60
    ```

The following scenario describes how rate limiting can be configured.

- Anonymous users are limited to 50 orders (`sales/backpressure/guest_limit` = `50`) from a single IP address within one minute (`sales/backpressure/period` = `60`).  If they exceed the order limit, then they will have to wait three times the specified `period` of time (`180` seconds) from their last request.
- If an authorized customers attempts to place more than `10` orders (`sales/backpressure/limit` = `10`) within the `period` of `60` seconds, then the user will not be able to place an order for a period of `180` seconds.

If you need to check a configuration, use the following CLI command:

Example:

```bash
$ bin/magento config:show | grep backpressure
```

Response:

```terminal
sales/backpressure/limit - 10
sales/backpressure/guest_limit - 50
sales/backpressure/period - 60
sales/backpressure/enabled - 1
```

### Log contents

If rate limiting has been enabled for the payment information endpoint and the GraphQL mutation via the UI/CLI, but the Redis service connection for store log requests has not been configured in the `app/etc/env.php` file, then the rate-limiting will not apply. The behavior will be the same if this option is disabled, but the application logs (`<magento-root>/var/log/system.log`) will contain the following message:

```text
...
[2022-11-11T15:46:32.716679+00:00] main.ERROR: Backpressure sliding window not applied. Invalid request logger type:  [] []
...
[2022-11-11T15:46:37.730863+00:00] main.ERROR: Backpressure sliding window not applied. Invalid request logger type:  [] []
...
```

### Example HTTP Responses

If rate limiting is applied to a REST request, then a response with HTTP status code `429 - Too Many Requests` will be generated.

Example:

```text
HTTP/1.1 429 Too Many Requests
...
Pragma: no-cache
Cache-Control: no-store
...
{"message":"Too Many Requests","trace":null}
```

If rate limiting is applied to a GraphQL request, then a response with HTTP status code `200 - Ok` will be generated and all relevant information will be present in the response body.

Example:

```text
HTTP/1.1 200 OK
...
Pragma: no-cache
Cache-Control: max-age=0, must-revalidate, no-cache, no-store
 ...
{
    "errors":[
        {
            "message":"Too Many Requests",
            "extensions":{"category":"graphql-too-many-requests"},
             "locations":[
                 {"line":2,"column":3}
             ],
             "path":["placeOrder"]
        }
    ],
    "data":{"placeOrder":null}
}
```

## Input limiting

This topic describes best practices for [API security](https://owasp.org/www-project-api-security/).

Imposing restrictions on the size and number of resources that a user can request through an API can help mitigate denial-of-service (DoS) vulnerabilities. By default, the following built-in API rate limiting is available:

-  REST requests containing inputs that represent a list of entities. When enabled, the default maximum is 20 for synchronous requests and 5,000 for asynchronous requests.
-  REST and GraphQL queries that allow paginated results can be limited to a maximum number of items per page. When enabled, the default maximum is 300.
-  REST queries that allow paginated results can have a default number of items per page imposed. When enabled, the default maximum is 20.

By default, these input limits are disabled, but you can use the following methods to enable them:

-  Set the values in the [Admin](https://experienceleague.adobe.com/en/docs/commerce-admin/config/services/magento-web-api).
-  Run the [`bin/magento config:set` command](https://experienceleague.adobe.com/docs/commerce-operations/configuration-guide/cli/configuration-management/set-configuration-values.html).
-  Add entries to the [`env.php` file](https://experienceleague.adobe.com/docs/commerce-operations/configuration-guide/files/config-reference-configphp.html#system).
-  Set [environment variables](https://experienceleague.adobe.com/docs/commerce-operations/configuration-guide/deployment/examples/example-environment-variables.html).

When input limiting has been enabled, the system uses the default value for each limitation listed above. You can also configure custom values.

Although some simple examples for configuring these values from the CLI are provided below, all of the values can be [configured per website and per store view](https://experienceleague.adobe.com/docs/commerce-operations/configuration-guide/cli/configuration-management/set-configuration-values.html) in addition to being configurable globally. In addition, these values can also be configured [via `env.php`](https://experienceleague.adobe.com/docs/commerce-operations/configuration-guide/files/config-reference-configphp.html#system)
as well as via [environment variables](https://experienceleague.adobe.com/docs/commerce-operations/configuration-guide/deployment/examples/example-environment-variables.html).

<InlineAlert variant="info" slots="text"/>

In addition, the Admin provides a configuration setting for limiting session sizes for Admin users and storefront visitors.

### Enable the input limiting system

To enable these input limiting features from the Admin, go to **Stores** > Settings > **Configuration** > **Services** > **Web Api Limits** or **GraphQL Input Limits** and set **Enable Input Limits** to **Yes**.

To enable with the CLI, run one or both of the following commands:

```bash
bin/magento config:set webapi/validation/input_limit_enabled 1
```

```bash
bin/magento config:set graphql/validation/input_limit_enabled 1
```

### Maximum parameter inputs

The `EntityArrayValidator` class constructor limits the number of objects that can be given to inputs that represent arrays of objects. For example, the `PUT /V1/guest-carts/{cartId}/collect-totals` endpoint contains the input parameter `additionalData->extension_attributes->gift_messages`, which represents a list of gift message information objects.

There are four possible input arrays:

-  `additional_data`
-  `agreement_ids`
-  `gift_messages`
-  `custom_attributes`

```json
{
  "paymentMethod": {
    "po_number": "string",
    "method": "string",
    "additional_data": [
      "string"
    ],
    "extension_attributes": {
      "agreement_ids": [
        "string"
      ]
    }
  },
  "shippingCarrierCode": "string",
  "shippingMethodCode": "string",
  "additionalData": {
    "extension_attributes": {
      "gift_messages": [
        {
          "gift_message_id": 0,
          "customer_id": 0,
          "sender": "string",
          "recipient": "string",
          "message": "string",
          "extension_attributes": {
            "entity_id": "string",
            "entity_type": "string",
            "wrapping_id": 0,
            "wrapping_allow_gift_receipt": true,
            "wrapping_add_printed_card": true
          }
        }
      ]
    },
    "custom_attributes": [
      {
        "attribute_code": "string",
        "value": "string"
      }
    ]
  }
}
```

By default, any one of these arrays can include up to 20 items, but you can change this value in the configuration UI via **Stores** > Settings > **Configuration** > **Services** > **Web API Input Limits** > **Input List Limit** or via CLI using the `webapi/validation/complex_array_limit` configuration path.

### Input limit for REST endpoints

Some REST endpoints can contain a high number of elements, and developers need a way to set the limit for each endpoint. The limit for a specific REST endpoint can be set in the `webapi.xml` configuration file for synchronous requests and `webapi_async.xml` for asynchronous requests.
To do this, assign a value for the `<data input-array-size-limit/>` attribute within a `<route>` definition. The value for `input-array-size-limit` must be a non-negative integer.

The following example sets the input limit for the `/V1/some-custom-route` route.
If the route works synchronously, open the `<module_dir>/etc/webapi.xml` configuration file. Otherwise, open `<module_dir>/etc/webapi_async.xml`.
Add the `data` tag with the `input-array-size-limit` attribute to the route configuration.

```xml
<?xml version="1.0"?>
<!--
Some custom module
-->    
<routes xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Webapi:etc/webapi.xsd">
    <route url="/V1/some-custom-route" method="POST">
        <service class="Vendor\Module\Api\EntityRepositoryInterface" method="save"/>
        <resources>
            <resource ref="Vendor_Entity::entities" />
        </resources>
        <data input-array-size-limit="5" /> <!--  limit only for route `/V1/some-custom-route`  -->
    </route>
</routes>
```

Clear the configuration cache for the changes to take effect.

```bash
bin/magento cache:clean config
```

### Values by default for REST endpoints

If you need to change the default limits for REST endpoints, then edit the `webapi` section of the `<magento_root>/app/etc/env.php` file as follows:

```php
[
//...
    'webapi' => [
        'sync' => [
            'default_input_array_size_limit' => <non-negative value>, //overrides values for synchronous REST endpoints
        ],
        'async' => [
            'default_input_array_size_limit' => <non-negative value>, //overrides values for asynchronous REST endpoints
        ],
    ]
//...
];
```

### Maximum page size

The maximum page size setting controls the pagination of various web API responses. By default, the maximum value is `300`. You can change the default in the Admin by selecting **Stores** > Settings > **Configuration** > **Services** > **Web API Input Limits** or **GraphQl Input Limits** >  **Maximum Page Size** field.

[GraphQL security configuration](../graphql/usage/security-configuration.md) describes how to set the maximum page size in GraphQL.

### Default page size

The Default Page Size setting controls the pagination of various web API responses. You can change the default value of `20` in the Admin by selecting **Stores** > Settings > **Configuration** > **Services** > **Web API Input Limits** > **Default Page Size**. To change the value from the CLI, run the following command:

```bash
bin/magento config:set webapi/validation/default_page_size 30
```
