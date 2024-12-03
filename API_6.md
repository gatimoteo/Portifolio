<html>
<body>
  
  <h1 align="center"> API 6º Semestre - 02/2024</h1>
  <a href="https://github.com/quarks-team/Projeto-Integrador-SPCGrafeno"><img src="https://img.shields.io/badge/GitHub-Repositório Projeto-181717?style=for-the-badge&logo=github"></a>
  
  <h2> Parceiro Acadêmico: <a href="https://spcgrafeno.com.br/">SPC Grafeno</a></h2>
  
  <h2 style="font-family:roboto;"> Resumo do Projeto :clipboard:</h2>
  
  <p align="justify" style="font-family:roboto;"> A SPC Grafeno é uma empresa especializada em tecnologia financeira que desenvolve soluções para gestão de crédito e garantia de recebíveis. A empresa enfrenta o desafio de analisar grandes volumes de dados com o objetivo de criar produtos financeiros mais eficazes, como ferramentas de identificação de risco e previsão de tendências de mercado para duplicatas de crédito.</p>
   <p align="justify" style="font-family:roboto;">A solução envolveu o desenvolvimento de um sistema inteligente capaz de avaliar a confiabilidade dos endossantes por meio de algoritmos de IA, gerando um score baseado em seus históricos de duplicatas. No entanto, para garantir a eficácia dos algoritmos e modelos de IA, foi necessário realizar um processo robusto de extração, transformação e carga (ETL) dos dados provenientes de várias fontes.</p>
  
  <h2 style="font-family:roboto;"> Tecnologias Adotadas :computer:</h2>
   
  <ul>
  <li><a href="https://vuejs.org/">Vue.js</a>:
      <p align="justify" style="font-family:roboto;">Vue.js é um framework JavaScript de código-aberto, focado no desenvolvimento de interfaces de usuário e aplicativos de página única.</p></li>
    
  <li><a href="https://pt.wikipedia.org/wiki/HTML/">HTML</a>:
  <p align="justify" style="font-family:roboto;"> HTML é uma linguagem de marcação utilizada na construção de páginas na Web. Documentos HTML podem ser interpretados por navegadores. A tecnologia é fruto da junção entre os padrões HyTime e SGML. </p>
    </li>

  <li><a href="https://pt.wikipedia.org/wiki/Cascading_Style_Sheets/">CSS</a>:
  <p align="justify" style="font-family:roboto;"> Cascading Style Sheets (CSS) é um mecanismo para adicionar estilos (cores, fontes, espaçamento, etc.) a uma página web. </p>
    </li>
    
  <li><a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript">Javascript</a>:
  <p align="justify" style="font-family:roboto;"> JavaScript é uma linguagem de programação interpretada estruturada, de script em alto nível com tipagem dinâmica fraca e multiparadigma. </p>
    </li> 

  <li><a href="https://www.typescriptlang.org/">Typescript</a>:
  <p align="justify" style="font-family:roboto;"> TypeScript é uma linguagem de programação de código aberto desenvolvida pela Microsoft. É um superconjunto sintático estrito de JavaScript e adiciona tipagem estática opcional à linguagem. </p>
    </li>

  <li><a href="https://nestjs.com/">Nest.js</a>:
  <p align="justify" style="font-family:roboto;"> NestJS é um framework Node.js de código aberto destinado ao desenvolvimento de aplicativos do lado do servidor. </p>
    </li>

  <li><a href="https://www.python.org/">Python</a>:
  <p align="justify" style="font-family:roboto;"> Python é uma linguagem de programação de alto nível,[10] interpretada de script, imperativa, orientada a objetos, funcional, de tipagem dinâmica e forte. </p>
    </li>

  <li><a href="https://www.mongodb.com/">MongoDB</a>:
  <p align="justify" style="font-family:roboto;"> MongoDB é um software de banco de dados orientado a documentos livre, de código aberto e multiplataforma, escrito na linguagem C++. </p>
    </li>  

  </ul>
  
  <h2 style="font-family:roboto;"> Contribuições Individuais :dart:</h2>
  
  <h3> Atribuições como Desenvolvedor </h3>

