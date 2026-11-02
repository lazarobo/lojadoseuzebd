# Loja do Seu Zé — Sistema de Controle de Vendas e Estoque

## O que é o sistema

Sistema de banco de dados para uma pequena loja varejista. Cobre três frentes:

- Operações de venda
- Cadeia de suprimentos (de onde vem a mercadoria)
- Controle financeiro de lucro

O Seu Zé consegue registrar vendas, saber de onde veio cada produto, quanto pagou, e calcular o lucro real.

---

## Entidades

### Cliente

Quem compra na loja.

- **Atributos:** código (PK), nome, CPF/CNPJ (único), e-mail (único)
- **Restrições:** não pode ter dois clientes com mesmo CPF nem mesmo e-mail — evita cadastro duplicado

### Fornecedor

Quem abastece a loja.

- **Atributos:** código (PK), razão social, CNPJ (único), telefone
- **Exemplos:** Distribuidora Norte (bebidas/salgados), Tabacaria Central, Doces & Cia, Fazenda Boa Vista (frutas/laticínios)

### Categoria

Organiza os produtos para relatórios.

- **Atributos:** código (PK), nome
- **Categorias:** Bebida, Fruta, Cigarro, Salgado, Doce, Laticínio
- Todo produto pertence obrigatoriamente a uma categoria
- Uma categoria pode ter vários produtos

### Produto

O que a loja vende.

- **Atributos:** código (PK), nome, preço de venda atual, quantidade em estoque, categoria (FK)
- **Exemplos:** Cafézinho (Bebida, R$2,00), Coxinha (Salgado, R$2,50), Bad Moro (Cigarro, R$50,00)
- Estoque atualiza automaticamente: sobe na compra, desce na venda

### Compra

Entrada de mercadoria — liga fornecedor a produto.

- **Atributos:** código (PK), fornecedor (FK), produto (FK), quantidade comprada, preço de custo unitário, data
- **Por que é importante:** sem o custo registrado, o Seu Zé saberia o faturamento mas não o lucro real
- **Exemplo:** cafézinho comprado a R$0,80, vendido a R$2,00 → lucro de R$1,20 por unidade

### Venda

Quando o cliente compra algo.

- **Atributos:** código (PK), data/hora, valor total, status (PAGA / PENDENTE / CANCELADA), cliente (FK)
- Cada venda tem um código único

### Item da Venda

Detalhe de cada produto dentro de uma venda.

- **Atributos:** código da venda (FK), código do produto (FK), quantidade, preço unitário gravado
- **Chave primária composta:** código da venda + código do produto (mesmo produto aparece no máximo 1 vez por venda — se quer 5 cafézinhos, é 1 linha com quantidade 5)
- **Preço gravado:** é o preço no momento da venda. Se o Seu Zé aumentar o cafézinho amanhã, as vendas de hoje não mudam. Preço no produto = atual, preço no item = histórico

### Pagamento da Venda

Como o cliente pagou.

- **Atributos:** código (PK), venda (FK), forma de pagamento (PIX, DINHEIRO), valor pago
- **Por que é tabela separada e não uma coluna na venda:** permite pagamento dividido (ex: venda 7 — R$5,00 em dinheiro + R$4,50 no PIX)
- Vendas PENDENTE e CANCELADA não têm pagamento

---

## Controle de estoque

- Compra registrada → estoque do produto **sobe**
- Venda concluída → estoque **desce**
- Venda cancelada → estoque **volta**
- O sistema valida antes de vender: se não tem estoque suficiente, a venda não é registrada

---

## Ciclo completo

```
Fornecedor → Compra → Estoque do Produto → Venda → Item da Venda → Pagamento
                                                          ↓
                                                       Cliente
```

Mercadoria entra pelo fornecedor, fica no estoque, sai pela venda pro cliente, e o pagamento fecha o ciclo financeiro.

---

# Diagrama lojadoseuzebd: 

[Link para o site do diagrama](https://dbdiagram.io/d/Copy-of-Banco-de-dados-Estoque-6ae531b28bf38111a8f0f3d2)

<img width="1175" height="684" alt="Image" src="https://github.com/user-attachments/assets/7bc3c296-8102-47ed-9c0c-2db4efe1d51f" />

# Justificativas (Mudanças)

O modelo inicial considerava apenas a saída de mercadorias (Vendas), mas agora adicionamos algumas mudanças sobre o estoque. Ao criar a tabela COMPRA integrada à entidade FORNECEDOR, isolamos o processo de abastecimento da cadeia de suprimentos (compras com preço de custo de um parceiro comercial) do processo de escoamento (vendas com preço de varejo ao cliente final). Isso se assemelha à operação de uma entrada de estoque.

Sem a tabela COMPRA (e o registro do custo de aquisição), o sistema do "Seu Zé" só conseguiria calcular o faturamento bruto, mascarando a real saúde financeira da lojinha. Ao registrar o preco_custo_unitario associado a cada produto e fornecedor no ato da reposição, o banco de dados ganhou inteligência para cruzar a entrada de mercadorias com as vendas por meio de subconsultas (CTE/WITH), permitindo extrair relatórios cruciais como o lucro líquido real e a margem de lucro por produto.

Um fornecedor pode vender muitos produtos para a loja, assim como um produto pode ser comprado de vários fornecedores diferentes ao longo do tempo. A tabela COMPRA atua perfeitamente como uma entidade associativa intermediária, quebrando essa relação complexa em duas relações do tipo "um para muitos" (1:N), respeitando as regras fundamentais de modelagem relacional.
