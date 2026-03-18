# Ánalise de Requisitos: Simulação com IA

## Metodologia de Setup (Meta-Prompting)
Para garantir que o assistente de IA atuasse estritamente como um Arquiteto de Dados Sênior e respeitasse as fases do projeto sem "alucinar" códigos futuros, utilizei técnicas de Meta-Prompting. O prompt de engenharia abaixo foi gerado com auxílio de IA para estabelecer as restrições arquiteturais (Local-to-Cloud, OLTP para OLAP) e o isolamento de fases:

    Prompt de Engenharia: Motor Antifraude Híbrido
    
    Atue como um Arquiteto de Dados Sênior.
    
    Contexto de Operação: Estou desenvolvendo em um ambiente nativo Ubuntu. O objetivo é construir um pipeline de dados ponta a ponta focado em detecção de fraudes financeiras, aplicando conceitos matemáticos de Teoria dos Grafos na modelagem de anéis de lavagem de dinheiro.
    
    Diretriz Arquitetural: Ignoraremos arquiteturas baseadas em mensageria (Kafka/Debezium) neste momento. O foco estrito é a arquitetura Híbrida (Local-to-Cloud) baseada em processamento batch/micro-batch.
    
    A Pilha Tecnológica Exigida:
    
    Local: Docker Compose, PostgreSQL (OLTP), Apache Spark (Processamento), Apache Airflow (Orquestração), Python (Simulação e Scripts).
    
    Cloud: AWS S3 (Data Lake), Amazon Redshift (Data Warehouse / OLAP).
    
    Protocolo de Interação (Regras Estritas):
    
    Isolamento de Fases: Você deve me guiar por 6 passos sequenciais. Nunca forneça o código do próximo passo antes que eu confirme a conclusão e o sucesso do passo atual.
    
    Densidade Técnica: Forneça os arquivos de configuração (.yaml), scripts (.py) e DDLs (.sql) completos e prontos para execução. Explique apenas a lógica arquitetural de alto nível.
    
    Depuração: Se eu reportar um erro, aja como um engenheiro de suporte nível 3. Peça os logs específicos antes de adivinhar a solução.
    
    Roadmap de Execução:
    
    Passo 1: docker-compose.yaml (Postgres, Spark Master/Worker, Airflow).
    
    Passo 2: Geração de Dados e DDL (Python + psycopg2) simulando transações normais e anéis de fraude ($A \rightarrow B \rightarrow C \rightarrow A$).
    
    Passo 3: Comandos AWS CLI para provisionamento de S3 e Redshift (Free Tier/Serverless).
    
    Passo 4: Script PySpark (Leitura JDBC $\rightarrow$ Detecção de Ciclos Direcionados $\rightarrow$ Escrita no S3 em Parquet).
    
    Passo 5: Script SQL para ingestão (Comando COPY do S3 para Redshift).

    Passo 6: Criação da DAG no Airflow para orquestrar os passos 2, 4 e 5.

    Confirme o entendimento destas regras com um "Ciente." e, em seguida, forneça imediatamente o código completo do Passo 1.

