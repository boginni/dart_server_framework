# O que a gente precisa?


## Gerenciamento de Rotas

### Utiliação de Métodos
Permitir utilização criação de todos os tipos de métodos como por exemplo:
 * POST
 * GET
 * PUT
 * DELETE
 ...

### Passagem de parâmetros na rota:
Permitir que as rotas sejam identificadas com variáveis como id
 * /routename/{id}
A adição de parâmetros não deve interefir com rotas sem parâmetros
 * /routename
 * /routename/{id}

### Operações em CRUD
A criação de rotas de mesmo nome não pode interferir se caso tenham métodos diferentes:
 * GET /routename 
 * GET /routename/{id}
 * POST /routename
 * PUT /routename/{id}
 * DELETE /routename/{id}

### Utilização de prefixos
Deverá ser possível adicionar prefixos para agrupar rotas em um mesmo prefixo:
 * /routes
  * /
  * /{id}
  * /name
Prefixos tambeḿ deverão ser possíveis de ser colocados dentro de outros prefixos:
 * /routes
  * /
  * /{id}
  * /name
   */
   */{id}
Parâmetos também poderão ser usados como prefixos:
 * /routes
  * /
  * /{id}
   * /
   * /{id}
   * /subroute
   * /subprefix
    * /
    * /{id} 
  * /name
   */
   */{id}

## Funcionalidade de Fluxo
Os exemplos abaixo devem ser seguidos como instrução e não definição final;

### Chamada de Função
Cada rota deve ser capaz de chamar uma função, essa função deve receber os dados do request e retornar uma resposta:
```dart
typedef RequestEndPoint = Future<Response> Function(Request request);

Future<Response> helloWorld(Request request){
    return Response('Hello World!');
}

Routes.get('/hello-world', helloWorld);
```

### Middlewares
Uma rota deverá poder ser interceptada por um middleware:

```dart
typedef RequestMiddleware = Future<Response> Function(Request request, RequestEndPoint next);

Future<Response> authorizationEnforcerMiddleware(Request request, RequestEndPoint next) {
    if(await request.checkAuthorization()){
        return await next();
    }

    return Request.error();
}
```
exemplo:

```dart
Future<Response> me(Request request){
    final user = GetUser();

    return Response(user.toJson());
}

Routes.get('/hello-world', (Request request) {
    return await authorizationEnforcerMiddleware(request, me);
});
```


