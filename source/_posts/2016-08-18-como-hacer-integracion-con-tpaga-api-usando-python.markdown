---
layout: post
title: "Cómo hacer integración con Tpaga API usando Python"
date: 2016-08-18 08:53:45 -0500
comments: true
categories: [Python, Tpaga, API, requests]
---

[Tpaga](https://tpaga.co/) es una plataforma que permite recibir pagos electrónicos. Tiene una esctuctura sencilla para entender y fácil para usar.

Para obtener nuestros claves de acceso y conectarnos con el API de Tpaga, creamos una cuenta en el "sandbox" de la plataforma: [sandbox.tpaga.co](https://sandbox.tpaga.co/).

{% img /images/tpaga_sandbox_login.png %}

Al registrarnos podemos ver que ahora tenemos dos claves que podemos usar para la autenticación: **Private Api Key** y **Public Api Key**:

{% img /images/tpaga_sandbox_dashboard.png %}

Tpaga tiene unos modelos básicos que nos permitirán organizar nuestros datos: **Customers** (Clientes), **Credit Cards** (Tarjatas de crédito) asociados a los Clientes y **Charges** (Transacciónes o cobros por tarjéta de crédito).

Ahora, cuando entendemos la estructura, podemos empezar a escribir nuestro cliente en Python. Primero instalamos la librería `requests` que nos permiritá hacer peticiones HTTP:

```bash
$ pip install requests
```

Tpaga, como muchos otros sitios web, acepta la autenticación mediante HTTP Basic Auth. La librería `requests` provee una forma fácil de usarla:

```python
>>> import requests
>>> TPAGA_PRIVATE_TOKEN = 'pk_test_qvbvuthlvqpijnr0elmtg5jh'
>>> tpaga_url = 'https://sandbox.tpaga.co/api/customer'
>>> requests.post(tpaga_url, json={}, auth=(TPAGA_PRIVATE_TOKEN, ''))
<Response [201]>
```

Escribimos un cliente sencillo que nos permitirá conectarse al API de Tpaga y mandar peticiones:

```python
from urllib import parse as urlparse
import requests

TPAGA_PRIVATE_TOKEN = 'pk_test_qvbvuthlvqpijnr0elmtg5jh'
TPAGA_PUBLIC_TOKEN = 'd13fr8n7vhvkuch3lq2ds5qhjnd2pdd2'
TPAGA_API_URL = 'https://sandbox.tpaga.co/api/'

class TpagaTestClient:
    def __init__(
            self,
            private_token=TPAGA_PRIVATE_TOKEN,
            public_token=TPAGA_PUBLIC_TOKEN,
            base_url=TPAGA_API_URL,
    ):
        self.base_url = base_url
        self.private_token = private_token
        self.public_token = public_token


    def api_post(self, path, data, token=None):
        if not token:
            token = self.private_token
        return requests.post(
            urlparse.urljoin(self.base_url, path),
            json=data, auth=(token, ''),
        )

    def fail(self, response):
        raise Exception(
            'Whoops, got\n\nSTATUS: {}\n\nHEADERS: {}\n\nCONTENT: {}'.format(
                response.status_code, 
                response.headers, 
                response.content,
            ))

    def json_from_response(self, response, expected_http_code=None):
        if not expected_http_code:
            expected_http_code = [201]
        if response.status_code not in expected_http_code:
            self.fail(response)
        if not response.content:
            return None
        return response.json()

```

El método `__init__` nos va a inicializar nuestor cliente, `api_post` - mandar peticiones POST a la ruta especificada (`path`) del API, `json_from_response` - obtener un objeto JSON de la respuesta de API, `fail` - imprimir los detalles de la respuesta si la petición no ha terminado con éxito.

### Crear un cliente

Para crear nuestor cliente, vamos a entiar una petición POST al endpoint `/customer`:

```python
class TpagaTestClient:
    # ...

    def create_customer(self, data):
        response = self.api_post('customer', data)
        return self.json_from_response(response)
```

```python
>> client = TpagaTestClient()
>> customer = client.create_customer({
    'firstName': 'Horns and Hoofs',
    'email': 'hornsandhoofs@example.com',
    'phone': '012345678'
})
>> customer_token = customer['id']
>> print('customer_token', customer_token)
customer_token qoodmh04sh7ghpp58opn5g0hssg4slq0
```

Otros campos que podemos enviar para guardar nuestros clientes se puede encontrar aquí: [tpaga.co/docs/swaggers/v2#!/Customer/createCustomer](https://tpaga.co/docs/swaggers/v2#!/Customer/createCustomer).

En el dashboard de Tpaga podemos asegurarnos de que el ciente ["Horns and Hoofs"](https://en.wikipedia.org/wiki/The_Little_Golden_Calf#Cultural_influence) fue creado exitosamente:

{% img /images/tpaga_dashboard_customers.png %}

Teniendo un token de nuestro cliente, podemos agregarle una tarjeta de crédito.

### Registrar una tarhjeta de crédito y asociarla al cliente

Creación de la tarjeta de crédito se realiza en dos pasos: tokenizar la tarjeta y asociarla un cliente.

```python
class TpagaTestClient:
    # ...

    def tokenize_cc(self, cc_name, expiry_month, expiry_year, cc_num):
        ccdata = {
            'primaryAccountNumber': cc_num,
            'expirationMonth': expiry_month,
            'expirationYear': expiry_year,
            'cardHolderName': cc_name,
        }
        response = self.api_post('tokenize/credit_card', ccdata, token=self.public_token)
        return self.json_from_response(response)


    def assoc_cc_to_customer(self, customer_token, cc_temp_token=None):
        cdata = {'token': cc_temp_token }
        response = self.api_post(
            'customer/{}/credit_card_token'.format(customer_token), cdata
        )
        return self.json_from_response(response)
```

```python
>> tokenized_credit_card = client.tokenize_cc(
    cc_name='Pepito Perez', 
    expiry_month="08", 
    expiry_year="2020", 
    cc_num='4111111111111111',
)
>> cc_temp_token = tokenized_credit_card['token']
>> credit_card = client.assoc_cc_to_customer(
    customer_token=customer_token, 
    cc_temp_token=cc_temp_token,
)
>> credit_card_token = credit_card['id']
print('credit_card_token', credit_card_token)
credit_card_token 2k54foql0hki0ot7avrg9nhpvbpqam55
```

Mirando la tabla de [sandbox.tpaga.co/merchantDashboard/cards](https://sandbox.tpaga.co/merchantDashboard/cards) vemos que nuestra tarjeta quedó registrada con Tpaga.

### Realizar el pago por la tarjeta de crédito

Con el token de la tarjeta de crédito, que obtuvimos en el paso anterior, ahora podemos realizar pagos. Para eso enviemos una petición POST al [addCreditCardCharge](https://tpaga.co/docs/swaggers/v2#!/Charge/addCreditCardCharge) endpoint con los siguientes paramentros:

* `orderId` - nuestro id interno que asociamos al pago, que luego nos ayudaría identificar la transacción en el dashboard de Tpaga;
* `amount`- cantidad de dinero para cobrar,
* `currency`- tipo de moneda, por ejemplo, 'COP',
* `creditCard` - token de la tarjeta de crédito.

```python
class TpagaTestClient:
    # ...

    def charge_cc(self, cc_token, order_id='BRG-2', amount=1000):
        cdata = {
              'orderId': order_id,
              'currency': 'COP',
              'taxAmount': 0,
              'description': 'One bridge in good condition.',
              'installments': 1,
              'amount': amount,
              'creditCard': cc_token,
        }
        response = self.api_post('charge/credit_card', cdata)
        return self.json_from_response(response)

    def refund_cc(self, cc_charge_id):
        cdata = {
            'id': cc_charge_id,
        }
        response = self.api_post('refund/credit_card', cdata)
        return self.json_from_response(response, expected_http_code=[202])
```

```python
>> charge_cc_response = client.charge_cc(cc_token=credit_card_token, amount=4500)
>> cc_charge_id = charge_cc_response['id']
print('charge_cc_response', charge_cc_response)
charge_cc_response {
    'id': '1rnfu463258eph0mlqli4105mjb85kut', 
    'creditCard': 'ifmjd9rbe8peqdjh09pln702306nfniu', 
    'thirdPartyId': None, 
    'installments': 1, 
    'tpagaFeeAmount': '868.00', 
    'customer': 'gl01l74skk0po9afrjiaaclt0hr5acsh', 
    'iacAmount': '0.00', 
    'transactionInfo': {'authorizationCode': '723045', 'status': 'authorized'},
    'netAmount': '3545.00', 
    'tipAmount': '0.00', 
    'reteIvaAmount': '0.00', 
    'reteIcaAmount': '19.00', 
    'paid': True, 
    'reteRentaAmount': '68.00', 
    'paymentTransaction': 'tta2hlk0e5n5dgr4kggm5j6vv8qoh3jP', 
    'orderId': 'BRG-2', 
    'description': 'One bridge in good condition.', 
    'currency': 'COP', 
    'errorMessage': 'Approved', 
    'taxAmount': '0.00', 
    'errorCode': '00', 
    'amount': '4500.00'
}
```

Si el pago fue exitoso, el código de respuesta es `201` y en el JSON podemos ver que la llave `paid` es `True` y `amount` es igual al valor cobrado de la tarjeta.

En el caso cuando el código de respuesta es `402`, tendríamos fijarnos en los valores de `errorCode` y `errorMessage` para entender qué pasó con la transacción. Por ejemplo, el código de error `43` significa que el dueño de la tarjeta la reportó como robada, y `61` - que el monto máximo de tarjeta fue excedido.

En otros casos necesitaremos verificar que los datos que pasamos en la petición sean válidos y tengan todos los valores necesarios. 

### Revertir el pago

Los bancos nos permiten revertir el pago dentro de 24 horas después de la transacción. Para hacerlo debenos mandar token de la transacción que queremos revertir al [refundCreditCardCharge](https://tpaga.co/docs/swaggers/v2#!/Credit_Card/refundCreditCardCharge) endpoint.

```python
>> refund_cc_response = client.refund_cc(cc_charge_id)
>> print('refund_cc_response', refund_cc_response)
refund_cc_response {
    'id': '1rnfu463258eph0mlqli4105mjb85kut', 
    'creditCard': 'ifmjd9rbe8peqdjh09pln702306nfniu', 
    'thirdPartyId': None, 
    'installments': 1, 
    'tpagaFeeAmount': '868.00', 
    'customer': 'gl01l74skk0po9afrjiaaclt0hr5acsh', 
    'iacAmount': '0.00', 
    'transactionInfo': {'authorizationCode': '723045', 'status': 'voided'}, 
    'netAmount': '3545.00', 
    'tipAmount': '0.00', 
    'reteIvaAmount': '0.00', 
    'reteIcaAmount': '19.00', 
    'paid': False, 
    'reteRentaAmount': '68.00', 
    'paymentTransaction': 'tta2hlk0e5n5dgr4kggm5j6vv8qoh3jP', 
    'orderId': 'BRG-2', 
    'description': 'One bridge in good condition.', 
    'currency': 'COP', 
    'errorMessage': 'Approved', 
    'taxAmount': '0.00', 
    'errorCode': '00', 
    'amount': '4500.00'
}
```

El JSON que nos devolvió Tpaga `transactionInfo.status` aparece como `voided` y el valor de `paid` ahora es falso:

{% img /images/tpaga_dashboard_refund.png %}

Enlaces:

* Documentación de Tpaga: [tpaga.co/docs/swaggers/v2](https://tpaga.co/docs/swaggers/v2)
* Tpaga FAQ [tpaga.zendesk.com/hc/es](https://tpaga.zendesk.com/hc/es)
* Documentación para la librería [Requests: HTTP for Humans](http://docs.python-requests.org/en/master/)

