# 🧩 Nós do N8N Utilizados

| Nó (Node) | Função no Fluxo | Configuração Principal |
|---|---|---|
| **Google Sheets Trigger** | Inicia o workflow quando uma nova resposta é adicionada à planilha vinculada ao Google Forms. | Selecionar a planilha e a aba de respostas do formulário. Poll Times: a cada minuto (ou conforme necessidade). |
| **Set (Edit Fields)** *(opcional)* | Padroniza e renomeia os campos recebidos, garantindo consistência (ex.: e-mail em minúsculas). | Mapear os campos do formulário para nomes padronizados (ex.: `nome`, `email`, `telefone`). |
| **IF** *(ou Filter)* | Valida se o e-mail está preenchido e se o formato é válido. Se inválido, o fluxo é interrompido. | Condição: `{{ $json["email"] }}` não está vazio **E** corresponde à expressão regular de e-mail. |
| **Google Sheets (Append Row)** | Salva os dados do cliente na planilha de CRM/controle. | Selecionar a planilha de destino, aba e mapear cada coluna com os campos recebidos. |
| **Gmail (Send Email) – Cliente** | Envia e-mail de confirmação para o cliente. | Destinatário: `{{ $json["email"] }}`; Assunto e corpo personalizados. |
| **Gmail (Send Email) – Equipe** | Envia notificação para o time comercial sobre o novo cadastro. | Destinatário: e-mail fixo da equipe; Assunto e corpo com os dados do cliente. |

---

## 🔄 Lógica de Funcionamento do Workflow

### 1. Gatilho (Trigger)

O nó **Google Sheets Trigger** monitora a planilha de respostas do Google Forms. Assim que uma nova linha é adicionada, o workflow é iniciado com todos os dados da submissão.

### 2. Padronização (Opcional)

O nó **Set** pode ser usado para renomear campos, converter e-mail para minúsculas, remover espaços extras e garantir que os dados estejam no formato correto antes da validação.

### 3. Validação do E-mail (Regra de Negócio)

O nó **IF** verifica se o campo de e-mail não está vazio e se corresponde a um formato válido.

- **Se verdadeiro:** o fluxo continua.
- **Se falso:** o fluxo é encerrado para aquele item (pode-se usar um nó **NoOp** para indicar que nada será feito).

Expressão regular sugerida:

```regex
^[^\s@]+@[^\s@]+\.[^\s@]+$
```

### 4. Registro do Cliente

Se o e-mail for válido, o nó **Google Sheets (Append Row)** adiciona uma nova linha na planilha de CRM, salvando todos os dados do cliente (nome, e-mail, telefone, empresa, data/hora, etc.).

### 5. Envio de E-mails

A partir do nó **Google Sheets** (ou em paralelo), conectam-se dois nós **Gmail**:

- **Gmail – Cliente:** envia um e-mail de confirmação para o endereço validado, com assunto do tipo “Cadastro recebido com sucesso!” e corpo personalizado usando os dados do cliente.
- **Gmail – Equipe:** envia uma notificação para o e-mail fixo da equipe comercial, com assunto “Novo cliente cadastrado: `{{ $json["nome"] }}`” e todos os detalhes no corpo.

> **Dica:** No N8N, você pode conectar a saída do nó **Google Sheets** diretamente aos dois nós **Gmail**. Ambos receberão os mesmos dados de entrada.

---

## ⚙️ Configurações Importantes

- **Credenciais:** Configure OAuth2 para Google Sheets e Gmail nas credenciais do N8N.
- **Mapeamento de Colunas:** No nó **Google Sheets (Append Row)**, use “Map Each Column Manually” para ter controle total sobre qual campo do formulário vai para qual coluna da planilha de destino.
- **Personalização dos E-mails:**
  - **Para o cliente:**
    - Destinatário: `{{ $json["email"] }}`
    - Assunto: `Confirmação de cadastro – {{ $json["nome"] }}`
    - Corpo: use HTML ou texto simples com `{{ $json["nome"] }}`, `{{ $json["email"] }}`, etc.
  - **Para a equipe:**
    - Destinatário: e-mail fixo (ex.: `comercial@empresa.com`)
    - Assunto: `Novo lead: {{ $json["nome"] }}`
    - Corpo: detalhes do cliente e link para a planilha, se desejar.
- **Tratamento de Erros:** Opcionalmente, adicione um nó **Error Trigger** ou configure “Continue On Fail” nos nós **Gmail** para evitar que falhas interrompam o fluxo.