<p>Neste projeto, fui responsável pela construção e otimização dos processos ETL essenciais para alimentar os modelos de IA com dados confiáveis e bem estruturados. A criação dos fluxos ETL envolveu várias etapas de extração, transformação e carga (ETL) dos dados provenientes de várias fontes, garantindo que informações como segmentação de mercado, localidade de pagamento e tipo de duplicata fossem tratadas adequadamente para os algoritmos de IA.</p>

<h3>Processos de ETL Implementados</h3>

<p>O primeiro passo foi garantir que as bases de dados estivessem corretamente integradas. Para isso, criei uma função chamada carregar_asset_parts, responsável por ler o arquivo CSV asset_parts e retornar um mapeamento dos identificadores de partes de ativos para seus respectivos nomes. O código a seguir faz essa leitura e valida as colunas necessárias:</p>

<details>
<pre><code>
def carregar_asset_parts(asset_parts_path):
    try:
        df_parts = pd.read_csv(asset_parts_path, low_memory=False, sep=";")
        if 'id' not in df_parts.columns or 'name' not in df_parts.columns:
            raise KeyError("As colunas 'id' e 'name' são obrigatórias em asset_parts.")
        return df_parts.set_index('id')['name'].to_dict()
    except Exception as e:
        logging.error(f"Erro ao carregar asset_parts: {e}")
        raise
</code></pre>
</details>

<p>Neste código, ao carregar o arquivo asset_parts, garantimos que as colunas necessárias ('id' e 'name') estivessem presentes, e então criamos um dicionário mapeando os IDs para os nomes das partes de ativos. Esse mapeamento foi crucial para associar os segmentos corretamente durante a transformação dos dados.</p>

<p>Após o carregamento inicial, a transformação dos dados seguiu com a criação de uma coluna de segmentação para as duplicatas. Com base no asset_parts_map (dicionário gerado no passo anterior), criei a função adicionar_segmento, que adiciona uma coluna segmento no DataFrame das duplicatas (df_bills), categorizando as duplicatas em segmentos como "Fundo", "Comércio" ou "Outros" de acordo com o nome do ativo. A lógica de segmentação foi a seguinte:</p>

<details>
<pre><code>
def adicionar_segmento(df_bills, asset_parts_map):
    def classificar_segmento(asset_part_id):
        name = asset_parts_map.get(asset_part_id, "").lower()
        if any(word in name for word in ["stg", "fidc", "investimento", "investimentos", "fgts", "finan", "invest", "credito"]):
            return "Fundo"
        elif "teste" in name:
            return "Outros"
        return "Comércio"
    
    df_bills['segmento'] = df_bills['endorser_original_id'].apply(classificar_segmento)
    return df_bills
</code></pre>
</details>

<p>Aqui, a função classificar_segmento busca palavras-chave nos nomes dos ativos para classificar cada duplicata em um segmento específico. Essa transformação foi essencial para enriquecer os dados e proporcionar uma melhor base para os algoritmos de IA.</p>

<p>Em seguida, adicionei variáveis no formato One-Hot Encoding para representar categorias como local de pagamento (payment_place) e tipo de duplicata (kind). As funções adicionar_colunas_payment_place, adicionar_colunas_segmento e adicionar_colunas_kind foram responsáveis por essas transformações. O código abaixo mostra a implementação de uma dessas funções para a coluna payment_place:</p>

<details>
<pre><code>
def adicionar_colunas_payment_place(df_bills):
    estados = ['SP', 'RJ', 'MG', 'RS']
    for estado in estados:
        coluna = f'payment_place_{estado.lower()}'
        df_bills[coluna] = df_bills['payment_place'].str.upper().fillna("").apply(lambda x: estado in x)
    return df_bills
</code></pre>
</details>

<p>Esse código cria colunas binárias para cada estado listado (SP, RJ, MG, RS), marcando com 1 se o estado aparecer na coluna payment_place e 0 caso contrário. Isso é útil para representar variáveis categóricas de forma que os modelos de IA possam processá-las corretamente.</p>

<p>Além das transformações de segmentação e categorização, foi necessário realizar manipulações de data e criar novas variáveis temporais. Utilizei as colunas de data para calcular a variável installment, que representa o número de meses entre a data de criação e a data de vencimento das duplicatas:</p>

