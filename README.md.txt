
# Utilizando AI Search para indexação e consulta de Dados

Olá a todos! O objetivo desse repositório é descrever o passo a passo sobre criar uma solução de mineração de conhecimento que facilite a pesquisa de insights sobre as experiências do cliente.

Isso faz parte do Desafio de Projeto do curso "Microsoft Azure AI Fundamentals" em parceria com a [DIO](https://dio.me)!

Serão utilizados os recursos: 
- Pesquisa de IA do Azure
- Serviço de IA do Azure
- Conta de armazenamento com contêineres de blob

As intruções de como criar esses recursos já foram passadas em labs anteriores.

## 👣 Passo a Passo para carregar documentos no Armazenamento do Azure
1. No painel de menu esquerdo da conta de armazenamento, selecione Contêineres.
2. Selecione **+ Contêiner**. Um painel do lado direito é aberto. 
3. Insira as seguintes configurações e clique em Criar:
- Nome: café-comentários
- Nível de acesso público: Contêiner (acesso de leitura anônimo para contêineres e blobs)
- Avançado: sem alterações.

4. Em uma nova guia do navegador, baixe as [revisões de café compactadas](https://aka.ms/mslearn-coffee-reviews) do e extraia os arquivos para a pasta de comentários.
5. No portal do Azure, selecione seu contêiner de avaliações de café. No contêiner, selecione Carregar.
6. No painel Carregar blob, selecione Selecionar um arquivo.
7. Na janela do Explorer, selecione todos os arquivos na pasta de comentários, selecione Abrir e selecione Carregar.
8. Depois que o carregamento for concluído, você poderá fechar o painel Carregar blob. Seus documentos agora estão em seu recipiente de armazenamento de revisões de café.

## 👣 Passo a Passo para indexar os documentos
1. No portal do Azure, navegue até seu recurso de Pesquisa de IA do Azure. Na página Visão geral, selecione Importar dados.
2. Na página Conectar aos seus dados, na lista Fonte de Dados, selecione Armazenamento de Blobs do Azure. Conclua os detalhes do armazenamento de dados com os seguintes valores:
- Fonte de dados: Armazenamento de Blobs do Azure
- Nome da fonte de dados: coffee-customer-data
- Dados a serem extraídos: conteúdo e metadados
- Modo de análise: Padrão
- Cadeia de conexão: *Selecione Escolher uma conexão existente. Selecione sua conta de armazenamento, selecione o contêiner de revisões de café e clique em Selecionar.
- Autenticação de identidade gerenciada: Nenhuma
- Nome do contêiner: essa configuração é preenchida automaticamente depois que você escolhe uma conexão existente.
- Pasta Blob: deixe isso em branco.
- Descrição: Comentários a Fourth Coffee shops.

3. Selecione Avançar: Adicionar habilidades cognitivas (Opcional).
4. Na seção Anexar Serviços Cognitivos, selecione seu recurso de serviços de IA do Azure.
5. Na seção Adicionar enriquecimentos:
- Altere o nome do Skillset para coffee-skillset.
- Marque a caixa de seleção Habilitar OCR e mesclar todo merged_content texto em campo
- Verifique se o campo Dados de origem está definido como merged_content.
- Altere o nível de granularidade de enriquecimento para Páginas (blocos de 5000 caracteres)
- Não selecione Habilitar enriquecimento incremental
- Selecione os seguintes campos enriquecidos:

| Habilidade Cognitiva   |      Parâmetro      |  Nome do campo |
|----------|:-------------:|------:|
| Extrair nomes de locais |   | Locais |
| Extrair frases-chave |      |   Frases-chave |
| Detectar sentimento |  |    sentimento |
| Gerar tags a partir de imagens |  |    imageTags |
| Gerar legendas a partir de imagens |  |    imageCaption |

6. Em Salvar enriquecimentos em um repositório de conhecimento, selecione:
- Projeções de imagens
- Documentos
- Páginas
- Frases-chave
- Entidades
- Detalhes da imagem
- Referências de imagens

7. Selecione Projeções de blob do Azure: Documento. Uma configuração para Nome do contêiner com as exibições preenchidas automaticamente pelo contêiner do repositório de conhecimento. Não altere o nome do contêiner.
8. Selecione Avançar: Personalizar índice de destino. Altere o nome do índice para coffee-index.
9. Verifique se a chave está definida como metadata_storage_path. Deixe o nome do Sugeridor em branco e o modo de Pesquisa preenchido automaticamente.
10. Revise as configurações padrão dos campos de índice. Selecione filtrável para todos os campos que já estão selecionados por padrão.
11. Selecione Avançar: Criar um indexador.
12. Altere o nome do indexador para coffee-indexer.
13. Deixe a Agenda definida como Uma vez.
14. Expanda as opções Avançadas. Verifique se a opção Base-64 Encode Keys está selecionada, pois as chaves de codificação podem tornar o índice mais eficiente.
15. Selecione Enviar para criar a fonte de dados, o conjunto de habilidades, o índice e o indexador. O indexador é executado automaticamente e executa o pipeline de indexação, que:
- Extrai os campos de metadados do documento e o conteúdo da fonte de dados.
- Executa o conjunto de habilidades cognitivas para gerar campos mais enriquecidos.
- Mapeia os campos extraídos para o índice.
16. Na metade inferior da página Visão geral do recurso de Pesquisa de IA do Azure, selecione a guia Indexadores. Esta guia mostra o indexador de café recém-criado. Aguarde um minuto; Atualize até que o Status indique êxito.
17. Selecione o nome do indexador para ver mais detalhes.


## 👣 Passo a Passo para consultar o indice
1. Na página Visão geral do serviço de Pesquisa, selecione Gerenciador de pesquisa na parte superior da tela.
2. Observe como o índice selecionado é o índice de café que você criou. No campo Cadeia de caracteres de consulta, insira e selecione Pesquisar. A consulta de pesquisa retorna todos os documentos no índice de pesquisa, incluindo uma contagem de todos os documentos no campo @odata.count. O índice de pesquisa deve retornar um documento JSON contendo os resultados da pesquisa. `search=*&$count=true`
3. Agora vamos filtrar por localização. Insira no campo Cadeia de caracteres de consulta e selecione Pesquisar. A consulta pesquisa todos os documentos no índice e filtra por revisões com um local de Chicago. `search=locations:'Chicago'`
4. Agora vamos filtrar por sentimento. Insira no campo Cadeia de caracteres de consulta e selecione Pesquisar. A consulta pesquisa todos os documentos no índice e filtra por avaliações com um sentimento negativo. `search=sentiment:'negative'`
5. Um dos problemas que podemos querer resolver é por que pode haver certas revisões. Vamos dar uma olhada nas frases-chave associadas à avaliação negativa. O que você acha que pode ser a causa da revisão?

## 👣 Passo a Passo para revisar o repositório de conhecimento
1. No portal do Azure, navegue de volta para sua conta de armazenamento do Azure.
2. No painel de menu esquerdo, selecione Contêineres. Selecione o contêiner de armazenamento de conhecimento.
3. Selecione qualquer um dos itens e clique no arquivo objectprojection.json.
4. Selecione Editar para ver o JSON produzido para um dos documentos do armazenamento de dados do Azure.
5. Selecione a trilha de blob de armazenamento no canto superior esquerdo da tela para retornar aos Contêineres da conta de armazenamento.
6. Em Contêineres, selecione o contêiner coffee-skillset-image-projection. Selecione qualquer um dos itens.
7. Selecione qualquer um dos .jpg arquivos. Selecione Editar para ver a imagem armazenada do documento. Observe como todas as imagens dos documentos são armazenadas dessa maneira.
8. Selecione a trilha de blob de armazenamento no canto superior esquerdo da tela para retornar aos Contêineres da conta de armazenamento.
9. Selecione Navegador de armazenamento no painel esquerdo e selecione Tabelas. Há uma tabela para cada entidade no índice. Selecione a tabela coffeeSkillsetKeyPhrases.

Veja as frases-chave que a loja de conhecimento conseguiu capturar do conteúdo das avaliações. Muitos dos campos são chaves, portanto, você pode vincular as tabelas como um banco de dados relacional. O último campo mostra as frases-chave que foram extraídas pelo conjunto de habilidades.


