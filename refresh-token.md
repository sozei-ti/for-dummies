# Introdução

Access tokens com curto tempo de duração contribuem para uma maior segurança de sua aplicação, porém quando expirados o usuário precisa logar novamente para gerar um novo token e frequentemente isso se torna uma experiência frustante.

O Refresh token vem para auxiliar no balanceamento entre a segurança de sua aplicação e a usabilidade do usuário, já que refresh tokens geralmente tem um período maior de duração.

No entanto, precisamos ter uma estratégia que limite ou reduza o poder do refresh token caso ele seja vazado, já que todos que possuem um refresh token podem solicitar um novo access token a qualquer momento.

# Medidas de Segurança

* O Refresh token deve ser um hash que não contenha nenhuma informação referente ao usuário ou ao access_token, e sim referenciar essa informação que está armazenada no banco de dados.

* Garantir que o usuário que está solicitando um novo access_token seja o mesmo usuário que gerou o refresh token usado para solicitação

* Não permitir que o usuário gere um novo access_token antes de ele ter sido expirado.


## Implementação

### Criando tabela para armazenar refresh tokens

```
user_tokens
  user_id  
  refresh_token
  device_id
  refresh_token_expiration_date
  access_token_expiration_date
```

### Gerando refresh token

Para garantirmos que o dispositivo que gerou o refresh token, seja o mesmo que irá utilizar ele, precisamos armazenar uma referência ao dispositivo do usuário, por isso utilizamos o device_id.

Ao criar a sessão do usuário, o front-end irá enviar o `device_id` que é uma informação única de seu dispositivo, como o `mac_address` em aplicações mobile por exemplo.

Um exemplo do request enviado pelo front_end na criação de uma sessão:
```json
{
	"email": "customer@customer.com",
	"password": "123123123",
	"device_id":"a848f104-25f3-4b43-a3d0-03829769990c"
}
```

Agora na API, na criação do access_token iremos criar também o refresh token com o device_id da sessão.

Iremos armazenar também a data de expiração de nosso `access_token` e `refresh_token`. 

No exemplo abaixo temos um access token com 1 dia de expiração e um refresh_token com 30 dias.

```ts
    const refresh_token = uuid();

    const refresh_token_expiration_date = this.dateProvider.addDays(30);

    const access_token_expiration_date = this.dateProvider.addDays(1);

    await this.userTokensRepository.create({
      user_id: user.id,
      device_id,
      refresh_token,
      refresh_token_expiration_date,
      access_token_expiration_date,
    });
```

Então retornaremos para o usuário seu `access_token` e `refresh_token`, assim o front-end armazenara esse refresh token em um storage para ele ser utilizado na atualização da sessão do usuário.

## Atualizando Sessão

Exemplo do request enviado pelo front-end para o refresh da session:
```json
{
  "refresh_token": "d59a897f-daff-4ccf-a5f9-4e61324225df",
  "device_id":"a848f104-25f3-4b43-a3d0-03829769990c"
}
```

Agora antes de gerarmos um novo access_token para esse usuário precisamos fazer algumas verificações.

* Verificar se esse refresh_token existe no banco de dados
* Verificar se o usuário desse refresh_token existe
* Verificar se o refresh_token foi expirado
* Verificar se o access_token já foi expirado
* Verificar se o device_id fornecido é o mesmo utilizado para gerar o refresh token

Um exemplo dessas verificações no RefreshTokenService:
```ts
// Checking token
    const userToken = await this.userTokensRepository.findByToken(
      refresh_token,
    );

    if (!userToken) {
      throw new AppError('Refresh token não encontrado', 404);
    }

    // Checking user
    const user = await this.usersRepository.findById(userToken.user_id);

    if (!user) {
      throw new AppError('Usuário não encontrado', 404);
    }

    // checking expiration dates
    const refreshTokenExpired = this.dateProvider.isAfter({
      date: new Date(),
      compare_date: userToken.refresh_token_expiration_date,
    });

    if (refreshTokenExpired) {
      throw new AppError('Refresh token inválido', 401);
    }

    const accessTokenExpired = this.dateProvider.isAfter({
      date: new Date(),
      compare_date: userToken.access_token_expiration_date,
    });

    if (!accessTokenExpired) {
      throw new AppError('Refresh token inválido', 401);
    }

    // Checking device_id
    const allowedDevice = Boolean(device_id === userToken.device_id);

    if (!allowedDevice) {
      throw new AppError('Refresh token inválido', 401);
    }
```

Então se o refresh_token fornecido passou por todas essas verificações, podemos apagar ele do banco de dados e gerar um novo access_token e refresh_token para o usuário.


# Conclusão 

Com as medidas tomadas, por mais que o nosso refresh fique exposto a invasores no front-end, nossa API está garantindo que um novo access_token só vai ser gerado, pelo mesmo dispositivo que ele foi criado.