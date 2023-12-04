# chDA Contact Center 2023

### Contexto do desafio:

**The challenge presented here is for a Data Analyst position that I applied for a telecommunications company. In order to maintain the company's confidentiality, I have made some changes to the document I sent, removing logos and any possible identification.** 

This challenge consists of a presentation of a common problem encountered in contact center operations through a fictitious company, a database provided by the company and the questions that need to be answered using data analysis.


The company wants to implement a solution to analyze the number and duration of call segments:
 - By reason for contact
 - By service line through which the call came in	
 - Who handled the customer's call

Objectives:
- 1. write the SQL queries needed for the data ETL process (structuring information, creating dimensions, etc.)
- 2. devise and present the technical solution for the data model needed to address the company's needs
- 3. implementing the model in Power BI. Modeling can be done in star schema or snowflake.
- 4. evaluate the capacity of the implemented solution to support the accumulation of a large volume of historical data, and/or suggest solutions for this purpose.


### Technologies used

 - SQL Server Management Studio 2022
 - Excel
 - Power BI
 - Figma
 - ExcaliDraw
 - Draw io




## Development
___________________________________________

Step 1 - Understanding the tables

    Technologies used: Excel

Step 2 - Data intake and processing

    Technologies used: SQL Server Management Studio, Excel

Step 3 - Conceptual and Physical Modeling

    Technologies used: ExcaliDraw, Draw io

The model adopted was starschema. This model was chosen as a way of ensuring the least branching, guaranteeing a simplified and easy-to-understand structure, simplifying modeling and bringing better performance. 

The starschema model is generally the most suitable for scenarios where simplicity and performance are essential at the end of data consumption.

The model below was built from the databases delivered by the client.

**Original model**

![Modelo](https://github.com/vicsfran/chDA-Contact-Center-2023/blob/9f1e683badc8ea10bc580a1081fedc76a9bdbfb6/Assets/Modelo%20original%20database.jpg)

**Starschema model**

![Modelo1](https://github.com/vicsfran/chDA-Contact-Center-2023/blob/b167a485948d2dff7019aff639f58a5823112824/Assets/STARSCHEMA%20-%20Modelo%20L%C3%B3gico.png)


Step 4 - Building the adopted model

    Technologies used: SQL Server Management Studio

ETL construction

  Extraction and transformation queries *__starschema model__*

Ticket dimension: **tb_dim_ticket**

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

Dimension Call: **tb_dim_chamada**

    SELECT
            ID_Segmento_de_Chamada,
            Linha_de_Atendimento,
            Classificação_Nível_1_da_Linha_de_Atendimento,
            Classificação_Nível_2_da_Linha_de_Atendimento
    FROM tb_chamada

Fact: **tb_fact**

    WITH cte_chamada 
	    as (
		    SELECT CONCAT(ID_Chamada,ID_Utilizador) as sk, *
		    FROM tb_chamada
		    ),
    cte_ticket
	    as (
		    SELECT CONCAT(ID_Chamada,Utilizador)    as sk, *
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
            ELSE t.ID_Ticket END                 	 as IDTicket,
		    c.Tempo_Total_de_Atendimento	 as tempo_atendimento

    FROM cte_chamada     as c
    LEFT JOIN cte_ticket as t
    ON t.sk = c.sk

    ORDER BY c.data_segmento_chamada, c.hora_início_segmento_chamada


Queries for loading:

Dimension table:

- Create table TB_DIM_CHAMADA
    
        CREATE TABLE tb_dim_chamada (
            id_segmento_de_chamada 		    VARCHAR(255) PRIMARY KEY,
            id_chamada 				    VARCHAR(255),
            nm_linha_de_atendimento 		    VARCHAR(255),
            nm_clas_nivel_1_da_linha_de_atendimento VARCHAR(255),
            nm_clas_nivel_2_da_linha_de_atendimento VARCHAR(255)
	    );

 - Create table TB_DIM_TICKET

        CREATE TABLE tb_dim_ticket (
	    id_ticket 			VARCHAR(255) PRIMARY KEY,
	    id_interacao 		VARCHAR(255),
	    nm_motivo_do_ticket_nivel_1 VARCHAR(255),
	    nm_motivo_do_ticket_nivel_2 VARCHAR(255)
	    );
	

- Create table TB_FACT

        CREATE TABLE tb_fato (
            num_tempo_total_atendimento 	INT,
            id_segmento_chamada 		INT,
            id_ticket 				INT,
            id_utilizador 			INT,
            id_tempo 				INT,
            FOREIGN KEY (id_segmento_chamada) REFERENCES tb_dim_chamada(id_segmento_chamada),
            FOREIGN KEY (id_ticket) 	      REFERENCES tb_dim_ticket(id_ticket)
        );

Step 5 - Data visualization construction

    Technologies used:  SQL Server Management Studio, Power BI, Figma

[Contact Center Dashboard - Power BI](https://app.powerbi.com/view?r=eyJrIjoiZDM0ZTg0ZTYtMjg5MC00ZmJjLTlhZmUtZmQ1OGQ3NDgyNWFjIiwidCI6IjgyYTU4NjE2LTY4ZDYtNDA1MS05Y2E5LWIyY2U2YmE1MjEzNCJ9&pageName=ReportSectione7a643de1916c78ede94)

    DAX

    Metrics created using DAX language


 

Step 6 - Solving the business issues based on the solution built

    Technologies used: Power BI

The company wants to implement a solution to analyze the number and duration of call segments:
 - By reason for contact
 - By service line through which the call came in	
 - Who handled the customer's call


