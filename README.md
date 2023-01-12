# TUTORIAL PARA ORGANIZAÇÃO DA BASE DE DADOS - ACESSIBILIDADE E CONECTIVIDADE DAS VIAS DE VIÇOSA

Este é um tutorial com o passo-a-passo de todo o procedimento metodológico utilizado para a preparação da base de dados do município de Viçosa do primeiro artigo que compõem a dissertação de mestrado. O objetivo do artigo é calcular a acessibilidade de suas principais vias. Os softwares utilizados durante o processo foram: `PostgreeSQL 14` (com o pacote de extensões para dados espaciais PostGis) através do gerenciador `pgAdmin`, `Python 3` (juntamente com a IDE Jupyter Notebook), `QGIS 3.22.7` e `ArcGIS 10.5`.

O objetivo desse tutorial é explicar de forma detalhada os procedimentos realizados para organizar a base de dados das vias de Viçosa e com isso, será possível entender o processo e replicá-lo para demais bases de dados.

## PASSO-A-PASSO:

### 1. ORIGEM DOS DADOS:

Os dados utilizados nesse trabalho foram fornecidos por Vieira e Castro (2021). O primeiro arquivo nomeado como `Rede_Viaria_RSU_2021` trata-se do shapefile contando as ruas de Viçosa e conta com os atributos como **Nome da Rua**, **Tipo de Eixo**, **Largura da rua**, **Pavimento**, **Declividade**, **Bairro**, entre outros. A malha viária de Vieira e Castro (2021) será utilizada para formar a rede de análise.

**Os arquivos `Rede_Viaria_RSU_2021` e `declividade_pontos_cotados_SAAE` estão localizados na pasta `DADOS INICIAIS`.**

### 2. SELEÇÃO DOS ATRIBUTOS UTEIS:

Diverssos atributos presentes em `Rede_Viaria_RSU_2021` não serão úteis para esse artigo. Portanto, da tabela de atributos dos dados originais, apenas o **sentido da via** permaneceu e todos os demais atributos foram excluidos através da ferramenta `Editar campos` do software `QGIS`. Quatro novos campos foram criados: **nome da rua** (referente ao nome da rua), e **tipo_pm**, sendo que o primeiro será utilizado para incluir o nome das vias e o segundo a classificação de acordo com o plano de mobilidade.

### 3. SELEÇÃO DAS VIAS A SEREM UTILIZADAS:
As vias mencionadas no Plano de Mobilidade de Viçosa foram selecionadas utilizando o `Open Street Map` e o `Google Maps` como pano de fundo. Durante esse processo os campos os campos **nome da rua** e **tipo_pm** foram preenchidos.

### 4. GARATINDO A INTEGRIDADE TOPOLÓGICA DA REDE:

Para garantir a integridade topológica da rede foi utilizado o complemento `Verificador de topologia` do software `QGIS`. No botão `Configurações`, seleciona-se o arquivo da malha viária e marca-se a opção `não devem ter dangles`, adiciona-se a regra e confirma. Na janela aberta clica-se em `Validar tudo`. Surgirão pontos vermelhos em alguns locais da malha viária e será possível descobrir quais linhas estão desconectadas (esses pontos vermelhos devem existir apenas onde a via realmente está interrompida, como as extremidades).

### 5. OBTENÇÃO DA REDE EM FORMA DE GRAFOS:

A malha viária obtida até então não está na forma de grafos. Uma rede viária na forma de grafos deve ser dividida apenas na junção entre os vértices. Para gerar a malha viária de Viçosa dessa forma utilizou-se o software `ArcGIS`. Primeiramente utilizou-se a ferramenta `Dissolve`para agregar todos elementos em apenas uma feição e para concluir utilizou-se a ferramenta `Explode Multipart Feature` da caixa de ferramentas `Advanced Editing`. Os atributos presentes no arquivo anterior devem ser reciados e novos também foram criados. Por fim, os atributos dessa rede foram: **nome da rua**, **tipo_pm** e **sentido da via**. Todos os atributos foram deixados em branco para serem prenchidos posteriormente.

**Esse arquivo foi nomeado como `vias_grafos_artigo1` e está localizado na pasta `DADOS_INTERMEDIARIOS`.**

