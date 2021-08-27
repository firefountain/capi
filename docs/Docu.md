# Descrição da CAPI RESTful API.

![Capi Logo](http://10.4.0.59/capi/Documentation/Assets/img/capi.png "Capi Logo")

## O que é a CAPI?
A Central API (CAPI) é uma API da Universidade Aberta que permiter acedar à informação de vários serviços directamente através de pedidos REST.

Neste momento estão disponiveis as paths para registo e login de clientes e após autenticação CRUD para Active Directory.

Esta API só permite aceder aos serviços após o cliente estar registado e autenticado na mesma. 

## Processo de autenticação
A CAPI utiliza autenticação via JSON Web Token (JWT).

JWT é um padrão aberto documentado pelo [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519 "RFC 7519"), que viabiliza a transferência de informações e garante a sua autenticidade.

Um JWT é divido em três partes separadas por ponto ”.”, um **header**, um **payload** e uma **signature**.


  - **Header**
    - Formato JSON e geralmente, contém o algoritmo de hashing utilizado na assinatura e o tipo de token (JWT)
```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```
  - **Payload**
    - O payload de um JWT é também um JSON e que contém as informações relevantes ao assunto, denominadas claims, na estrutura de mapa, ou seja, chave=valor.
```json
{
  "client_name": "teste1",
  "iat": 1629906752,
  "exp": 1629910352
}
```
  - **Signature**
    - Signature é a terceira e última parte do JWT, para que possa ter um token, precisa-se de um Cabeçalho, Corpo, o Algoritmo definido no Cabeçalho, um secret definido pela aplicação.
```json
HMACSHA256
  (
  base64UrlEncode(header) +  "." + base64UrlEncode(payload),
  your-256-bit-secret
  ) 
```
 - Resultado:
 ```json
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJjbGllbnRfbmFtZSI6InRlc3RlMSIsImlhdCI6MTYyOTkwNjc1MiwiZXhwIjoxNjI5OTEwMzUyfQ.Vskrf57VjUpEvlfLK3dMIlkAIXCcNDTIdbD3n_YOlq8
```

![Capi flow](http://10.4.0.59/capi/Documentation/Assets/img/flow.png "Capi flow")


>CONTACT
>
>NAME: CAPI
>
>URL: https://capi.uab.pt/developer/support
>
>Terms of service: https://capi.uab.pt/about/legal/terms/api/