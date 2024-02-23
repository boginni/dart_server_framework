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

Routes.get('/me', (Request request) {
    return await authorizationEnforcerMiddleware(request, me);
});
```

### Middlewares com prefixo
um Middleware deve conseguir agrupar um prefixo:

```dart
final UserController userController;

Routes.middleware(authorizationEnforcerMiddleware).prefix('/user', (Router router) {
    router.get('/', userController.all);
    router.get('/{id}', userController.index);
    router.post('/', userController.store);
    router.put('/{id}', userController.update);
    router.delete('/{id}', userController.delete);
});
```

### Validação de Request
Um Request deve conseguir validar e injetar no request Automaticamente


Declaração do request:
```dart
class CreateUserBody extends RequestBody {

    @Validation([Validator.min(4), Validador.mandatory])
    final String nome;
    @Validation([Validator.email, Validador.mandatory])
    final String login;
    @Validation([Validator.password, Validador.mandatory])
    final String senha;
    @Validation([Validador.optional])
    final String cidade;
    @Validation([Validador.optional])
    final int idade;

    const CreateUserBody({
        required this.nome,
        required this.login,
        required this.senha,
        required this.cidade,
        required this.idade,
    });
}
```

na função de request:
```dart

class UserController {


    Future<Response> store<CreateUserBody>(Request request){
        userRepository.create(request);
    }

}

``` 
### Injeção de Parametros como Entity
Um Request deve conseguir extrair uma entity e injetar na função


no caso de um request com ID
`DELETE: /user/{id}`


Declaração do request:
```dart
class UserEntity {
    final int id;
    final String name;
    final String login;
    final String senha;
    final String cidade;
    final int idade;
}
```

na função de request:
```dart
class UserController {

    Future<Response> delete<CreateUserBody>(Request request, UserEntity entity){
        userRepository.delete(entity.id);
    }

}
``` 

#### Problema Multiplos parâmetros na rota:
A o roteamente deverá conseguir encontrar Todas an entidades do request
`DELETE: /company/{id}/car/{carModelId}`

```dart
class CompanyCarTypeController {

    Future<Response> delete(Request request, CompanyEntity company, CarTypeEntity carType){
        company.deleteCarType(carType.id);
    }

}
```

## Database


### Query Builder
A api não pode dependenter do tipo de banco de dados, é necessário evitar a utilização de sql:

#### QueryBuilder Método 1
```dart
class ProfileRepository {

    final QueryBuilder queryBuilder;

    UserRepository({
        required this.queryBuilder,
    });

    Future<Pagination<UserEntity>> getFollowers(userId) async {
       return await queryBuilder.select('user_followers').where('user_id', comparision: '=', userId).paginated();
    }

}
```

#### QueryBuilder Método 2
aidna há necessidade de definir melhor como funciona o query builder, mas o ideal é que não haja necessidade de passar nome de campos por String `user_id`

```dart

class ProfileRepository {

    UserRepository({
        required this.queryBuilder,
    });

    Future<Pagination<UserEntity>> getFollowers(userId) async {
        /// queryBuilder<UserFollowerEntity>
       final queryBuilder = UserFollowerEntity.select();

       return queryBuilder.where('user_id', comparision: '=', userId).paginated()..map((e) => e.follower);
    }

}
```

### Tratamento de Entitades 

Classes devem permitir saber que tipo de dados vão poder retornar;

```dart

class UserEntity {
    final int id;
    @Exposed()
    final String name;
    final String login;
    final String senha;
    @Exposed()
    final String cidade;
    @Exposed()
    final int idade;
}
```

