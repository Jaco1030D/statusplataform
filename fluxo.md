# Fluxo usuário atualizado:

## -> usuário insere o arquivo e ele é enviado

**Vai:**
Ocorre um Post na */api/files/upload-default com um formdata file

**retorna:**
- file
- project
- workspace

---

## -> Arquivo é processado

**vai:**
conexão websocket iniciando com mensagem upload-file, nela vai um {"_id": "idDoArquivo" }

**Mensagens que o websocket pode retornar:**

a mensagem de progresso(upload-progress) retorna:
```json
{
    "fileId": "691c7de5b8d670f6f1e86a07",
    "message": "Upload concluido",
    "percent": 100
}
```

a mensagem de erro (upload-error) retorna: 
```json
{
     message: `Erro durante o processamento: ${error.message}`,
     error: error.message
}
```

a mensagem final(file-uploaded) retorna:
- fileId
- segments
- message
- percent
- pagination
- fileUrl -> use isso para mostrar o pdf na tela para o usuário

---

## -> Traduzir o documento

**vai:**
Conexão websocket iniciando com mensagem translate-file, nela vai:
```json
{
  "id": "idDoArquivo",
  "fileId": "idDoArquivo",  // alternativa ao "id"
  "languages": {            // opcional, padrão: { source: 'pt-br', target: 'EN-USA' }
    "source": "pt-br",
    "target": "EN-USA"
  }
}
```

**Mensagens que o websocket pode retornar:**

A mensagem de progresso (translation-progress) retorna:
```json
{
  "fileId": "691c7de5b8d670f6f1e86a07",
  "message": "Buscando segmentos para tradução...",
  "percent": 5
}
```

A mensagem de batch (translation-batch) retorna:
```json
{
  "fileId": "691c7de5b8d670f6f1e86a07",
  "segments": [
    {
      "id": "idDoSegmento",
      "target_segment": "Texto traduzido"
    }
  ]
}
```

A mensagem de erro (translation-error) retorna:
```json
{
  "id": "691c7de5b8d670f6f1e86a07",
  "message": "Erro ao conectar ao serviço de tradução."
}
```

A mensagem final (translation-completed) retorna:
- fileId
- result (objeto com status e dados da tradução)

O translation-batch é emitido durante o processamento conforme os segmentos são traduzidos
O translation-progress mostra o progresso geral (5%, 15%, 30%, 60%, 100%)
O translation-completed só é emitido quando todos os segmentos foram processados com sucesso

---

## -> usuário se loga

**vai:**
Requisição HTTP POST para /api/users/register com body:
```json
{  "name": "Nome do Usuário",  "email": "usuario@email.com",  "password": "senha123",  "country": "Brasil",  "role": "user", workspaceId: "o que foi retornado na primeira etapa", projectId: "o que foi retornado na primeira etapa"}
```

**Resposta de sucesso (201 Created) retorna:**
```json
{  "_id": "691c7de5b8d670f6f1e86a07",  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}
```

**Resposta de erro retorna:**

Email já cadastrado (422 Unprocessable Entity):
```json
{  "errors": ["Por favor, utilize outro e-mail."]}
```

Erro interno (500 Internal Server Error):
```json
{  "errors": ["Erro interno. Tente novamente mais tarde."]}
```

Falha na criação (422 Unprocessable Entity):
```json
{  "errors": ["Houve um erro, por favor tente novamente mais tarde."]}
```

---

## -> usuário escolhe plano e vai para o stripe

há um webhook que captura quando um pagamento é feito com sucesso, ele salva um usuário como premium

---

## -> Stripe redireciona para o dashboard principal

Obtem todos os workspace e pega o id do primeiro(até esse momento há apenas 1) GET /api/workspaces/all
Obtem todos os Projects e pega o id do primeiro(até esse momento há apenas 1) GET /api/projects?id=68c88dcd22ffac28704f4fa8
Obtem todos os files usando o id do projeto GET /api/files

