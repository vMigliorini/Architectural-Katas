# Kata de Arquitetura — Sistema de Empréstimo de Livros

## Contexto

A **iB2B** deseja informatizar o processo de empréstimo de livros de sua biblioteca.

O sistema deve apoiar todo o ciclo de uso do acervo, desde o cadastro de pessoas e livros até reserva, empréstimo, renovação, devolução, inspeção, eventual reparo e cobrança de multas por atraso.

O desafio envolve diferentes perfis de usuário e áreas da organização, que precisam colaborar ao longo do processo sem perder a consistência das informações sobre pessoas, estudantes, atendentes, livros e transações.

## Domínio

O domínio possui os seguintes conceitos principais:

- **Pessoa**: possui nome, sobrenome e CPF.
- **Estudante**: é um papel associado a uma pessoa e possui um identificador próprio.
- **Atendente**: é um papel associado a uma pessoa e possui um identificador próprio.
- **Livro**: possui identificador, título e um estado que representa sua situação atual.
- **Reserva**: registra a intenção de um estudante de utilizar um livro disponível.
- **Empréstimo**: registra que um livro foi entregue a um estudante por um período determinado.
- **Devolução**: registra o retorno de um livro emprestado.
- **Multa**: representa uma cobrança gerada quando a devolução ocorre após o vencimento.
- **Boleto**: meio utilizado para cobrança e pagamento de uma multa.

Reserva, empréstimo e devolução são tipos de transação e sempre estão relacionados a um livro, um estudante, um atendente e uma data.

## Atores

O sistema é utilizado por quatro atores principais:

- **Administrador**: mantém os cadastros de pessoas, estudantes, atendentes e livros.
- **Estudante**: consulta e utiliza o acervo por meio de reservas, empréstimos e renovações.
- **Atendente**: entrega livros, registra devoluções, inspeciona exemplares e participa do processo de cobrança quando há atraso.
- **Banco**: recebe o pagamento de boletos e informa sua quitação.

## Áreas envolvidas

As informações do processo ficam distribuídas entre diferentes áreas da organização:

- **RH**: pessoas e atendentes.
- **Acadêmico**: estudantes.
- **Biblioteca**: livros, reservas, empréstimos, renovações e devoluções.
- **Financeiro**: multas, boletos e quitações.
- **Banco**: serviço externo responsável pelo pagamento do boleto.

Essas áreas precisam trocar informações para completar os fluxos de negócio. Por exemplo, a Biblioteca precisa validar estudantes e atendentes cadastrados, enquanto o Financeiro precisa identificar o estudante responsável por uma multa.

## Cadastro

O Administrador pode cadastrar, consultar, atualizar e excluir:

- pessoas;
- estudantes;
- atendentes;
- livros.

Um estudante ou atendente só pode ser cadastrado se a pessoa à qual ele está associado já existir.

Ao cadastrar um livro, ele passa a estar disponível para uso.

## Reserva

Um estudante pode solicitar a reserva de um livro.

A reserva é aceita somente quando:

- o livro está cadastrado;
- o estudante está cadastrado;
- o livro está **LIVRE**.

Quando a reserva é aceita:

1. uma reserva é registrada para aquele estudante;
2. o livro passa para o estado **RESERVADO**.

Caso essas condições não sejam atendidas, a reserva é recusada.

## Empréstimo

Um livro só pode ser emprestado quando existe uma reserva registrada para o mesmo estudante.

Quando o empréstimo é efetivado:

1. a reserva deixa de estar ativa;
2. um empréstimo é registrado;
3. o livro passa para o estado **EMPRESTADO**;
4. o Atendente entrega o exemplar ao Estudante.

## Renovação

Um estudante pode renovar um empréstimo já existente.

A renovação é permitida quando:

- o livro está **EMPRESTADO** ou **RENOVADO** para aquele estudante;
- o limite máximo de renovações ainda não foi atingido.

Cada renovação:

- acrescenta **30 dias** à data de vencimento;
- incrementa a quantidade de renovações;
- mantém o livro no estado **RENOVADO**.

Um empréstimo pode ser renovado no máximo **duas vezes**.

## Devolução e inspeção

O Estudante devolve o livro ao Atendente, que registra a devolução.

Nesse momento, o livro passa para o estado **DEVOLVIDO** e precisa ser inspecionado.

Após a inspeção:

- se estiver em bom estado, volta para **LIVRE**;
- se estiver danificado, passa para **REPARANDO**;
- depois do reparo, pode voltar a **LIVRE**;
- um exemplar sem possibilidade de reparação pode chegar ao estado final **INCINERADO**.

Também pode existir a necessidade de enviar um livro livre para reparo caso algum dano seja identificado no acervo.

## Multas

Se a devolução ocorrer depois da data de vencimento, deve ser gerada uma multa de **R$ 1,00 por dia de atraso**.

O fluxo envolve Biblioteca e Financeiro:

1. a Biblioteca identifica o atraso;
2. o Financeiro identifica o estudante responsável;
3. a multa é calculada;
4. um boleto é emitido;
5. o boleto é enviado ao estudante por e-mail.

O Estudante realiza o pagamento no Banco, que informa a quitação ao Financeiro. A multa então é registrada como quitada.

## Ciclo de vida do livro

Ao longo do processo, um livro pode assumir os seguintes estados:

- **LIVRE**
- **RESERVADO**
- **EMPRESTADO**
- **RENOVADO**
- **DEVOLVIDO**
- **REPARANDO**
- **INCINERADO**

O estado do livro deve representar de forma consistente sua situação real no fluxo de reserva, empréstimo, devolução e manutenção.

## Cenário técnico

O sistema é acessado por meio de uma **interface web** executada no navegador do usuário.

A aplicação de negócio é executada em um servidor de aplicação e se comunica com os serviços e bancos de dados necessários ao funcionamento dos módulos de RH, Acadêmico, Biblioteca e Financeiro.

O cenário também considera:

- comunicação web via **HTTPS**;
- persistência principal em **MySQL**;
- uma réplica dos dados em **Oracle**;
- comunicação da aplicação com o banco de dados via **JDBC**;
- replicação entre os servidores de dados;
- integração com um serviço bancário externo para cobrança e confirmação de pagamentos.

## O desafio

O objetivo desta kata é projetar uma arquitetura capaz de sustentar esse domínio e seus fluxos, mantendo consistência entre as diferentes áreas, regras de negócio e estados do livro.

Alguns pontos que tornam o problema interessante do ponto de vista arquitetural são:

- separação de responsabilidades entre áreas com dados distintos;
- comunicação entre módulos internos;
- integração com um sistema bancário externo;
- consistência do ciclo de vida de um livro;
- coordenação entre reserva, empréstimo, renovação e devolução;
- processamento de multas e pagamentos;
- evolução independente das diferentes partes do sistema;
- persistência e replicação de dados;
- exposição das funcionalidades por meio de uma aplicação web.

A solução arquitetural fica em aberto.
