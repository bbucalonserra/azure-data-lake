# Educação Básica em Escolas em territórios indígenas

## Objetivo
A educação é um aspe­cto crucial do desenvolvimento humano e­ social, desempenhando um pape­l fundamental na criação de sociedade­s mais justas e igualitárias. O Brasil, com sua composição cultural diversificada e nume­rosos grupos étnicos, incluindo comunidades indígenas que contribue­m muito para a identidade da nação, exige­ uma consideração especial pe­la educação indígena. É importante priorizar a pre­servação das tradições culturais e a capacitação das comunidade­s nativas.

O objetivo de­ste projeto é examinar e­ analisar diferentes face­tas da educação indígena no Brasil. O foco será nos dados e­ducacionais existentes. Por me­io dessa análise, pretende-se abordar questões importante­s que fornecerão insights sobre­ a situação atual da educação indígena no país e de­stacarão as áreas que reque­rem atenção para melhorias.

## O Projeto
### 1. Busca Pelos Dados
A busca pelos dados foi realizada por meio do site do governo (https://www.gov.br/inep/pt-br/acesso-a-informacao), em que foi-se buscado tabelas referentes ao censo escolar do Brasil em 2022. Foram encontradas duas tabelas para fazer parte da análise: 
- [Microdados do Censo Escolar da Educacao no Brasil 2022](https://github.com/bbucalonserra/data_engineering/blob/main/tables_bronze/microdados_ed_basica_2022.xlsx)
- [Taxa Rendimento Escolas 2022](https://github.com/bbucalonserra/data_engineering/blob/main/tables_bronze/tx_rend_escolas_2022.xlsx)

A primeira tabela é referente a diversas informações sobre as escolas no Brasil (código da escola, têm ou nao internet, salas, etc.), possui 385 colunas. Já a segunda é em relação a taxa de rendimento escolar (notas, taxa de abandono, taxa de reprovação, etc.), possui 12 colunas.


### 2. Coleta
A coleta de dados foi realizada por meio do download de duas tabelas diretamente do site oficial do governo. Essa etapa foi crucial para garantir que os dados utilizados sejam precisos e confiáveis, uma vez que as fontes governamentais são reconhecidas como uma fonte primária de informações educacionais. A decisão de obter os dados diretamente do site do governo foi tomada para garantir que as informações coletadas estejam alinhadas com as fontes oficiais disponíveis

### 3. Modelagem e Carga
A modelagem de dados é um processo fundamental no campo da ciência da computação e da gestão de informações. A principal finalidade da modelagem de dados é garantir que os dados sejam organizados, armazenados e gerenciados de forma eficiente e precisa para atender às necessidades específicas de uma organização ou projeto. Aqui, Modelagem e Carga estão no mesmo tópico porque será utilizado diretamente um sistema de Data Lake, armazenando os dados por camadas. 

#### 3.1 Definição do Sistema de Cloud Computing
A escolha da `Azure` como sitema de cloud computing deste projeto foi baseada em sua reputação global como uma das principais plataformas de nuvem do mundo. Além disso, a Azure é conhecida por sua excelente integração com ferramentas amplamente utilizadas, como o Power BI, o que facilita a análise de dados. A plataforma Azure também se destaca pela sua segurança robusta e capacidade de escalabilidade, garantindo a proteção dos dados e sua flexibilidade.

#### 3.2 Recursos de Armazenamento
Inicialmente, foi-se criado uma conta na Azure de forma gratuita com uma quantia a ser utilizada por fins acadêmicos. Aqui, foi-se criado, em ordem:
- Um `Resource Group` (contêiner lógico que ajuda a gerenciar e organizar recursos relacionados no Azure)
- `Storage Account` (um recurso da Azure que fornece armazenamento em nuvem de arquivos), sendo utilizado o `Azure Data Lake Storage 2gen`
- Container Bronze, Silver e Gold

Com isto, foi-se inserido os dados coletados do site governo no container Bronze. Posteriormente, após a realização de ETL, os dados ficarão armazenados na camada Silver e, por fim, na camada Gold. A definição de cada camada utilizada foi:
- Bronze: dados brutos (raw) no formato da coleta, como JSON, CSV, XLS, Parquet
- Silver: dados limpos e transformados, removendo colunas indesejadas, caracteres estranhos e removendo espaços
- Gold: dados organizados, com join realizado entre tabelas caso necessário e com regras de negócios aplicados de acordo com as métricas / perguntas definidas que deverão ser respondidas

#### 3.3 ETL - Extract, Transform e Load (Bronze - Silver)
Após a inserção dos dados brutos na camada Bronze, a próxima etapa é a realização das transformações nos dados. Para tal atividade, foi-se utilizado o recurso `Azure Data Factory`, visto que, além de ser uma ferramenta visual e de fácil uso, as transformações necessárias não são avançadas. A linguagem utilizada por este recurso é chamada de "Linguagem de Expressão de Transformação de Dados" (Data Flow Expression Language). Essa linguagem permite que você defina transformações de dados usando uma sintaxe semelhante ao SQL e inclui funções e operadores para executar operações de transformação, filtragem, projeção e muito mais. Abaixo, estão as transformações utilizadas no Data Factory:

![ETL - Bronze para Silver](https://github.com/bbucalonserra/data_engineering/blob/main/pictures/ETL_bronze_to_silver.PNG)


Descrição das transformações:
- Coleta dos dados do Data Lake
- `SELECT`  para selecionar as colunas utilizadas na análise
- `DERIVED COLUMN` para para remoção de caracteres especiais e estranhos das colunas
- `SINK` para enviar os dados transformados de volta ao Data Lake, porém, agora armazenados na camada / container Silver

#### 3.4 Análise de Dados Silver
Agora, é necessário realizar verificações das transformações realizadas nos dados brutos. É possível realizar estas verificações no próprio `Azure Data Factory`, porém, como posteriormente será necessário realizar análises e, juntando ao fato de que a interface do `Azure Data Factory` é um pouco lenta para esta função, será usado o recurso Azure Databricks. Nesta etapa, também é comum realizar análise dos dados e verificar se as transformações foram eficientes e coletar avaliações dos dados da camada Silver. Estas avaliações serão utilizadas posteriormente para a criação da camada Gold.
Para criar uma conexão entre o Data Lake e o Databricks, será necessário criar os seguintes recursos:
- Databricks (plataforma de análise de dados na nuvem que combina recursos de big data e análise avançada)
- Registration App (entidade que representa uma aplicação ou serviço que deseja acessar os recursos na plataforma Azure. É identificação para um aplicativo ou serviço que quer fazer alterações na nuvem da Microsoft)
- Key Vault (serviço de gerenciamento de chaves e segredos)

Com os recursos criados, basta entrar no Databricks, criar um notebook e utilizar o seguinte código em `Spark`:

```py
service_credential = dbutils.secrets.get(scope="<scope>",key="<service-credential-key>")

spark.conf.set("fs.azure.account.auth.type.<storage-account>.dfs.core.windows.net", "OAuth")
spark.conf.set("fs.azure.account.oauth.provider.type.<storage-account>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
spark.conf.set("fs.azure.account.oauth2.client.id.<storage-account>.dfs.core.windows.net", "<application-id>")
spark.conf.set("fs.azure.account.oauth2.client.secret.<storage-account>.dfs.core.windows.net", service_credential)
spark.conf.set("fs.azure.account.oauth2.client.endpoint.<storage-account>.dfs.core.windows.net", "https://login.microsoftonline.com/<directory-id>/oauth2/token")
```
Em que:

- scope = secret scope, criado no próprio Databricks
- service-credential-key = credential key do Key Vault
- storage-account = Storage Account
- application-id = application ID do App Registration
- directory-id = directory ID do App Registration






