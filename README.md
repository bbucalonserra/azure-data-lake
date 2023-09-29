# Educação Básica nas Escolas em territórios indígenas

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
A modelagem de dados é um processo fundamental no campo da ciência da computação e da gestão de informações. A principal finalidade da modelagem de dados é garantir que os dados sejam organizados, armazenados e gerenciados de forma eficiente e precisa para atender às necessidades específicas de uma organização ou projeto. **Aqui, Modelagem e Carga estão no mesmo tópico porque será utilizado diretamente um sistema de Data Lake, armazenando os dados por camadas.**

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


#### 3.3 Conexão Data Lake e Databricks
Agora, é necessário realizar verificações das transformações realizadas nos dados brutos. Para isto, será usado o recurso Azure Databricks. 
Para criar uma conexão entre o Data Lake e o Databricks, será necessário criar os seguintes recursos:
- Databricks (plataforma de análise de dados na nuvem que combina recursos de big data e análise avançada)
- Registration App (entidade que representa uma aplicação ou serviço que deseja acessar os recursos na plataforma Azure. É identificação para um aplicativo ou serviço que quer fazer alterações na nuvem da Microsoft)
- Key Vault (serviço de gerenciamento de chaves e segredos)

Com os recursos criados, basta entrar no Databricks, criar um notebook e utilizar o seguinte código em `Spark`:

<details>
  <summary>Mostrar Código</summary>
  
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

</details>
</details>

Feito isto, tem-se uma conexão entre o Databricks e o Data Lake. Agora já é possível criar tabelas e popula-las com os dados do Lake.


#### 3.4 Criação de Schema
Dentro do Databricks, por viés organizacional, será necessário criar schemas para armazenas as tabelas de análises. Será criado um schema por camada do Data Lake. Para isto, basta abrir um notebook e utilizar os seguintes comandos em SQL:

```py
CREATE SCHEMA bronze;

CREATE SCHEMA silver;

CREATE SCHEMA gold;
```

### 3.5 Criação das Tabelas da Camada Bronze
No próprio Databricks, será aberto um notebook para verificar a qualidade dos dados presentes na camada Bronze. Para isto, a utilização de SPARK para leitura dos dados em CSV armazenados como `BLOBS` será utilizada em conjunto a criação de views:

**Tabela microdados_ed_basica_2022**

Visualização da Tabela
```py
spark.read.options(delimiter = ';', header = True).csv('abfss://bronze@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/microdados_ed_basica_2022.csv').display()
```
Criação de View
```py
spark.read.options(delimiter = ';', header = True).csv('abfss://bronze@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/microdados_ed_basica_2022.csv').createOrReplaceTempView('microdados_ed_basica_2022')
```
**Tabela tx_rend_escolas_2022**

Visualização da Tabela
```py
spark.read.options(delimiter = ';', header = True).csv('abfss://bronze@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/tx_rend_escolas_2022.csv').display()
```
Criação de View
```py
spark.read.options(delimiter = ';', header = True).csv('abfss://bronze@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/tx_rend_escolas_2022.csv').createOrReplaceTempView('microdados_ed_basica_2022')
```

Com isto, foi-se visto algumas inconsistências nos dados, como caracteres especiais e colunas indesejadas.
Foi feito o armazenamento destes dados no schema BRONZE. Para esta atividade, basta utilizar comandos em SQL:

**Tabela microdados_ed_basica_2022**
```py
CREATE TABLE bronze.educacao_basica_2022 USING CSV LOCATION 'abfss://bronze@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/microdados_ed_basica_2022.csv'
OPTIONS (
  header = "true",
  delimiter = ";"
)
```
**Tabela tx_rend_escolas_2022**
```py
CREATE TABLE bronze.rend_escolar_2022
USING CSV LOCATION 'abfss://bronze@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/tx_rend_escolas_2022.csv'
OPTIONS (
  header = "true",
  delimiter = ";"
)
```

Observação: a tipologia dos dados ainda não foi definida por serem dados crus (raw). Elas serão definidas na camada Silver.


