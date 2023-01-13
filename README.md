# TUTORIAL PARA ORGANIZAÇÃO DA BASE DE DADOS - ACESSIBILIDADE E CONECTIVIDADE DAS VIAS DE VIÇOSA

Este é um tutorial com o passo-a-passo de todo o procedimento metodológico utilizado para a preparação da base de dados do município de Viçosa do primeiro artigo que compõem a dissertação de mestrado. O objetivo do artigo é calcular a acessibilidade de suas principais vias. Os softwares utilizados durante o processo foram: `PostgreeSQL 14` (com o pacote de extensões para dados espaciais PostGis) através do gerenciador `pgAdmin`, `Python 3` (juntamente com a IDE Jupyter Notebook), `QGIS 3.22.7` e `ArcGIS 10.5`.

O objetivo desse tutorial é explicar de forma detalhada os procedimentos realizados para organizar a base de dados das vias de Viçosa e com isso, será possível entender o processo e replicá-lo para demais bases de dados.

## PASSO-A-PASSO:

### 1. ORIGEM DOS DADOS:

Os dados utilizados nesse trabalho foram fornecidos por Vieira e Castro (2021). O primeiro arquivo nomeado como `Rede_Viaria_RSU_2021` trata-se do shapefile contando as ruas de Viçosa e conta com os atributos como **Nome da Rua**, **Tipo de Eixo**, **Largura da rua**, **Pavimento**, **Declividade**, **Bairro**, entre outros. A malha viária de Vieira e Castro (2021) será utilizada para formar a rede de análise.

**Os arquivos `Rede_Viaria_RSU_2021` e `declividade_pontos_cotados_SAAE` estão localizados na pasta `DADOS INICIAIS`.**

### 2. SELEÇÃO DOS ATRIBUTOS UTEIS:

Diverssos atributos presentes em `Rede_Viaria_RSU_2021` não serão úteis para esse artigo. Portanto, da tabela de atributos dos dados originais, apenas o **sentido da via** permaneceu e todos os demais atributos foram excluidos através da ferramenta `Editar campos` do software `QGIS`. Quatro novos campos foram criados: **nome da rua** e **tipo_pm**, sendo que o primeiro será utilizado para incluir o nome das vias e o segundo a classificação de acordo com o plano de mobilidade.

### 3. SELEÇÃO DAS VIAS A SEREM UTILIZADAS:
As vias mencionadas no Plano de Mobilidade de Viçosa foram selecionadas utilizando o `Open Street Map` e o `Google Maps` como pano de fundo. Durante esse processo os campos os campos **nome da rua** e **tipo_pm** foram preenchidos.

### 4. GARATINDO A INTEGRIDADE TOPOLÓGICA DA REDE:

Para garantir a integridade topológica da rede foi utilizado o complemento `Verificador de topologia` do software `QGIS`. No botão `Configurações`, seleciona-se o arquivo da malha viária e marca-se a opção `não devem ter dangles`, adiciona-se a regra e confirma. Na janela aberta clica-se em `Validar tudo`. Surgirão pontos vermelhos em alguns locais da malha viária e será possível descobrir quais linhas estão desconectadas (esses pontos vermelhos devem existir apenas onde a via realmente está interrompida, como as extremidades).

### 5. OBTENÇÃO DA REDE EM FORMA DE GRAFOS:

A malha viária obtida até então não está na forma de grafos. Uma rede viária na forma de grafos deve ser dividida apenas na junção entre os vértices. Para gerar a malha viária de Viçosa dessa forma utilizou-se o software `ArcGIS`. Primeiramente utilizou-se a ferramenta `Dissolve`para agregar todos elementos em apenas uma feição e para concluir utilizou-se a ferramenta `Explode Multipart Feature` da caixa de ferramentas `Advanced Editing`. Os atributos presentes no arquivo anterior devem ser reciados e novos também foram criados. Por fim, os atributos dessa rede foram: **nome_rua** (referente aos nomes da ruas que compõe a linha), **comp_via** (comprimento total da linha) **comp_nome_rua** (referente ao comprimento das ruas que compõem a linha), **tipo_pm** (referente a classificação de acordo com o plano de mobilidade), **comp_tipo_pm** (comprimento da rua referente ao plano de mobilidade) e **sentido da via** (referente se a rua é sentido único ou não). Todos os atributos foram deixados em branco para serem prenchidos posteriormente.

**Esse arquivo foi nomeado como `vias_grafos` e está localizado na pasta `DADOS_INTERMEDIARIOS`.**

### 6. DIVIDINDO A MALHA VIÁRIA (GERADA APÓS O PASSO 4) EM DIVERSAS MALHAS COM SEUS RESPECTIVOS ATRIBUTOS:

Antes de preencher os atributos do arquivo `vias_grafos`, realizou-se um procedimento com a malha obtida do `passo 4`. A partir dessa malha, gerou-se três  novos arquivos utilizando a ferramenta Dissolve do software `ArcGIS`, com o objetivo de separar cada atributo da rede em um arquivo diferente.

