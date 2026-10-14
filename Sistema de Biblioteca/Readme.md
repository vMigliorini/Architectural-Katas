# Kata de Arquitetura — Sistema de Empréstimo de Livros

## Contexto

A **iB2B** deseja informatizar o processo de empréstimo de livros de sua biblioteca.

O sistema deve permitir o cadastro e a manutenção das informações necessárias ao funcionamento da biblioteca, além de suportar reserva, consulta de reservas, empréstimo, renovação, devolução, inspeção de livros, reparo, geração de multas e quitação de boletos.

Reserva, empréstimo e devolução são transações. Toda transação ocorre em uma data e envolve um livro, um estudante e um atendente.

## Domínio

### Pessoa

Toda pessoa possui:

- nome;
- sobrenome;
- CPF.

### Estudante

O estudante é um papel assumido por uma pessoa e possui um identificador próprio.

Um estudante só pode ser cadastrado se a pessoa à qual ele está associado já estiver cadastrada.

### Atendente

O atendente é um papel assumido por uma pessoa e possui um identificador próprio.

Um atendente só pode ser cadastrado se a pessoa à qual ele está associado já estiver cadastrada.

### Livro

Todo livro possui:

- identificador;
- título;
- estado.

Os livros cadastrados pertencem ao acervo da biblioteca.

### Transações

As movimentações de reserva, empréstimo e devolução possuem um único ponto de acesso interno, denominado **Movimentação**, sendo mantidas por associações diferentes.

As informações de Pessoa, Estudante e Atendente possuem um único ponto de acesso interno, denominado **Relacionamento**, também mantidas por associações diferentes.

## Atores

O sistema possui os seguintes atores:

- **Administrador**;
- **Estudante**;
- **Atendente**;
- **Banco**, como participante externo do processo de pagamento.

### Administrador

O Administrador é responsável pelas funcionalidades de manutenção de:

- Livro;
- Pessoa;
- Estudante;
- Atendente.

Cada uma dessas funcionalidades permite:

- inserir;
- consultar;
- atualizar;
- excluir.

### Estudante

O Estudante pode:

- solicitar a reserva de um livro;
- consultar suas reservas;
- efetivar o empréstimo de um livro reservado;
- renovar um empréstimo.

### Atendente

O Atendente participa da entrega e da devolução dos livros e pode:

- registrar a devolução;
- inspecionar o livro devolvido;
- encaminhar um livro danificado para reparo;
- registrar o retorno de um livro reparado;
- solicitar a emissão de multa quando houver atraso.

## Organização interna

As informações do sistema são distribuídas entre quatro áreas:

- **Acadêmico**: estudantes;
- **RH**: pessoas e atendentes;
- **Biblioteca**: livros, reservas, empréstimos, renovações e devoluções;
- **Financeiro**: multas, boletos e quitações.

O **Banco** é um sistema externo utilizado para cobrança e confirmação de pagamentos.

## Cadastro e manutenção

### Livro

Ao inserir um livro:

- se ele já estiver cadastrado, o cadastro existente é apresentado;
- caso contrário, ele é criado e inserido no acervo.

Um livro cadastrado inicialmente fica **LIVRE**.

Um livro também pode ser consultado, atualizado ou excluído.

### Pessoa

Ao inserir uma pessoa:

- se ela já estiver cadastrada, seus dados são apresentados;
- caso contrário, ela é registrada em `Relacionamento`.

Uma pessoa também pode ser consultada, atualizada ou excluída.

### Estudante

Para cadastrar um estudante:

- a pessoa correspondente deve estar cadastrada;
- o estudante ainda não pode estar cadastrado.

O estudante é registrado por meio de `Relacionamento`.

### Atendente

Para cadastrar um atendente:

- a pessoa correspondente deve estar cadastrada;
- o atendente ainda não pode estar cadastrado.

O atendente é registrado por meio de `Relacionamento`.

## Reserva

O Estudante solicita à Biblioteca a reserva de um livro.

A reserva só pode ser realizada quando:

- o livro está cadastrado;
- o estudante está cadastrado;
- o livro está **LIVRE**.

Quando a reserva é aceita:

1. uma `Reserva` é criada para o livro e o estudante;
2. a reserva é adicionada à `Movimentação`;
3. o livro passa para **RESERVADO**.

Caso as condições não sejam atendidas, a reserva é recusada.

## Consulta de reserva

Uma reserva pode ser consultada na `Movimentação` utilizando o livro e o estudante envolvidos.

Quando a reserva existe, seus dados são apresentados.

## Empréstimo

O empréstimo só pode ser efetivado quando existe uma reserva registrada para o mesmo livro e estudante.

Quando o empréstimo é realizado:

1. a `Reserva` é removida da `Movimentação`;
2. um `Empréstimo` é inserido na `Movimentação`;
3. o livro passa para **EMPRESTADO**;
4. o Atendente entrega o livro ao Estudante.

## Renovação

O Estudante pode solicitar a renovação de um empréstimo.

A renovação é permitida quando:

- existe um empréstimo registrado para o livro e o estudante;
- o livro está **EMPRESTADO** ou **RENOVADO**;
- o número de renovações é menor que 2.

Na primeira renovação:

- o livro passa de **EMPRESTADO** para **RENOVADO**;
- a data de vencimento é estendida em 30 dias;
- o número de renovações é incrementado.

