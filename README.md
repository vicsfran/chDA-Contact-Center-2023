# chDA Contact Center 2023

### Contexto do desafio:

**O desafio exposto aqui é para uma vaga de Analista de Dados que concorri em Julho/2023 para uma empresa de telecomunicação. Para manter o sigilo da empresa, realizei algumas alterações no documento enviado, removendo os logos e qualquer indentificação possível.** 

Esse desafio é composto por uma apresentação de um problema comum encontrado em operações de contact center através de uma empresa ficticia, uma base de dados fornecida pela empresa e as questões que precisam ser respondidas utilizando a análise de dados.

A empresa pretende que seja implementada uma solução que permita analisar a quantidade e duração de segmentos de chamada:
 - Por motivo de contacto
 - Por linha de atendimento por onde a chamada entrou	
 - Por quem tratou da chamada do cliente

Objetivos:
- 1.	Escrever as queries de SQL necessárias para o processo de ETL dos dados (estruturação da informação, criação de dimensões, etc.)
- 2.	Idear e apresentar a solução técnica do modelo de dados necessário para endereçar a necessidade da empresa
- 3.	Implementar o modelo ideado em Power BI. A modelação poder ser feita em star schema ou em snowflake
- 4.	Avaliar a capacidade da solução implementada em suportar a acumulação de um grande volume de dados históricos, e/ou sugerir soluções para esse efeito.


### Tecnologias utilizadas

 - SQL Server Management Studio 2022
 - Excel
 - Power BI
 - Figma
 - ExcaliDraw
 - Draw io




## Desenvolvimento
___________________________________________

Passo 1 - Entendimento das tabelas

    Tecnologias utilizadas: Excel

Passo 2 - Ingestão e Tratamento dos dados

    Tecnologias utilizadas: SQL Server Management Studio, Excel

Passo 3 - Modelagem Conceitual e Física

    Tecnologias utilizadas: ExcaliDraw, Draw io

O modelo adotado foi o starschema. Esse modelo  foi escolhido como forma de garantir o menor tipo de ramificação garantindo uma estrutura simplificada e de fácil entendimento simplificando a modelagem e trazendo melhor desempenho. 

O modelo starschema geralmente é o mais adequado onde se preza por cenários que a simplicidade e o desempenho são essenciais na ponta final do consumo dos dados.

O modelo abaixo foi construído a partir das databases que foram entregues:

![Modelo](https://github.com/vicsfran/chDA-Contact-Center-2023/blob/986dd73584a92ccf4e19be380176bd57d68eae79/Assets/STARSCHEMA%20-%20Modelo%20L%C3%B3gico.png)

Cada ligação, pode gerar ou não um ticket de atendimento o que me da uma relação de 1 pra muitos.


Passo 4 - Construção do modelo adotado

    Tecnologias utilizadas: SQL Server Management Studio

Construção ETL

  Querys de extração e transformação *__modelo starschema__*

Dimensão Ticket: **tb_dim_ticket**

    SELECT  
        CONCAT(ID_Chamada,Utilizador) as SK,
        ID_Ticket,
        Data_Criação_Ticket,
        Hora_Criação_Ticket,
        Empresa_do_Utilizador,
        Equipa_do_Utilizador,
        Classificação_Nível_1_da_Equipa,
        Classificação_Nível_2_da_Equipa,
        Classificação_Nível_3_da_Equipa,
        Motivo_do_Ticket_Nível_1,
        REPLACE(Motivo_do_Ticket_Nível_2, 'CONSUMOS', 'Consumos') AS Motivo_do_Ticket_Nível_2 
    FROM tb_ticket
    ORDER BY data_criação_ticket, hora_criação_ticket

Dimensão Chamada: **tb_dim_chamada**

    SELECT
            ID_Segmento_de_Chamada,
            Linha_de_Atendimento,
            Classificação_Nível_1_da_Linha_de_Atendimento,
            Classificação_Nível_2_da_Linha_de_Atendimento
    FROM tb_chamada

Fato: **tb_fato**

    WITH cte_chamada 
	    as (
		    SELECT CONCAT(ID_Chamada,ID_Utilizador) as sk, *
		    FROM tb_chamada
		    ),
    cte_ticket
	    as (
		    SELECT CONCAT(ID_Chamada,Utilizador) as sk, *
		    FROM tb_ticket
		    )
		
    SELECT  
		    c.sk,
		    c.ID_Segmento_de_Chamada             as IDSegmento_da_chamada,
		    c."ID_Utilizador",
		    c.Data_Segmento_Chamada,
		    c.Hora_Início_Segmento_Chamada,
		    c.Hora_Fim_Segmento_Chamada,
		    CASE WHEN t.ID_Ticket IS NULL THEN NULL	   
            ELSE t.ID_Ticket END                 as IDTicket,
		    c.Tempo_Total_de_Atendimento		 as tempo_atendimento

    FROM cte_chamada     as c
    LEFT JOIN cte_ticket as t
    ON t.sk = c.sk

    ORDER BY c.data_segmento_chamada, c.hora_início_segmento_chamada


Querys para carga:

Tabela dimensão:

- Criar a tabela TB_DIM_CHAMADA
    
        CREATE TABLE tb_dim_chamada (
            id_segmento_de_chamada 				  VARCHAR(255) PRIMARY KEY,
            id_chamada 							  VARCHAR(255),
            nm_linha_de_atendimento 				  VARCHAR(255),
            nm_clas_nivel_1_da_linha_de_atendimento VARCHAR(255),
            nm_clas_nivel_2_da_linha_de_atendimento VARCHAR(255)
	    );

 - Criar a tabela TB_DIM_TICKET

        CREATE TABLE tb_dim_ticket (
	    id_ticket 					 VARCHAR(255) PRIMARY KEY,
	    id_interacao 			     VARCHAR(255),
	    nm_motivo_do_ticket_nivel_1 VARCHAR(255),
	    nm_motivo_do_ticket_nivel_2 VARCHAR(255)
	    );
	

- Criar a tabela TB_FATO

        CREATE TABLE tb_fato (
            num_tempo_total_atendimento 	INT,
            id_segmento_chamada 			INT,
            id_ticket 					INT,
            id_utilizador 				INT,
            id_tempo 						INT,
            FOREIGN KEY (id_segmento_chamada) REFERENCES tb_dim_chamada(id_segmento_chamada),
            FOREIGN KEY (id_ticket) 		    REFERENCES tb_dim_ticket(id_ticket)
        );

Passo 5 - Construção DataViz

    Tecnologias utilizadas:  SQL Server Management Studio, Power BI, Figma

Passo 6 - Resolução das questões de negócios a partir da solução construída

    Tecnologias utilizadas: Power BI


