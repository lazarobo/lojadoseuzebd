# lojadoseuzebd
Projeto de BD.  Enviar link do github com  Diagrama Entidade Relacionamento (se necessário) Arquivos com comandos pra criar e popular BD Exemplos de utilização do BD (idealmente com aplicação)

# Minimundo

O sistema proposto visa gerenciar as operações de vendas, a cadeia de suprimentos e o controle financeiro de lucro de uma pequena loja varejista.

O estabelecimento comercial possui diversos Clientes cadastrados, dos quais armazenamos o nome, CPF/CNPJ e o e-mail. Cada cliente é identificado por um código único, e não pode haver dois clientes com o mesmo e-mail ou o mesmo documento físico.

Para abastecer a loja, o sistema gerencia o cadastro de Fornecedores, de quem armazenamos a razão social, o CNPJ (único) e um telefone de contato. Cada fornecedor possui um código único de identificação.

A loja comercializa uma variedade de Produtos. Cada produto possui um nome, preço de venda atual e a quantidade disponível em estoque. Cada produto é identificado por um código único. Para manter a organização e relatórios, cada produto pertence obrigatoriamente a uma única Categoria, e uma categoria pode abrigar vários produtos. Cada categoria possui um código único e um nome.

A entrada de mercadorias no estoque ocorre por meio de uma Compra. O processo de compra funciona como uma entidade intermediária entre fornecedores e produtos. Sempre que a loja se abastece, registra-se qual fornecedor vendeu a mercadoria, qual produto foi adquirido, a quantidade comprada, a data da operação e o preço de custo unitário cobrado pelo fornecedor. Essa estrutura garante o rastreamento do preço de custo histórico, permitindo que o sistema calcule posteriormente o lucro líquido real obtido pela loja.

Quando um cliente realiza uma aquisição, será registrada uma Venda. A venda armazena a data e horário em que foi feita, o valor total bruto, o status atual (ex: 'PAGA', 'PENDENTE' ou 'CANCELADA') e fica associada ao cliente que realizou a operação. Cada venda é identificada por um código único. Para que uma venda seja concluída, é obrigatório registrar o seu Pagamento, onde mapeamos a forma de pagamento utilizada (ex: 'PIX', 'DINHEIRO') e o valor efetivamente pago.

Uma venda pode ter um ou vários produtos, e um mesmo produto pode ser vendido em diferentes ocasiões. Para gerenciar essa relação, cada venda possui Itens da Venda, onde é registrada a quantidade comprada de cada produto e o preço unitário praticado no momento exato daquela venda. Esse preço é armazenado no item para preservar o histórico financeiro, mesmo que o preço de varejo do produto mude no cadastro principal. Um mesmo produto aparece no máximo uma vez em cada venda, e cada item é identificado pela combinação (chave composta) do código da venda mais o código do produto.

Sempre que uma compra é registrada no sistema, a quantidade em estoque do produto aumenta. Inversamente, sempre que uma venda é concluída com sucesso, a quantidade em estoque de cada produto vendido deve ser reduzida de acordo com a quantidade informada no item.


# Diagrama lojadoseuzebd: 

[Link para o site do diagrama](https://dbdiagram.io/d/Copy-of-Banco-de-dados-Estoque-6ae531b28bf38111a8f0f3d2)

<img width="1175" height="684" alt="Image" src="https://github.com/user-attachments/assets/7bc3c296-8102-47ed-9c0c-2db4efe1d51f" />

# Justificativas (Mudanças)

O modelo inicial considerava apenas a saída de mercadorias (Vendas), mas agora adicionamos algumas mudanças sobre o estoque. Ao criar a tabela COMPRA integrada à entidade FORNECEDOR, isolamos o processo de abastecimento da cadeia de suprimentos (compras com preço de custo de um parceiro comercial) do processo de escoamento (vendas com preço de varejo ao cliente final). Isso se assemelha à operação de uma entrada de estoque.

Sem a tabela COMPRA (e o registro do custo de aquisição), o sistema do "Seu Zé" só conseguiria calcular o faturamento bruto, mascarando a real saúde financeira da lojinha. Ao registrar o preco_custo_unitario associado a cada produto e fornecedor no ato da reposição, o banco de dados ganhou inteligência para cruzar a entrada de mercadorias com as vendas por meio de subconsultas (CTE/WITH), permitindo extrair relatórios cruciais como o lucro líquido real e a margem de lucro por produto.

Um fornecedor pode vender muitos produtos para a loja, assim como um produto pode ser comprado de vários fornecedores diferentes ao longo do tempo. A tabela COMPRA atua perfeitamente como uma entidade associativa intermediária, quebrando essa relação complexa em duas relações do tipo "um para muitos" (1:N), respeitando as regras fundamentais de modelagem relacional.
