# Description of CAPI RESTful API.

![Capi Logo](http://10.4.0.59/capi/Documentation/Assets/img/capi.png "Capi Logo")

## O que é a CAPI?
A Central API (CAPI) é uma API da Universidade Aberta que permiter acedar a vários serviços directamente através de pedidos REST.

Neste momento estão disponiveis as paths para registo e login de clientes e após autenticação CRUD para Active Directory.

Esta API só permite aceder aos serviços após o cliente estar registado e autenticado na mesma. 

### Index
- [Autenticação](#Autenticação)

### Autenticação
A CAPI utiliza autenticação via JSON Web Token (JWT).

JWT é um padrão aberto documentado pelo [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519 "RFC 7519"), que viabiliza a transferência de informações e garante a sua autenticidade.

Um JWT é divido em três partes separadas por ponto ”.”, um header, um payload e uma signature.


  - Header
    - Formato JSON e geralmente, contém o algoritmo de hashing utilizado na assinatura e o tipo de token (JWT)

```json http
{
  "method": "post",
  "url": "http://capi.local:8080/auth/login",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "client_name": "teste1",
    "password": "1234567890"
  }
}
```

>CONTACT
>
>NAME: CAPI
>URL: https://capi.uab.pt/developer/support
>Terms of service: https://capi.uab.pt/about/legal/terms/api/