<details>
<pre><code>
df_bills['installment'] = ((df_bills['due_date'] - df_bills['created_at']).dt.days / 30).fillna(0).astype(int)
</code></pre>
</details>

<p>Além disso, criei variáveis para o mês e o trimestre de vencimento das duplicatas com as linhas abaixo:</p>

<details>
<pre><code>
df_bills['month_due_date'] = df_bills['due_date'].dt.month.fillna(0).astype(int)
df_bills['quarter_due_date'] = df_bills['due_date'].dt.quarter.fillna(0).astype(int)
</code></pre>
</details>

<p>Essas variáveis temporais foram essenciais para fornecer informações adicionais aos algoritmos de IA, permitindo a análise de padrões ao longo do tempo.</p>

<p>Por fim, criei a coluna result, que classifica o estado da duplicata como "finalizada" ou "cancelada". Esse cálculo foi realizado da seguinte forma:</p>

<details>
<pre><code>
df_bills['result'] = df_bills['state'].apply(lambda x: 1 if x == 'finished' else 0 if x == 'canceled' else None).fillna(0).astype(int)
</code></pre>
</details>

<p>Após as transformações, utilizei a função load_data_to_db para inserir os dados no banco de dados PostgreSQL. O código abaixo ilustra a inserção de dados transformados no banco:</p>

<details>
<pre><code>
def load_data_to_db(df, connection):
    insert_query = """
    INSERT INTO ia_duplicate_prediction (
        id, installment, month_due_date, quarter_due_date,
        payment_place_sp, payment_place_rj, payment_place_mg, payment_place_rs,
        segment_financial, segment_commercial, segment_industrial, segment_educational,
        kind_receivable, kind_invoice, kind_check, result
    ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
    """
    
    for _, row in df.iterrows():
        values = (
            str(uuid.uuid4()),  # Gera ID único
            row['installment'], 
            row['month_due_date'], 
            row['quarter_due_date'],
            row['payment_place_sp'], 
            row['payment_place_rj'], 
            row['payment_place_mg'], 
            row['payment_place_rs'],
            row['segment_financial'], 
            row['segment_commercial'], 
            row['segment_industrial'], 
            row['segment_educational'],
            row['kind_receivable'], 
            row['kind_invoice'], 
            row['kind_check'], 
            row['result']
        )
        connection.execute_query(insert_query, values)
</code></pre>
</details>

<p>Essa função insere cada linha de dados transformados no banco de dados, garantindo que todas as informações necessárias para a análise de crédito e risco estejam disponíveis para os modelos de IA.</p>

<p>O processo ETL foi orquestrado na função run_etl_and_insert, que carrega os dados, aplica as transformações e insere os dados no banco de dados:</p>

<details>
<pre><code>
def run_etl_and_insert(asset_parts_path, asset_trade_bills_path):
    try:
        # Carrega e transforma os dados
        asset_parts_map = carregar_asset_parts(asset_parts_path)
        df_bills = pd.read_csv(asset_trade_bills_path, low_memory=False, sep=";")
        df_bills = adicionar_segmento(df_bills, asset_parts_map)
        df_bills = adicionar_colunas_payment_place(df_bills)
        df_bills = adicionar_colunas_segmento(df_bills)
        df_bills = adicionar_colunas_kind(df_bills)
        
        # Realiza outras transformações e insere no banco
        with PostgresConnection() as connection:
            load_data_to_db(df_bills, connection)
        
        logging.info("Processo ETL e inserção concluídos com sucesso.")
    except Exception as e:
        logging.error(f"Erro no processo ETL: {str(e)}")
        raise
</code></pre>
</details>

<p>Esse fluxo automatizado garantiu a execução contínua do processo ETL, permitindo que os dados fossem atualizados e inseridos no banco de forma eficiente, oferecendo dados limpos e prontos para serem usados pelos modelos de IA.</p>

<p>Através desse trabalho, fui capaz de garantir que os dados utilizados nos algoritmos de IA fossem limpos, completos e estruturados corretamente, o que foi fundamental para a construção de um sistema robusto e preciso na avaliação de risco de duplicatas de crédito.</p>

  <h2 style="font-family:roboto;"> Aprendizados Efetivos :book:</h2>   
