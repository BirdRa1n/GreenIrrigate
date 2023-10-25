
# Documentação do ESP

O controller `getStatus` é responsável por gerenciar solicitações para obter o status de um dispositivo ESP (Internet das Coisas). Ele realiza verificações, atualizações no banco de dados e determina o estado da bomba de água com base em eventos de irrigação e umidade.

## Endpoint

```
GET /api/status
```

## Parâmetros da Solicitação

- **UUID** (obrigatório): Um identificador exclusivo do dispositivo ESP.
- **HUMIDITY**: O nível de umidade, se disponível.

## Respostas

### Sucesso

- **Código**: 200 OK
- **Exemplo de Resposta**:

```json
{
  "msg": "ligado",
  "waterPumpStatus": true
}
```

- **Descrição**: A bomba de água está ligada, pois o dispositivo ESP está dentro do período de um evento de irrigação.

### Falha na Autenticação do ESP

- **Código**: 401 Unauthorized
- **Exemplo de Resposta**:

```json
{
  "msg": "Falha na autenticação do ESP"
}
```

- **Descrição**: O UUID fornecido não corresponde a nenhum dispositivo ESP registrado.

### Parâmetros Ausentes ou Inválidos

- **Código**: 400 Bad Request
- **Exemplo de Resposta**:

```json
{
  "msg": "Definir o UUID é obrigatório"
}
```

- **Descrição**: O parâmetro UUID é obrigatório e está ausente na solicitação.

## Lógica do Controller

1. O controller verifica a presença do UUID na solicitação. Se ausente, responde com um erro 400.

2. Obtém o horário atual e ajusta-o para o fuso horário de Brasília (GMT-3).

3. Consulta o banco de dados em busca do dispositivo ESP com o UUID fornecido.

4. Se o dispositivo ESP não for encontrado, responde com um erro 401 indicando falha na autenticação do ESP.

5. Atualiza o campo `lastAcess` do dispositivo ESP no banco de dados para refletir o momento atual.

6. Verifica a presença de `HUMIDITY` na solicitação e, se presente, verifica o último registro de umidade. Se não houver registro ou se o intervalo de tempo for maior que 1 minuto, cria um novo registro de umidade.

7. Consulta os eventos de irrigação associados ao dispositivo ESP.

8. Itera sobre os eventos de irrigação e verifica se o horário atual está dentro do período de qualquer evento.

9. Se estiver dentro de um evento, atualiza o status da bomba de água para `true` e retorna uma resposta indicando que a bomba está ligada.

10. Se não estiver dentro de um evento, atualiza o status da bomba de água para `false` e retorna uma resposta indicando que a bomba está desligada.

## Exemplo de Uso

```shell
curl -X GET \
  -H "Content-Type: application/json" \
  -d '{"UUID": "seu_uuid_aqui", "HUMIDITY": 45}' \
  http://localhost:3000/api/status
```

## Notas

- Certifique-se de substituir `http://localhost:3000` pela URL real da sua API.
- Este controller assume que os modelos de dados, como `ESP`, `HumidityLog` e `IrrigationEvent`, estão definidos corretamente na sua aplicação.