Uma segunda renovação pode ser realizada enquanto o limite não tiver sido atingido, mantendo o estado **RENOVADO** e acrescentando mais 30 dias ao vencimento.

Um empréstimo pode ser renovado no máximo **duas vezes**.

## Devolução e inspeção

O Estudante devolve o livro ao Atendente.

O Atendente registra a devolução na Biblioteca:

1. uma `Devolução` é inserida na `Movimentação`;
2. o `Empréstimo` correspondente é removido;
3. o livro passa para **DEVOLVIDO**.

Depois da devolução, o livro é inspecionado.

### Livro em bom estado

Se o livro estiver em bom estado:

- ele volta para **LIVRE**.

### Livro danificado

Se o livro estiver danificado:

- ele passa para **REPARANDO**;
- é encaminhado para reparo;
- após retornar do reparo, volta para **LIVRE**.

Um livro que esteja **LIVRE** também pode ser enviado para reparo caso seja identificado algum dano no acervo.

## Multa por atraso

Se a devolução ocorrer depois da data de vencimento, a Biblioteca solicita ao Financeiro a emissão de uma multa.

A multa é calculada no valor de:

**R$ 1,00 por dia de atraso.**

O Financeiro:

1. identifica o estudante pelo CPF;
2. registra a multa;
3. emite o boleto bancário;
4. envia o boleto ao Estudante por e-mail.

## Pagamento

O Estudante paga o boleto no Banco.

O Banco informa a quitação ao Financeiro, que registra a multa como quitada.

## Ciclo de vida do livro

No fluxo descrito, o livro assume os seguintes estados:

- **LIVRE**;
- **RESERVADO**;
- **EMPRESTADO**;
- **RENOVADO**;
- **DEVOLVIDO**;
- **REPARANDO**.

As principais mudanças de estado são:

| Origem | Evento | Condição | Destino |
|---|---|---|---|
| Estado inicial | cadastrar livro | livro criado | **LIVRE** |
| **LIVRE** | reservar | estudante cadastrado | **RESERVADO** |
| **RESERVADO** | emprestar | reserva do mesmo estudante | **EMPRESTADO** |
| **EMPRESTADO** | renovar | primeira renovação | **RENOVADO** |
| **RENOVADO** | renovar | limite de duas renovações não atingido | **RENOVADO** |
| **EMPRESTADO** ou **RENOVADO** | devolver | empréstimo registrado | **DEVOLVIDO** |
| **DEVOLVIDO** | inspecionar | livro em bom estado | **LIVRE** |
| **DEVOLVIDO** | inspecionar | livro danificado | **REPARANDO** |
| **LIVRE** | enviar para reparo | dano identificado no acervo | **REPARANDO** |
| **REPARANDO** | retornar do reparo | reparo concluído | **LIVRE** |
| **LIVRE** | excluir livro | exclusão do cadastro | estado final |

Quando uma devolução ocorre com atraso, a geração da multa faz parte do processamento da devolução.

## Estrutura do sistema

O sistema é organizado em torno dos componentes correspondentes às áreas **RH**, **Acadêmico**, **Biblioteca** e **Financeiro**, além do **Banco** como componente externo e de uma **Interface Web** utilizada pelos usuários.

A comunicação entre os componentes ocorre por meio de interfaces fornecidas e requeridas.

Como referência para as responsabilidades entre componentes:

| Componente | Interfaces fornecidas | Interfaces requeridas |
|---|---|---|
| RH | `iPessoa`, `iAtendente` | — |
| Acadêmico | `iEstudante` | `iPessoa` |
| Biblioteca | `iLivro`, `iEmprestimo`, `iDevolucao` | `iEstudante`, `iAtendente`, `iMulta` |
| Financeiro | `iMulta`, `iQuitacao` | `iPessoa`, `iCobranca` |
| Banco | `iCobranca` | `iQuitacao` |
| Interface Web | — | `iLivro`, `iPessoa`, `iEstudante`, `iAtendente`, `iEmprestimo`, `iDevolucao` |

As operações de cadastro são expostas pelas interfaces:

- `iLivro`;
- `iPessoa`;
- `iEstudante`;
- `iAtendente`.

Elas são implementadas, respectivamente, por:

- `TratadorDeAcervo`;
- `TratadorDePessoa`;
- `TratadorDeEstudante`;
- `TratadorDeAtendente`.

Os serviços de reserva, consulta de reserva, empréstimo e renovação são tratados por `TratadorDeEmprestimo`.

Os serviços de devolução e emissão de multa são tratados por `TratadorDeDevolucao`.

A comunicação entre os tratadores e `Relacionamento`, `Acervo` e `Movimentação` utiliza delegação.

## Cenário de implantação

A interface com o usuário é executada em um **WebBrowser** no computador do usuário.

O navegador se comunica com o servidor de aplicação por **HTTPS**.

A lógica de negócio é executada em um servidor de aplicação com:

- **Ubuntu**;
- **JVM 1090**;
- **Tomcat 10.0.11**.

O sistema utiliza dois servidores de dados distintos, ambos do modelo **Dell Edge 3600**:

### Banco principal

- **MySQL**;
- executado em **Windows 10**.

### Réplica

- **Oracle**;
- executado em **Ubuntu**;
- funciona como réplica do banco MySQL.

A aplicação acessa o MySQL por **JDBC**, e os dados são replicados entre MySQL e Oracle.