#### 3.6 ETL - Extract, Transform e Load (Bronze - Silver)
Após a inserção dos dados brutos na camada Bronze, selecionado as colunas, notado algumas inconsistências nos dados e criado as tabelas, a próxima etapa é a realização das transformações. Para tal atividade, foi-se utilizado o recurso `Azure Data Factory`, visto que, além de ser uma ferramenta visual e de fácil uso, as transformações necessárias não são avançadas. A linguagem utilizada por este recurso é chamada de "Linguagem de Expressão de Transformação de Dados" (Data Flow Expression Language). Essa linguagem permite que você defina transformações de dados usando uma sintaxe semelhante ao SQL e inclui funções e operadores para executar operações de transformação, filtragem, projeção e muito mais. Abaixo, estão as transformações utilizadas no Data Factory:

![ETL - Bronze para Silver](https://github.com/bbucalonserra/data_engineering/blob/main/pictures/ETL_bronze_to_silver.PNG)


Descrição das transformações:
- Coleta dos dados do Data Lake
- `SELECT`  para selecionar as colunas utilizadas na análise
- `DERIVED COLUMN` para para remoção de caracteres especiais e estranhos das colunas
- `SINK` para enviar os dados transformados de volta ao Data Lake, porém, agora armazenados na camada / container Silver



#### 3.7 Criação das Tabelas da Camada Silver
A próxima etapa é análisar os dados resultantes da ETL da camada Bronze para Silver. Para isto, será necessário criar as novas tabelas após a ETL no Databricks já com a **tipologia dos dados definida e as variáveis de null ou not null também**:

**Tabela Educacao_basica_2022**
```py
CREATE TABLE silver.educacao_basica_2022
  (NU_ANO_CENSO INT NOT NULL,
  NO_REGIAO STRING NOT NULL,
  NO_UF STRING NOT NULL,
  NO_MESORREGIAO STRING NOT NULL,
  NO_ENTIDADE STRING NOT NULL,
  CO_ENTIDADE INTEGER NOT NULL,
  TP_DEPENDENCIA INT,
  TP_CATEGORIA_ESCOLA_PRIVADA INT,
  TP_LOCALIZACAO INT,
  QT_MAT_BAS INT,
  QT_MAT_BAS_FEM INT,
  QT_MAT_BAS_MASC INT,
  QT_EQUIP_DVD INT,
  QT_EQUIP_TV INT,
  QT_EQUIP_LOUSA_DIGITAL INT,
  QT_EQUIP_MULTIMIDIA INT,
  QT_EQUIP_VIDEOCASSETE INT,
  QT_EQUIP_PARABOLICA INT,
  QT_EQUIP_COPIADORA INT,
  QT_EQUIP_RETROPROJETOR INT,
  QT_EQUIP_IMPRESSORA INT,
  QT_EQUIP_IMPRESSORA_MULT INT,
  QT_EQUIP_FAX INT,
  QT_EQUIP_FOTO INT,
  QT_COMPUTADOR INT,
  QT_COMP_ADMINISTRATIVO INT,
  QT_SALAS_EXISTENTES INT,
  IN_INTERNET INT,
  IN_EDUCACAO_INDIGENA INT,
  TP_INDIGENA_LINGUA INT,
  CO_LINGUA_INDIGENA_1 INT,
  CO_LINGUA_INDIGENA_2 INT,
  CO_LINGUA_INDIGENA_3 INT,
  IN_MATERIAL_PED_INDIGENA INT)
USING CSV LOCATION 'abfss://silver@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/educacao_basica_2022_silver'
OPTIONS (
  header = "true",
  delimiter = ","
)
```

**Tabela tx_rend_escolas_2022**
```py
CREATE TABLE silver.tx_rend_escolas_2022 
(
  Ano INT NOT NULL, 
  Regiao STRING NOT NULL,
  UF STRING NOT NULL,
  Codigo_do_Municipio INT NOT NULL,
  Nome_do_Municipio STRING,
  Codigo_da_Escola INT NOT NULL,
  Nome_da_Escola STRING,
  Localizacao STRING,
  Dependencia_Administrativa STRING,
  Taxa_de_Aprovacao_Educacao_Basica FLOAT,
  Taxa_de_Reprovacao_Educacao_Basica FLOAT,
  Taxa_de_Abandono_Educacao_Basica FLOAT
)
USING CSV LOCATION 'abfss://silver@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/tx_rend_escolas_2022_silver'
OPTIONS (
  header = "true",
  delimiter = ","
)
```

#### 3.8 ETL - Extract, Transform e Load (Silver - Gold)
Agora, será realizado a segunda e última ETL que será referente a camada Silver para a Gold. Aqui, foi-se feito o join das duas tabelas através da coluna de Código da Escola (1:1), realizado a soma total de equipamentos por escola (visto que para a análise, apenas é interessante saber a quantidade total e nao separada por tipo de equipamento) e remoção de mais algumas colunas nao utilizadas:

![ETL - Silver para Gold](https://github.com/bbucalonserra/data_engineering/blob/main/pictures/ETL_silver_to_gold.PNG)

Descrição das transformações:
- Coleta dos dados do Data Lake
- `JOIN`  para juntar ambas as tabelas
- `SELECT` para remoção de algumas colunas
- `DERIVED COLUMN` para remoção de caracteres especiais remanescentes
- `SINK` para enviar os dados transformados de volta ao Data Lake, porém, agora armazenados na camada / container Gold

#### 3.9 Criação das Tabelas da Camada Gold
Por fim, agora é possível realizar a análise final de uma forma muito mais prática, rápida e consistente, visto que apenas possuimos colunas utilizáveis e de acordo com as regras de negócio das análises. 

``` py
CREATE TABLE gold.educacao_rend_escolas_joined
(
  Ano_Censo INT NOT NULL,
  Regiao STRING NOT NULL,
  Nome_UF STRING NOT NULL,
  UF STRING NOT NULL,
  Nome_do_Municipio STRING,
  Codigo_do_Municipio INTEGER,
  Dependencia INTEGER,
  Dependencia_Administrativa STRING,
  Localizacao STRING,
  Tipo_Localizacao INTEGER,
  Codigo_da_Escola INTEGER NOT NULL,
  Nome_da_Escola STRING,
  Categoria_Escola_Privada INTEGER,
  Localizacao_Diferenciada INTEGER,
  Matriculas_Eduacao_Basica INTEGER NOT NULL,
  Matriculas_Educacao_Basica_Femino INTEGER NOT NULL,
  Matriculas_Educacao_Basica_Masculino INTEGER NOT NULL,
  Total_Equipamentos INTEGER,
  Computadores INTEGER,
  Computadores_Administrativos INTEGER,
  Salas_Existentes INTEGER,
  Internet INTEGER,
  Educacao_Indigena INTEGER,
  Lingua_Indigena INTEGER,
  Lingua_Indigena_1 INTEGER,
  Lingua_Indigena_2 INTEGER,
  Lingua_Indigena_3 INTEGER,
  Material_Indigena INTEGER,
  Taxa_de_Aprovacao_Educacao_Basica FLOAT, 
  Taxa_de_Reprovacao_Educacao_Basica FLOAT,
  Taxa_de_Abandono_Educacao_Basica FLOAT,
  PRIMARY KEY "Codigo_da_Escola"
)

USING CSV LOCATION 'abfss://gold@educacaobasica.dfs.core.windows.net/microdados_ed_basica_2022/educacao_rend_escolas_joined'
OPTIONS (
  header = "true",
  delimiter = ","
)
```


### 4. Análise


1. Qual é a taxa de abandono escolar nas escolas indígenas em relação ao total de matrículas?  Este valor é mais mais ou menos elevado que escolas comuns?

<details>
  <summary>Mostrar Resposta</summary>
  
<img src="https://github.com/bbucalonserra/data_engineering/blob/main/graphics/Taxa_de_Abandono.PNG" align="left"
     alt="taxa_de_abandono">

Resposta: A taxa de abandono escolar nas escolas indígenas é de 18.59%, enquanto nas escolas comuns é de 7.32%. Portanto, podemos concluir que a taxa de abandono escolar nas escolas indígenas é consideravelmente mais elevada do que nas escolas comuns, com 11.27% maior que escolas comuns. Isso sugere que as escolas indígenas podem enfrentar desafios adicionais ou diferentes que contribuem para uma taxa de abandono mais alta em comparação com as escolas não indígenas. É importante investigar e abordar esses desafios para melhorar o acesso e a qualidade da educação para as comunidades indígenas.

</details>
</details>


