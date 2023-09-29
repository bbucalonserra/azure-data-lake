# Educação Básica em Escolas em territórios indígenas

## Objetivos
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

#### 2.1 Definição do Sistema de Cloud Computing
A escolha da Azure como sitema de cloud computing deste projeto foi baseada em sua reputação global como uma das principais plataformas de nuvem do mundo. Além disso, a Azure é conhecida por sua excelente integração com ferramentas amplamente utilizadas, como o Power BI, o que facilita a análise de dados. A plataforma Azure também se destaca pela sua segurança robusta e capacidade de escalabilidade, garantindo a proteção dos dados e sua flexibilidade.
