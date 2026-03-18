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