Na ferramenta mencionada selecionou-se a malha viária (que não está na forma de grafos) com todos atributos e na aba `Dissolve_Fields (optional)` marcou-se um dos atributos desejados para a separação. Na aba `Output Feature Class` atribuiu-se o nome do novo arquivo (de forma que remeta ao atributo selecionado) e executou-se a ferramenta. Esse processo deve ser feito duas vezes para gerar os arquivos para cada um dos três atributos (**nome da rua**, **tipo_pm**, **sentido da via**). Esses três novos dados foram manipulados pela ferramenta `Explodir linhas` do software `QGIS`, com o objetivo de obter os atributos de cada segmento de reta presente na malha viária. Os três arquivos foram nomeados como `vias_nomeruas` (com o nome de rua de cada segmento de reta), `vias_tipopm` (com o tipo de classificação de via de acordo com o Plnao de Mobilidade de Viçosa para cada segmento de reta) e `vias_oneway` (com o sentido da via para cada segmento de reta).

**Os cinco arquivos se encontram na pasta `DADOS_INTERMEDIARIOS`**

### 7. IMPORTANDO TODOS OS DADOS PARA O BANCO DE DADOS

Todos arquivos gerados foram importados para o banco de dados. O banco usado nessa dissertação possui o nome `dissertacao_artigo1` e possui todas extensões espaciais provenientes do PostGIS.

Os arquivos `vias_grafos`, `vias_nomeruas`, `vias_tipopm` e `vias_oneway` foram importados através da ferramenta `Gerenciador BD...` disponível no software `QGis`. O procedimento é feito selecionando de forma individual o arquivo que será importado na aba **Entrada**, em **Esquema** seleciona-se a opção _public_ e em **Tabela** atribuiu-se o nome desejado da tabela (igual ao nome do arquivo em formato shapefile). Isso foi feito para todos os arquivos.

### 8. COMPLETANDO O ARQUIVO VIAS_GRAFOS

Os atributos do arquivo `vias_grafos` estão majoritariamente em branco. Para preenchê-los utilizou-se da lingaguem Python através da IDE `Jupyter Notebook`. O procedimento necessita da instalação, através do prompt de comando, do pacote `psycopg2` (necessário para manipular o banco de dados dentro da linaguem Python). Depois disso é possível executar os comandos em lingaguem Python.

**Todos os scripts explicados estão localizados na pasta `SCRIPTS_DISSERTACAO`. Essa pasta se divide em duas sub-pastas: SCRIPTS_SQL (com os codigos executados em linguagem SQL) e SCRIPTS_PYTHON (com o ambiente virtual utilizado nesse trabalho). Para acessar os Scripts que estão separados individualmente é necessário acessar a pasta projetos que está localizada em SCRIPTS_PYTHON >> av_dissertacao >> projetos.**

#### 8.1. INSTALAÇÃO DO PACOTE psycopg2

Para instalar o pacote psycopg2 é necessário digitar o seguinte comando no **prompt de comandos**:

    pip install psycopg2

#### 8.2. SCRIPTS EM PYTHON:

Os comandos descritos a seguir foram todos executados no **Jupyter Notebook**.

#### 8.2.1. IMPORTANDO O PACOTE psycopg2:

    import psycopg2 as pg

#### 8.2.2. CONECTANDO COM O BANCO DE DADOS:

Inseriu-se as configurações do banco de dados. Ele está sendo utilizado no servidor local (host = 'localhost'), o nome do banco de dados é dissertacao (database = 'dissertacao_artigo1'), o nome de usuário é postgres (user='postgres') e a senha do banco de dados é admin (password = 'admin').

    con = pg.connect(host='localhost', 
                    database='dissertacao_artigo1',
                    user='postgres', 
                    password='admin')

    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL

    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado DISSERTAÇÃO, que possui usuário postgres e senha admin.

#### 8.2.3. VENDO QUANTOS GRAFOS A MALHA POSSUI

É necessário descobrir quantas linhas a rede possui. Para isso executa-se os seguintes comandos:

    tabela_grafos = 'vias_grafos' #TABELA COM OS GRAFOS
    sql = f'select max(id) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA


#### 8.2.4. ATRIBUINDO NOME DAS VIAS PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais nomes de ruas presentes na tabela `vias_nomeruas` estão contidas na linha com id em análise. Esses nomes (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'nome_rua' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_nomeruas' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vnr' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'nome_rua' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    atributo_comp_grafo = 'comp_nome_rua' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS COMPRIMENTOS DOS NOMES DAS RUAS NA TABELA DO SHP DOS GRAFOS

    #ADICIONANDO O NOME DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f"SELECT {abr_grafos}.id FROM {tabela_grafos} {abr_grafos}, {tabela_grafos_anel} {abr_grafos_a} WHERE ST_Contains({abr_grafos}.geom, {abr_grafos_a}.geom) and {abr_grafos_a}.id = {id_i}" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ID DA TABELA DOS DOS GRAFOS (QUE JÁ ESTÁ COMPLETA).

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        sql = f'select sum(st_length({abr_dados}.geom)) from {tabela_dados} {abr_dados}, {tabela_grafos} {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados_grafo}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O COMPRIMENTO DE VIA DOATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_comp_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS DE COMPRIMENTOS SERÃO TRANSFORMADAS EM UMA STRING DA SEGUINTE FORMA: COMP_1; COMP_2; COMP_3; ... ; COMP_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_comp_str = lista_comp_str + str(dado) + '; '
            else:
                lista_comp_str = lista_comp_str + str(dado)

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DE COMP DE CADA ATRIBUTO:

        sql = f"update {tabela_grafos} set {atributo_comp_grafo}='{lista_comp_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_comp_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 8.2.5. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

    #ESSA ESTRUTURA DE REPETIÇÃO PARA CONFERIR OS COMPRIMENTOS DE ARCO E SE AS INFORMAÇÕES ADICIONADAS ANTERIORMENTE ESTÃO CONSISTENTES:

    list_len = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A QUANTIDADE DE RUAS PRESENTE NA LINHA DO SHP DOS GRAFOS É IGUAL A QUANTIDADE DE COMPRIMENTOS DE RUA

    list_comp = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A A SOMA DOS COMPRIMENTOS DE CADA RUA É IGUAL AO COMPRIMENTO TOTAL DA LINHA DO SHP DOS GRAFOS

    for id_i in range(1, id_max + 1): #ESTRUTURA DE REPETIÇÃO PARA PERCORRER DO ARCO 1 AO ARCO DE ID MAXIMO

        sql = f'select {atributo_dados}, {atributo_comp_grafo}, comp_via from "{tabela_grafos}" where id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS INFORMAÇÕES DE COMPRIMENTO DE VIA, NOME DE RUAS E COMPRIMENTO DE CADA NOME DE RUA DO SHP DOS GRAFOS.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        dados_nome = dados_consultados[0][0].split(';') #LISTA COM NOME DAS RUAS DO ARCO

        comp_dado = dados_consultados[0][1].split(';') #LISTA COM O COMPRIMENTO DE CADA RUA DA LISTA

        comp_total = round(dados_consultados[0][2], 4) #COMPRIMENTO TOTAL DO ARCO

        sum_comp = 0 #CRIANDO VARIAVEL PARA RECEBER A SOMA DE CADA COMPRIMENTO DE RUA

        for comp in comp_dado: #ADICIONANDO A VARIAVEL ANTERIOR A SOMA DOS COMPRIMENTOS DE RUA
            sum_comp = sum_comp + float(comp)

        sum_comp = round(sum_comp, 4) #ARRENDONDANDO PARA 4 CASAS DECIMAIS A SOMA DAS RUAS

        len_id = len(dados_nome) == len(comp_dado) #A QUANTIDADE DE RUAS É IGUAL A QUANTIDADE DO COMPRIMENTO DESSAS RUAS?

        list_len.append(len_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA

        comp_id = sum_comp == comp_total #A SOMA DO COMPRIMENTO DAS RUAS É IGUAL AO COMPRIMENTO DO ARCO?

        list_comp.append(comp_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA      

        print(f'===---===---===\nLinha de id {id_i}:\n\nTem a mesma quantidade de item em comp_ruas e nome_ruas?\n{len_id}\n\nComprimento total é igual ao somatorio de cada rua?\n{comp_id}\n\nSoma das ruas:{sum_comp}\nComprimento total do arco: {comp_total}\nDiferença: {comp_total - sum_comp}\n===---===---===\n') #MENSAGEM COM AS COMPARAÇÕES PARA CONFERIR A CONSISTENCIA ENTRE AS INFORMAÇÕES
        
#### 8.2.5.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 8.2.5.2. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

     list_comp.count(False)

#### 8.2.6. ATRIBUINDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais classificações de acordo com o Plano de Mobilidade de Viçosa presentes na tabela `vias_tipopm` estão contidas na linha com id em análise. Essas classificações (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'tipo_pm' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_tipopm' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vpm' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'tipo_pm' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    atributo_comp_grafo = 'comp_tipo_pm' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS COMPRIMENTOS DOS NOMES DAS RUAS NA TABELA DO SHP DOS GRAFOS

    #ADICIONANDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        sql = f'select sum(st_length({abr_dados}.geom)) from {tabela_dados} {abr_dados}, {tabela_grafos} {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados_grafo}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O COMPRIMENTO DE VIA DOATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_comp_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS DE COMPRIMENTOS SERÃO TRANSFORMADAS EM UMA STRING DA SEGUINTE FORMA: COMP_1; COMP_2; COMP_3; ... ; COMP_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_comp_str = lista_comp_str + str(dado) + '; '
            else:
                lista_comp_str = lista_comp_str + str(dado)

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DE COMP DE CADA ATRIBUTO:

        sql = f"update {tabela_grafos} set {atributo_comp_grafo}='{lista_comp_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_comp_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 8.2.7. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

    #ESSA ESTRUTURA DE REPETIÇÃO PARA CONFERIR OS COMPRIMENTOS DE ARCO E SE AS INFORMAÇÕES ADICIONADAS ANTERIORMENTE ESTÃO CONSISTENTES:

    list_len = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A QUANTIDADE DE RUAS PRESENTE NA LINHA DO SHP DOS GRAFOS É IGUAL A QUANTIDADE DE COMPRIMENTOS DE RUA

    list_comp = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A A SOMA DOS COMPRIMENTOS DE CADA RUA É IGUAL AO COMPRIMENTO TOTAL DA LINHA DO SHP DOS GRAFOS

    for id_i in range(1, id_max + 1): #ESTRUTURA DE REPETIÇÃO PARA PERCORRER DO ARCO 1 AO ARCO DE ID MAXIMO

        sql = f'select {atributo_dados}, {atributo_comp_grafo}, comp_via from "{tabela_grafos}" where id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS INFORMAÇÕES DE COMPRIMENTO DE VIA, NOME DE RUAS E COMPRIMENTO DE CADA NOME DE RUA DO SHP DOS GRAFOS.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        dados_nome = dados_consultados[0][0].split(';') #LISTA COM NOME DAS RUAS DO ARCO

        comp_dado = dados_consultados[0][1].split(';') #LISTA COM O COMPRIMENTO DE CADA RUA DA LISTA

        comp_total = round(dados_consultados[0][2], 4) #COMPRIMENTO TOTAL DO ARCO

        sum_comp = 0 #CRIANDO VARIAVEL PARA RECEBER A SOMA DE CADA COMPRIMENTO DE RUA

        for comp in comp_dado: #ADICIONANDO A VARIAVEL ANTERIOR A SOMA DOS COMPRIMENTOS DE RUA
            sum_comp = sum_comp + float(comp)

        sum_comp = round(sum_comp, 4) #ARRENDONDANDO PARA 4 CASAS DECIMAIS A SOMA DAS RUAS

        len_id = len(dados_nome) == len(comp_dado) #A QUANTIDADE DE RUAS É IGUAL A QUANTIDADE DO COMPRIMENTO DESSAS RUAS?

        list_len.append(len_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA

        comp_id = sum_comp == comp_total #A SOMA DO COMPRIMENTO DAS RUAS É IGUAL AO COMPRIMENTO DO ARCO?

        list_comp.append(comp_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA      

        print(f'===---===---===\nLinha de id {id_i}:\n\nTem a mesma quantidade de item em comp_tipo_pm e tipo_pm?\n{len_id}\n\nComprimento total é igual ao somatorio de cada rua?\n{comp_id}\n\nSoma das ruas:{sum_comp}\nComprimento total do arco: {comp_total}\nDiferença: {comp_total - sum_comp}\n===---===---===\n') #MENSAGEM COM AS COMPARAÇÕES PARA CONFERIR A CONSISTENCIA ENTRE AS INFORMAÇÕES

#### 8.2.7.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 8.2.7.2. QUANTIDADE DE FALTOS (SOMA DOS COMPRIMENTOS)

    list_comp.count(False)

#### 8.2.8. ATRIBUINDO O SENTIDO DA VIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará qual sentido da via presente na tabela `vias_oneway` estão contidas na linha com id em análise. Esse sentido de via (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

OBS.: Caso a linha possua mais de um sentido de via significa que existe um erro na geração da rede viária e isso deve ser consertado manualmente.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'oneway' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_oneway' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vow' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'oneway' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO SENTIDO DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 8.2.9. ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

Após a conclusão dos processos acima, a tabela `via_grafos` foi preenchida e a conexão com o banco de dados pode ser encerrada com os seguintes comandos:

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO
    con.close() #ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

**O arquivo `vias_grafos` preenchido se encontra na pasta `DADOS_FINAIS`.**

### 9. PREPARAÇÃO DA REDE COM ANÉL VIÁRIO

Para criar a rede com o anél viário proposto por Silva (2012) utilizou-se do mapa com seu traçado disponibilizado em sua tese. O mapa foi georreferenciado através da ferramenta `Georreferenciador` do software `QGIS` e serviu como pano de fundo para selecionar da base disponibilizada por Vieira e Castro (2021) aquelas vias que compunham o anel. A união do anel viário com a malha atual de Viçosa gerou uma malha viária fictícia com a simulação do Anel Viário. Os `passos 4 e 5 ` desse tutorial foram realizados para essa nova malha com o objetivo de garantir a integridade topológica da rede e transformá-la na forma de grafos. Com isso, gerou-se uma rede nomeada como `vias_grafos_anel` com os mesmo atributos da rede `via_grafos` mais um adicional: id_grafo_semanel, que possui o objetivo indicar qual o id correspondente da tabela `via_grafos` na tabela `via_grafos_anel`. Portanto, os atributos dessa nova rede são: **comp_via**, **nome_rua**, **comp_nome_rua**, **tipo_pm**, **comp_tipo_pm**, **sentido da via**, e **id_grafo_semanel**.

Esse dado foi importado para o banco de dados através da ferramenta `Gerenciador DB...`.

**O arquivo `vias_grafos_anel` se encontra na pasta `DADOS_INTERMEDIARIOS`.**

### 10. COMPLETANDO O ARQUIVO VIA_GRAFOS_ANEL

Para completar a rede `via_grafos_anel` realizou-se os mesmos processos da rede anterior. Algumas informações referentes ao anél viário foram preenchidas de forma manual através do software `QGIS` após a execução dos códigos, uma vez que o esse elemento da rede tem características específicas, tais como largura, classificação e pavimentação.

Os códigos utilizados são semelhantes.

#### 10.1. SCRIPTS EM PYTHON:

#### 10.1.1. IMPORTANDO O PACOTE psycopg2:

    import psycopg2 as pg

#### 10.1.2. CONECTANDO COM O BANCO DE DADOS:

Inseriu-se as configurações do banco de dados. Ele está sendo utilizado no servidor local (host = 'localhost'), o nome do banco de dados é dissertacao (database = 'dissertacao_artigo1'), o nome de usuário é postgres (user='postgres') e a senha do banco de dados é admin (password = 'admin').

    con = pg.connect(host='localhost', 
                    database='dissertacao_artigo1',
                    user='postgres', 
                    password='admin')

    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL

    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado DISSERTAÇÃO, que possui usuário postgres e senha admin.

#### 10.1.3. VENDO QUANTOS GRAFOS A MALHA POSSUI

É necessário descobrir quantas linhas a rede possui. Para isso executa-se os seguintes comandos:

    tabela_grafos = 'vias_grafos_anel' #TABELA COM OS GRAFOS
    sql = f'select max(id) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA
    
#### 10.1.4. ATRIBUINDO O ID DO GRAFO SEM ANEL NO SHP COM ANEL VIÁRIO

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos_anel = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA COM O ANEL
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos_a = 'vga' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA COM O ANEL
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA

    #CONSULTANDO OS DADOS DE CADA GRAFO DO MALHA COM ANEL VIARIO:

    for id_i in range(1, id_max + 1):

        sql = f"SELECT {abr_grafos}.id FROM {tabela_grafos} {abr_grafos}, {tabela_grafos_anel} {abr_grafos_a} WHERE ST_Contains({abr_grafos}.geom, {abr_grafos_a}.geom) and {abr_grafos_a}.id = {id_i}" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ID DA TABELA DOS DOS GRAFOS (QUE JÁ ESTÁ COMPLETA).

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        #INSERINDO AS INFORMAÇÕES CONSULTADAS:

        if dados_consultados == []: #CONFERINDO SE A CONSULTA É VAZIA. CASO SEJA, PULA A ITERAÇÃO.
            print(f'ID grafo anel: {id_i} -> Passa!\n')
            continue        

        else:
            sql = f"UPDATE {tabela_grafos_anel} SET id_grafo_semanel = '{dados_consultados[0][0]}' WHERE id = {id_i}" #COMANDO EM SQL A SER EXECUTADO. SERÁ ADICIONADO OS ATRIBUTOS CONSULTADOS NA MALHA COM ANEL VIARIO.

            cur.execute(sql) #EXECUTANDO O COMANDO

            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

            print(f'ID grafo anel: {id_i} -> ID grafo sem anel: {dados_consultados[0][0]}\n')



#### 10.1.5. ATRIBUINDO NOME DAS VIAS PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais nomes de ruas presentes na tabela `vias_nomeruas` estão contidas na linha com id em análise. Esses nomes (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'nome_rua' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_nomeruas' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vnr' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'nome_rua' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    atributo_comp_grafo = 'comp_nome_rua' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS COMPRIMENTOS DOS NOMES DAS RUAS NA TABELA DO SHP DOS GRAFOS

    #ADICIONANDO O NOME DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        if dados_consultados == []:
            continue

        else:

            lista_dados = []

            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])

            lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_dados_str = lista_dados_str + dado + '; '
                else:
                    lista_dados_str = lista_dados_str + dado

            sql = f'select sum(st_length({abr_dados}.geom)) from {tabela_dados} {abr_dados}, {tabela_grafos} {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados_grafo}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O COMPRIMENTO DE VIA DOATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

            cur.execute(sql) #EXECUTANDO O COMANDO

            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])

            lista_comp_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS DE COMPRIMENTOS SERÃO TRANSFORMADAS EM UMA STRING DA SEGUINTE FORMA: COMP_1; COMP_2; COMP_3; ... ; COMP_n

            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_comp_str = lista_comp_str + str(dado) + '; '
                else:
                    lista_comp_str = lista_comp_str + str(dado)

            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

            sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

            print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DE COMP DE CADA ATRIBUTO:

            sql = f"update {tabela_grafos} set {atributo_comp_grafo}='{lista_comp_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

            print(f'ID: {id_i} {atributo_comp_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.


#### 10.1.6. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

    #ESSA ESTRUTURA DE REPETIÇÃO PARA CONFERIR OS COMPRIMENTOS DE ARCO E SE AS INFORMAÇÕES ADICIONADAS ANTERIORMENTE ESTÃO CONSISTENTES:

    list_len = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A QUANTIDADE DE RUAS PRESENTE NA LINHA DO SHP DOS GRAFOS É IGUAL A QUANTIDADE DE COMPRIMENTOS DE RUA

    list_comp = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A A SOMA DOS COMPRIMENTOS DE CADA RUA É IGUAL AO COMPRIMENTO TOTAL DA LINHA DO SHP DOS GRAFOS

    for id_i in range(1, id_max + 1): #ESTRUTURA DE REPETIÇÃO PARA PERCORRER DO ARCO 1 AO ARCO DE ID MAXIMO

        sql = f'select {atributo_dados}, {atributo_comp_grafo}, comp_via from "{tabela_grafos}" where id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS INFORMAÇÕES DE COMPRIMENTO DE VIA, NOME DE RUAS E COMPRIMENTO DE CADA NOME DE RUA DO SHP DOS GRAFOS.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        dados_nome = dados_consultados[0][0].split(';') #LISTA COM NOME DAS RUAS DO ARCO

        comp_dado = dados_consultados[0][1].split(';') #LISTA COM O COMPRIMENTO DE CADA RUA DA LISTA

        comp_total = round(dados_consultados[0][2], 4) #COMPRIMENTO TOTAL DO ARCO

        sum_comp = 0 #CRIANDO VARIAVEL PARA RECEBER A SOMA DE CADA COMPRIMENTO DE RUA

        for comp in comp_dado: #ADICIONANDO A VARIAVEL ANTERIOR A SOMA DOS COMPRIMENTOS DE RUA
            sum_comp = sum_comp + float(comp)

        sum_comp = round(sum_comp, 4) #ARRENDONDANDO PARA 4 CASAS DECIMAIS A SOMA DAS RUAS

        len_id = len(dados_nome) == len(comp_dado) #A QUANTIDADE DE RUAS É IGUAL A QUANTIDADE DO COMPRIMENTO DESSAS RUAS?

        list_len.append(len_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA

        comp_id = sum_comp == comp_total #A SOMA DO COMPRIMENTO DAS RUAS É IGUAL AO COMPRIMENTO DO ARCO?

        list_comp.append(comp_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA      

        print(f'===---===---===\nLinha de id {id_i}:\n\nTem a mesma quantidade de item em comp_ruas e nome_ruas?\n{len_id}\n\nComprimento total é igual ao somatorio de cada rua?\n{comp_id}\n\nSoma das ruas:{sum_comp}\nComprimento total do arco: {comp_total}\nDiferença: {comp_total - sum_comp}\n===---===---===\n') #MENSAGEM COM AS COMPARAÇÕES PARA CONFERIR A CONSISTENCIA ENTRE AS INFORMAÇÕES
   
        
#### 10.1.6.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 10.1.6.2. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

     list_comp.count(False)

#### 10.1.7. ATRIBUINDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais classificações de acordo com o Plano de Mobilidade de Viçosa presentes na tabela `vias_tipopm` estão contidas na linha com id em análise. Essas classificações (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'tipo_pm' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_tipopm' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vpm' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'tipo_pm' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    atributo_comp_grafo = 'comp_tipo_pm' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS COMPRIMENTOS DOS NOMES DAS RUAS NA TABELA DO SHP DOS GRAFOS

    #ADICIONANDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        if dados_consultados == []:
            continue

        else:

            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])

            lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_dados_str = lista_dados_str + dado + '; '
                else:
                    lista_dados_str = lista_dados_str + dado

            sql = f'select sum(st_length({abr_dados}.geom)) from {tabela_dados} {abr_dados}, {tabela_grafos} {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados_grafo}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O COMPRIMENTO DE VIA DOATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

            cur.execute(sql) #EXECUTANDO O COMANDO

            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])

            lista_comp_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS DE COMPRIMENTOS SERÃO TRANSFORMADAS EM UMA STRING DA SEGUINTE FORMA: COMP_1; COMP_2; COMP_3; ... ; COMP_n

            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_comp_str = lista_comp_str + str(dado) + '; '
                else:
                    lista_comp_str = lista_comp_str + str(dado)

            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

            sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

            print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DE COMP DE CADA ATRIBUTO:

            sql = f"update {tabela_grafos} set {atributo_comp_grafo}='{lista_comp_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

            print(f'ID: {id_i} {atributo_comp_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 10.1.8. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

    #ESSA ESTRUTURA DE REPETIÇÃO PARA CONFERIR OS COMPRIMENTOS DE ARCO E SE AS INFORMAÇÕES ADICIONADAS ANTERIORMENTE ESTÃO CONSISTENTES:

    list_len = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A QUANTIDADE DE RUAS PRESENTE NA LINHA DO SHP DOS GRAFOS É IGUAL A QUANTIDADE DE COMPRIMENTOS DE RUA

    list_comp = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A A SOMA DOS COMPRIMENTOS DE CADA RUA É IGUAL AO COMPRIMENTO TOTAL DA LINHA DO SHP DOS GRAFOS

    for id_i in range(1, id_max + 1): #ESTRUTURA DE REPETIÇÃO PARA PERCORRER DO ARCO 1 AO ARCO DE ID MAXIMO

        sql = f'select {atributo_dados}, {atributo_comp_grafo}, comp_via from "{tabela_grafos}" where id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS INFORMAÇÕES DE COMPRIMENTO DE VIA, NOME DE RUAS E COMPRIMENTO DE CADA NOME DE RUA DO SHP DOS GRAFOS.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        dados_nome = dados_consultados[0][0].split(';') #LISTA COM NOME DAS RUAS DO ARCO

        comp_dado = dados_consultados[0][1].split(';') #LISTA COM O COMPRIMENTO DE CADA RUA DA LISTA

        comp_total = round(dados_consultados[0][2], 4) #COMPRIMENTO TOTAL DO ARCO

        sum_comp = 0 #CRIANDO VARIAVEL PARA RECEBER A SOMA DE CADA COMPRIMENTO DE RUA

        for comp in comp_dado: #ADICIONANDO A VARIAVEL ANTERIOR A SOMA DOS COMPRIMENTOS DE RUA
            sum_comp = sum_comp + float(comp)

        sum_comp = round(sum_comp, 4) #ARRENDONDANDO PARA 4 CASAS DECIMAIS A SOMA DAS RUAS

        len_id = len(dados_nome) == len(comp_dado) #A QUANTIDADE DE RUAS É IGUAL A QUANTIDADE DO COMPRIMENTO DESSAS RUAS?

        list_len.append(len_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA

        comp_id = sum_comp == comp_total #A SOMA DO COMPRIMENTO DAS RUAS É IGUAL AO COMPRIMENTO DO ARCO?

        list_comp.append(comp_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA      

        print(f'===---===---===\nLinha de id {id_i}:\n\nTem a mesma quantidade de item em comp_tipo_pm e tipo_pm?\n{len_id}\n\nComprimento total é igual ao somatorio de cada rua?\n{comp_id}\n\nSoma das ruas:{sum_comp}\nComprimento total do arco: {comp_total}\nDiferença: {comp_total - sum_comp}\n===---===---===\n') #MENSAGEM COM AS COMPARAÇÕES PARA CONFERIR A CONSISTENCIA ENTRE AS INFORMAÇÕES


#### 10.1.8.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 10.1.8.2. QUANTIDADE DE FALTOS (SOMA DOS COMPRIMENTOS)

    list_comp.count(False)

#### 10.1.9. ATRIBUINDO O SENTIDO DA VIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará qual sentido da via presente na tabela `vias_oneway` estão contidas na linha com id em análise. Esse sentido de via (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

OBS.: Caso a linha possua mais de um sentido de via significa que existe um erro na geração da rede viária e isso deve ser consertado manualmente.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos_anel' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'oneway' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_oneway' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vow' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'oneway' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA

    #ADICIONANDO SENTIDO DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA

    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX

        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.

        cur.execute(sql) #EXECUTANDO O COMANDO

        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS

        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS

        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])

        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n

        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado

        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:

        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO

        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.


#### 10.1.10. ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

Após a conclusão dos processos acima, a tabela `via_grafos` foi preenchida e a conexão com o banco de dados pode ser encerrada com os seguintes comandos:

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO
    con.close() #ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

**O arquivo `vias_grafos` preenchido se encontra na pasta `DADOS_FINAIS`.**

### 11. CÁLCULO DA CONECTIVIDADE E ACESSIBILIDADE (REDE SEM ANEL VIÁRIO)

Essa parte foi dividida em duas etapas: pré-processamento e cálculos. Foi utilizados comandos em linguaguem `SQL` e linguaguem `Python` nessas etapas. Nos título dos tópicos terá informações de os comandos foram executados no pgAdmin (comando SQL) ou no Jupyter Notebook (comando Python).

#### 11.1. PRÉ-PROCESSAMENTO

#### 11.1.1. CRIAÇÃO DA REDE A SER ANALISADA `(SQL)`

Uma cópia da tabela `via_grafos` foi criada e nomeada como `rede_vicosa` com o seguinte comando:

    -- CRIANDO UMA NOVA TABELA SIMILAR AO 'vias_grafos' CHAMADA 'rede_vicosa':
    SELECT * INTO rede_vicosa FROM vias_grafos ORDER BY id;
    
#### 11.1.2. CRIAÇÃO DE NOVAS COLUNAS PARA A TABELA `(SQL)`

Para resolver problemas de rede com a extensão `pgRouting` é necessário a criação das colunas de nó inicial, no final, custo e custo reverso. Utiliza-se o seguinte comando:

    -- CRIANDO OS CAMPOS PARA OS VÉRTICES DE FIM E INICIO DO GRAFO:
    ALTER TABLE rede_vicosa
    ADD source INT4,
    ADD target INT4,
    ADD cost REAL,
    ADD reverse_cost REAL;

#### 11.1.3. RENOMEANDO O CAMPO 'geom' PARA 'the_geom' `(SQL)`

    -- RENOMEANDO O CAMPO 'geom' PARA 'the_geom':
    ALTER TABLE rede_vicosa
    RENAME COLUMN geom TO the_geom;

#### 11.1.4. REDEFININDO OS DADOS DE DIREÇÃO DA VIA PARA 'YES' OU 'NO' `(SQL)`

    -- VIAS COM MÃO ÚNICA:
    UPDATE rede_vicosa SET oneway = 'YES' WHERE oneway = 'TF';

    -- VIAS COM MÃO DUPLA:
    UPDATE rede_vicosa SET oneway = 'NO' WHERE oneway = 'B';
    
 #### 11.1.5. DEFININDO OS CUSTOS DOS ARCOS `(SQL)`

    -- DEFININDO CUSTOS 1 PARA TODOS OS VÉRTICES:
    UPDATE rede_vicosa SET cost = 1, reverse_cost = 1;

    -- DEFININDO O CUSTO REVERSOS = -1 PARA OS GRAFOS COM DIREÇÃO ÚNICA:
    UPDATE rede_vicosa SET reverse_cost = '-1' WHERE oneway = 'YES';

#### 11.1.6. CRIANDO A TOPOLOGIA DA REDE `(SQL)`

    SELECT pgr_createTopology('rede_vicosa', 1);
    
As colunas source e target serão preenchidas com id's de vérticies iniciais e finais. Uma nova tabela surgirá com a geometria dos vértices gerados. 

#### 11.1.7. CRIANDO VÉRTICES NO MEIO DAS LINHAS `(SQL)`

Para o cálculo da acessibilidade é necessário computar o custo entre arcos, porém, a extensão só permite o cálculo entre nós. Para contornar esse problema, criou-se nós no meio de cada linha. O custo para atravessar esses nós centrais (com o custo de cada metade de arco igual a 0,5) é igual o custo entre arcos.

A tabela com pontos intermediários chama-se `mid_points` e para criá-la usou o seguinte comando:

    -- CRIANDO OS VÉRTICES NO MEIO DE CADA GRAFO:
    CREATE TABLE mid_points AS
    SELECT id, ST_LineInterpolatePoint(ST_LineMerge(the_geom), 0.5) as geom
    FROM rede_vicosa;

    ALTER TABLE mid_points
    ADD CONSTRAINT mid_points_pk PRIMARY KEY (id);

    CREATE INDEX sidx_mid_points
     ON mid_points
     USING GIST (geom);

#### 11.1.8. VERIFICAR SE AS COLUNAS SOURCE E TARGET ESTÃO CORRETAS `(SQL)`

Foi necessário verificar no `QGIS` se as colunas source e target estão preenchidas corretamente com os valores de id's dos vértices para as linhas com sentido de via unidirecional. Foi possível constatar que as linhas com id 1, 62, 71 e 96 estavam com o sentido invertido e isso foi consertado com os seguintes comandos:

    -- CONFERIR SE OS VÉRTICES SOURCE E TARGET ESTÃO DE ACORDO COM A REALIDADE PARA OS GRAFOS QUE POSSUEM APENAS UMA MÃO.
    -- DE FORMA MANUAL, DEVE-SE OBSERVAR AQUELES GRAFOS QUE NECESSIATARAM DE ALTERAR O CAMPO 'source' e 'target' DURANTE A EDIÇÃO DA CAMADA 'rede_vicosa'

    -- id do grafo: 1
    UPDATE rede_vicosa
    SET source = '41', target = '5'
    WHERE id = 1;

    -- id do grafo: 62
    UPDATE rede_vicosa
    SET source = '82', target = '49'
    WHERE id = 62;

    -- id do grafo: 71
    UPDATE rede_vicosa
    SET source = '85', target = '83'
    WHERE id = 71;

    -- id do grafo: 96
    UPDATE rede_vicosa
    SET source = '72', target = '61'
    WHERE id = 96;
    
  #### 11.1.9. CRIANDO A REDE COM PONTOS INTERMEDIÁRIOS ATRAVÉS DO QGIS
 
 Para criar a rede com pontos intermediários é necessário usar o software `QGIS` através da ferramenta `Connect nodes to lines` presente no complemento `Networks`. Criou-se uma cópia da camada `rede_vicosa` renomeando-a para `rede_vicosa_mp_prov` e a ferramenta foi utilizada nessa nova camada e o resultado foi importado para dentro do banco de dados utilizando o Gerenciador BD do `QGIS`. Durante a importação o id dessa rede foi nomeado como id0.
 
  #### 12.1.10. REMOVENTO ELEMENTOS INUTEIS DA `rede_vicosa_mp_prov` `(SQL)`
 
A criação dessa nova rede não resultou em um arquivo pronto para ser trabalhado e novos processamento foram realizados para "limpar" a tabela. A tabela limpa foi nomeada como `rede_vicosa_mp` e os comandos foram os seguintes:
 
    -- REMOVENDO DA CAMADA rede_vicosa_mp AQUELAS GEOMETRIAS NÃO UTEIS PARA A ATUAL ANÁLISE

    CREATE TABLE rede_vicosa_mp AS
    SELECT rvmpp.id, rvmpp.comp_via, rvmpp.nome_rua, rvmpp.tipo_pm, rvmpp.largura_me, rvmpp.pavimento, rvmpp.oneway, rvmpp.decliv_me, rvmpp.source, rvmpp.target, rvmpp.cost, rvmpp.reverse_co, ST_Difference(rvmpp.geom, ptos.geom) as geom 
    FROM (SELECT rvmpp.*
    FROM rede_vicosa_mp_prov rvmpp, mid_points mp
    WHERE ST_Contains(mp.geom, rvmpp.geom)) as ptos, rede_vicosa_mp_prov rvmpp
    WHERE ST_Intersects(rvmpp.geom, ptos.geom);

    DELETE FROM rede_vicosa_mp
    WHERE ST_LENGTH(geom) = 0;

    ALTER TABLE rede_vicosa_mp
    ADD COLUMN id0 SERIAL PRIMARY KEY;
    
  
