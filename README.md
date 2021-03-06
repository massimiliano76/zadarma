# zadarma

![Zadarma Nodes.js](https://raw.githubusercontent.com/gravitymir/zadarma/master/zadarma_node.jpeg)

Module which help you work with API Zadarma (v1)

## Requirements:

* Node.js > 14.0.0

## How to use?

An official documentation on Zadarma API is [here](https://zadarma.com/support/api/).

## Getting Started

#### Install

``` shell
#npm
npm install zadarma
```

## Authorization keys

Page authorization keys: [here](https://my.zadarma.com/api/#)

## Usage examples

``` js
const { api } = require("zadarma");
```

#### single account use

``` js
//Example configure default config
process.env.ZADARMA_USER_KEY = 'a248a6a984459935b569';//your user key
process.env.ZADARMA_SECRET_KEY = '8a8e91d214fb728889c7';//your secret key

const { api } = require("zadarma");
  let tariff = await api({api_method: '/v1/tariff/'});
  let balance = await api({api_method: '/v1/info/balance/'});

  console.log(tariff);
  console.log(balance);
```

#### multi account use

``` js
//Example with send "api_user_key" && "api_secret_key"
const { api: z_api } = require("zadarma");
let response = await z_api({
    api_method: '/v1/direct_numbers/',
    api_user_key: 'a248a6a984459935b569', //your user key
    api_secret_key: '8a8e91d214fb728889c7' //your secret key
});
console.log(response);
```

``` js
const { api: z_api } = require("zadarma");

let method = '/v1/pbx/internal/';
let user_key = 'your_user_key';
let secret_key = 'your_secret_key';

let response = await z_api({
    api_method: method,
    api_user_key: user_key,
    api_secret_key: secret_key
});
console.log(response);
```

#### parameters

``` js
//Example with parameters
let response = await z_api({
    api_method: '/v1/request/callback/',
    params: {
        from: '73919100000',
        to: '67200000000',
        sip: '100',
        predicted: 'predicted'
    }
});
console.log(response);
```

#### http_method "post"

``` js
//Example with http_method "post" for api_method "/v1/sms/send/"
let from = '67200000000'; //[optional] your verified phone number
let to = '67200000000';
let message = 'test sms 0987654321\nтестовый текст';

let response = await z_api({
    http_method: 'POST',
    api_method: '/v1/sms/send/',
    params: {
        caller_id: from, //[optional]
        number: to,
        message: message
    }
});
console.log(response);
```

#### zcrm methods examples

``` js
//example get all customers
let response = await z_api({
    api_method: '/v1/zcrm/customers'
});
console.log(response);
```

``` js
//example create customer
let response = await z_api({
    http_method: 'POST',
    api_method: '/v1/zcrm/customers',
    params: {
        customer: {
            "name": "Good company 32",
            "status": "company",
            "type": "client",
            "responsible_user_id": "",
            "employees_count": "50",
            "comment": "",
            "country": "GB",
            "city": "London",
            "address": "",
            "zip": "",
            "website": "",
            "lead_source": "manual",
            "phones": [{
                "type": "work",
                "phone": "+44123456100"
            }],
            "contacts": [{
                "type": "email_work",
                "value": "good_company@example.com"
            }],
            "labels": [],
            "custom_properties": []
        }
    }
});
console.log(response);
```

``` js
//example create, get, delete customers

//create labal 1
let response = await z_api({
    http_method: 'POST',
    api_method: '/v1/zcrm/customers/labels',
    params: {
        name: 'label 1'
    }
});

//create label 2
response = await z_api({
    http_method: 'POST',
    api_method: '/v1/zcrm/customers/labels',
    params: {
        name: 'label 2'
    }
});

//get all labels
response = await z_api({
    api_method: '/v1/zcrm/customers/labels'
});
console.log(response.data.labels);

for (label of response.data.labels) {
    //delete label
    response = await z_api({
        http_method: 'DELETE',
        api_method: `/v1/zcrm/customers/labels/${label.id}`
    });
    console.log( `label id ${label.id} deleted!` );
}

//get all labels
response = await z_api({
    api_method: '/v1/zcrm/customers/labels'
});
console.log(response.data.labels);
```

# Handle event requests with Express

[API settings and description page](https://zadarma.com/support/api/)

[Set link to your hendler server on API description page](https://my.zadarma.com/api/#apitab-zcrm)


### Evenst

1. NOTIFY_START - start incoming call
1. NOTIFY_INTERNAL - dialing up to the internal on incoming call
1. NOTIFY_ANSWER - the internal answered theincoming call
1. NOTIFY_END - end incoming call
1. NOTIFY_OUT_START
1. NOTIFY_OUT_END
1. NOTIFY_RECORD - call recording is being prepared for download
1. NOTIFY_IVR
1. SPEECH_RECOGNITION
1. NUMBER_LOOKUP
1. CALL_TRACKING
1. SMS - incoming sms


```js
const { zadarma_express_handler } = require("zadarma");

const express = require('express');

const app = express();

zadarma_express_handler.on('NOTIFY_START', function(response){
  console.log(response);
});
zadarma_express_handler.on('NOTIFY_END', function(response){
  let all_calls_before_clear_storage = zadarma_express_handler.get_temporary_storage();
  zadarma_express_handler.clear_temporary_storage();
  console.log(response);
});
zadarma_express_handler.on('NOTIFY_OUT_START', function(response){
  console.log(response);
});
zadarma_express_handler.on('NOTIFY_OUT_END', function(response){
  zadarma_express_handler.clear_temporary_storage(response.pbx_call_id);
  console.log(response);
});
zadarma_express_handler.on('NOTIFY_RECORD', function(response){
  console.log(response);
});
zadarma_express_handler.on('SMS', function(response){
  console.log(response);
});

//set api key or default your_api_key = process.env.ZADARMA_SECRET_KEY
zadarma_express_handler.set_api_secret_key(your_api_key);

app.use('/zadarma', zadarma_express_handler);
app.listen(3000);
````