<h3 align="center"> Hard Skills </h3>
  <table align="center">
    <tr>
      <th width="300px">Tecnologia/Metodologia</th>
      <th width="300px">Classificação</th>
    </tr>
    <tr>
      <td>Linguagens de Programação (Python)</td>
      <td>★★★★★★★☆☆☆</td>
    </tr>
    <tr>
      <td>Manipulação de Dados com Pandas</td>
      <td>★★★★★★☆☆☆☆</td>
    </tr>
    <tr>
      <td>Modelagem de Dados</td>
      <td>★★★★★★☆☆☆☆</td>
    </tr>
    <tr>
      <td>Tratamento de Erros e Exceções</td>
      <td>★★★★★☆☆☆☆☆</td>
    </tr>
     <tr>
      <td>Automação de Processos</td>
      <td>★★★★★★☆☆☆☆</td>
    </tr>
  </table>
  
  <h3 align="center">Soft Skills</h3>
   <table align="center">
    <tr>
      <th width="300px">Habilidade</th>
      <th width="300px">Descrição</th>
    </tr>
    <tr>
      <td>Resolução de Problemas</td>
      <td>Como desenvolvedor tive que identificar e resolver problemas durante o processo de transformação de dados, como garantir a presença de colunas essenciais nos arquivos CSV e lidar com erros durante a carga dos dados no banco de dados. A habilidade de diagnosticar e resolver erros de maneira eficiente é uma soft skill valiosa em qualquer área de tecnologia.</td>
    </tr>
    <tr>
      <td>Comunicação Clara e Documentação</td>
      <td>Tive que fazer o uso de logs de erros e sucessos (logging) ao longo do processo de ETL e isso reflete uma boa prática de documentação e comunicação do estado do sistema. Manter registros claros de processos, erros e sucessos é fundamental para trabalhar de forma colaborativa e garantir a rastreabilidade do trabalho.</td>
    </tr>
    <tr>
      <td>Gestão de Tempo e Priorização</td>
      <td>O projeto envolveu várias etapas, como a carga de dados, transformação e inserção no banco de dados. Tive que exercitar a habilidade de gerenciar essas etapas, garantindo que tudo fosse feito no tempo adequado para alimentar os algoritmos de IA, é um exemplo de boa gestão de tempo e priorização.</td>
    </tr>
    <tr>
      <td>Adaptação e Flexibilidade</td>
      <td>A adaptação às diferentes fontes de dados e a modificação das transformações conforme as necessidades do sistema de IA fez com que eu tivesse que ser flexível e me ajustar rapidamente às mudanças nos requisitos do projeto ou problemas imprevistos.</td>
    </tr>
  </table>
  
---

 <h2 align="center"> Navegação Projetos :link:</h2>
 
   <p align="justify" style="font-family:roboto;"><li><a href="https://github.com/gatimoteo/Portifolio/blob/main/API_1.md"> 1º Semestre: Assistente Virtual para Idosos - O app que facilita o uso de smartphones para usuários da terceira idade</a></li></p>
    <p align="justify" style="font-family:roboto;"><li><a href="https://github.com/gatimoteo/Portifolio/blob/main/API_2.md"> 2º Semestre: Trinity - Cadastro e análise de contas com mais simplicidade</a></li></p>
   <p align="justify" style="font-family:roboto;"><li><a href="https://github.com/gatimoteo/Portifolio/blob/main/API_4.md"> 4° Semestre: AgendHouse - Uma plataforma de gerenciamento de eventos</a></li></p>
   <p align="justify" style="font-family:roboto;"><li><a href="https://github.com/gatimoteo/Portifolio/blob/main/API_5.md"> 5° Semestre: Quarks - Dashboard Web de Análise de Faturas de Energia, Água e Gás</a></li></p>
   <p align="justify" style="font-family:roboto;"><li>6° Semestre: Quarks - Soluções de IA para o mercado de duplicatas de crédito</li></p>
   <p align="justify" style="font-family:roboto;"><li><a href="https://github.com/gatimoteo/Portifolio/blob/main/README.md"> Voltar para página inicial</a></li></p>

</body>
</html>