## 2. Transcrição da Entrevista

        **Diretor (Stakeholder):** "Olá! Muito prazer. Fui eu quem solicitou a sua alocação neste projeto. Nossa FinTech cresceu muito rápido e agora estamos sob forte pressão e auditoria dos órgãos reguladores. Nós temos uma suspeita forte de que criminosos estão usando nossa plataforma para lavar dinheiro. O padrão que a auditoria apontou é a triangulação: um fraudador usa uma 'Conta A' para mandar dinheiro para uma 'Conta B', que manda para a 'Conta C', e depois volta para a 'Conta A'. Isso cria um ciclo fechado para despistar a origem do dinheiro. Precisamos construir um pipeline de dados que consiga ler o nosso banco e identificar automaticamente esses ciclos para podermos bloquear essas contas. Eu conheço o problema de negócio, mas não sei como modelar isso em um banco de dados."
        
        **Engenheiro de Dados (Eu):** Como o sistema garante que o João da Silva que abriu a conta é realmente o João da Silva, e como vocês descobrem se ele é dono de outras empresas que também transacionam com a FinTech?
        
        **Diretor:** "Excelente pergunta! Você tocou num ponto que a nossa auditoria bateu muito forte, o nosso processo de KYC (Know Your Customer). Respondendo à sua primeira pergunta: no nosso sistema, todo Cliente que se cadastra é classificado por um `tipo_cliente` (Pessoa Física ou Pessoa Jurídica). O que garante a unicidade dele no nosso banco de dados é o documento. Se for PF, exigimos o CPF validado na Receita Federal. Se for PJ, exigimos o CNPJ. Então, no nosso banco, não podem existir dois clientes com o mesmo documento. 
        Sobre a sua segunda pergunta, que é a parte mais crítica da fraude: sim, o João da Silva pode ser dono de uma empresa! Nós temos um registro de Vínculo Societário. Ou seja, um Cliente Pessoa Física pode estar ligado a um Cliente Pessoa Jurídica como 'Sócio' ou 'Representante'. E adivinha? É exatamente aí que o anel de lavagem de dinheiro acontece muitas vezes. O fraudador abre uma Conta no nome dele (PF), outra Conta no nome da empresa dele (PJ), e uma terceira Conta no nome de um 'laranja'. O dinheiro fica girando entre essas contas para simular pagamentos legítimos de serviços que nunca existiram. 
        Ah, um detalhe importante para a sua modelagem: a transferência de dinheiro (a Transação) nunca acontece diretamente de 'Cliente para Cliente'. Ela sempre acontece de uma Conta de Origem para uma Conta de Destino. E um mesmo Cliente pode ter mais de uma Conta aberta com a gente. Toda transação precisa registrar o valor, a data/hora exata (timestamp) e gerar um código único de transação."
        
        **Engenheiro de Dados:** Doutor, quando o senhor quer rastrear o João no sistema, o senhor usa apenas o CPF dele ou o sistema gera um 'Número de Registro Interno' (como um número de contrato ou ficha) para ele? Pergunto isso porque, se o João mudar de CPF por algum motivo legal, precisamos saber como o histórico dele é mantido.
        
        **Diretor:** "Rapaz, que pergunta excelente! Nós usamos um Número de Registro Interno. No nosso banco de dados, o CPF ou CNPJ é tratado apenas como um dado obrigatório e único de cadastro. Mas, por baixo dos panos, o sistema gera um `ID_Cliente` numérico e sequencial para todo mundo que entra. É esse `ID_Cliente` que nós usamos para amarrar o João às Contas que ele possui. O mesmo princípio se aplica às contas e às movimentações: toda conta ganha um `ID_Conta` interno e toda transferência ganha um `ID_Transacao`."
        
        **Engenheiro de Dados:** Ok. Agora sobre o banco: vocês usam PostgreSQL ou outro relacional?
        
        **Diretor:** "Sim, a nossa equipe definiu o PostgreSQL como o nosso banco de dados transacional principal (OLTP). É dentro desse Postgres que todas as tabelas de Clientes, Contas e Transações residem. O nosso grande gargalo é que o PostgreSQL tradicional não foi feito para rodar algoritmos complexos de Teoria dos Grafos em tempo real. Por isso precisamos que você extraia esses dados, processe os grafos externamente em Spark e jogue o resultado mastigado para um Data Warehouse na nuvem."
        
        **Engenheiro de Dados:** Ciclos maiores que 3 também são suspeitos?
        
        **Diretor:** "Com toda a certeza! O ciclo de 3 passos ($A \rightarrow B \rightarrow C \rightarrow A$) é o mais comum e o mais 'preguiçoso'. Mas as quadrilhas mais sofisticadas criam teias muito maiores: o dinheiro passa por 4, 5 ou até 6 contas diferentes antes de voltar para a origem ($A \rightarrow B \rightarrow C \rightarrow D \rightarrow E \rightarrow A$). Para o nosso escopo inicial, a auditoria determinou que precisamos focar em ciclos de até 5 saltos."
        
        **Engenheiro de Dados:** Existe algum campo que identifica “conta suspeita” hoje (score, flag, etc.)?
        
        **Diretor:** "Sim. Na nossa tabela de Contas existe um campo chamado `status` ('Ativa', 'Bloqueada' ou 'Em Análise') e um campo `score_risco` (de 0 a 100). O grande problema é que esse nosso score atual é muito 'burro', ele só olha para a conta de forma individual. O objetivo do seu pipeline é evoluir isso, salvando no Data Warehouse os IDs de todas as contas que participaram dessa ciranda."
        
        **Engenheiro de Dados:** Precisamos bloquear automaticamente ou só gerar alerta + relatório para compliance?
        
        **Diretor:** "Só gerar alerta e relatório para a equipe de Compliance. Não vamos bloquear automaticamente para evitar falsos positivos que parem operações legítimas. O fluxo é: seu pipeline identifica o anel ($A \rightarrow B \rightarrow C \rightarrow A$) e salva no Redshift. Meu time pluga um BI lá, lê o alerta, analisa e aperta o botão de bloqueio internamente."



