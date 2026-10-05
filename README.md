# 📊 Desafio Power BI — Transformação de Dados (MySQL + Azure)

## 📑 Índice
- Contexto
- Objetivos
- Fontes
- Infraestrutura
- Transformações Realizadas
- Modelo de Dados
- Construção do Relatório
- Anomalias Encontradas
- Mesclar x Anexar
- Arquivos
- Autor

# Contexto:
- Este projeto é a entrega do desafio de Coleta e Processamento de Dados com Power BI, da trilha de Analista de Dados da [DIO](https://www.dio.me/). Os dados vêm de um banco MySQL hospedado no Azure, integrado ao Power BI Desktop, onde foram limpos, transformados e analisados em um relatório visual.

# Objetivos:
- Configurar o setup de banco de dados MySQL na Azure;
- Popular o servidor com os scripts fornecidos no curso (base de teste `Company`);
- Integrar o MySQL com o Power BI;
- Realizar as transformações indicadas (tipos, nulos, mesclas, colunas e agrupamentos);
- Criar um relatório para verificar as informações e possíveis anomalias.

# Fontes:
- Base de dados `Company` (`azure_company`): scripts de criação das tabelas e de inserção dos dados fornecidos no curso
- Conteúdo de referência: módulos do curso de Power BI Analyst da DIO

# Infraestrutura
- **Azure Database for MySQL (Servidor Flexível)**, criado na conta gratuita do Azure, com acesso público e regra de firewall para o IP de acesso;
- Conexão ao servidor pelo **Cloud Shell** e pelo **MySQL Workbench**, para criar o banco `azure_company`, as tabelas e inserir os dados;
- Conexão do **Power BI Desktop** ao banco pelo conector de MySQL.

**Tabelas (6):** `employee`, `departament`, `dept_locations`, `project`, `works_on` e `dependent`.

# Transformações Realizadas
Todas as etapas foram feitas no Power Query (Transformar dados). **Nenhuma query SQL foi usada nas junções:** as mesclas foram feitas pelo próprio Power BI.

**1. Cabeçalhos e tipos de dados**
- Valores monetários (`Salary`) e horas (`Hours`) em **Número Decimal**;
- Datas (`Bdate`) com tipo **Data**, exibidas em `dd/mm/aaaa`.

**2. Valores nulos e gerentes**
- O único nulo em `Super_ssn` é o do **James Borg**, o gerente geral, que não tem superior. Ele foi mantido, e o valor vazio foi substituído por **"Sem Gerente"**;
- **Nenhum departamento está sem gerente** (`Mgr_ssn` é obrigatório), então não houve lacunas para preencher.

**3. Substituição de valores e colunas complexas**
- `Address` foi dividida em **Número, Rua, Cidade e Estado**;
- No endereço do Ramesh Narayan, o nome da rua tinha um hífen (`Fire-Oak`), que quebraria a divisão. Foi substituído por espaço (`Fire Oak`) antes de separar.

**4. Mescla `employee` × `departament`**
- Base: `employee`; chaves: `Dno` = `Dnumber`; junção **Externa Esquerda**, para nenhum colaborador ficar de fora, mesmo sem departamento. Resultado: tabela `employee_department`, com o nome do departamento (`Dname`).

**5. Gerentes e nome completo**
- Autojunção de `employee` (`Super_ssn` = `Ssn`), também **Externa Esquerda**: uma junção interna eliminaria o James Borg, que não tem gerente;
- `Fname` e `Lname` mesclados em uma coluna `Name`; o mesmo para o nome do gerente (`Gerente`).

**6. Departamento + localização**
- Mescla de `departament` com `dept_locations` (`Dnumber`) e junção das colunas `Dname` e `Dlocation`, resultando na tabela `department_location`, com uma linha única por combinação (ex.: `Research - Houston`). Isso prepara o modelo estrela de um módulo futuro.

**7. Agrupamento e limpeza**
- **Agrupar por** gerente, contando colaboradores: tabela `employee_manager`;
- Remoção das colunas não usadas (`Minit`, colunas de navegação do conector, datas de gerência etc.), mantendo as colunas de chave;
- As consultas originais (`employee`, `dept_locations`) permanecem no Power Query como fonte, mas foram ocultadas do modelo.

# Modelo de Dados
Relações entre as tabelas (muitos para um, filtro em uma direção):
- `works_on[Essn]` → `employee_department[Ssn]`
- `works_on[Pno]` → `project[Pnumber]`
- `dependent[Essn]` → `employee_department[Ssn]`
- `project[Dnum]` → `department[Dnumber]`

A relação `department` → `employee_department` foi removida, porque criava dois caminhos de filtro entre as mesmas tabelas (ambiguidade). Como `employee_department` já traz `Dname`, ela não era necessária.

# Construção do Relatório

**Página 1 — Visão geral dos Colaboradores**
- Segmentador de departamento;
- Cartões: total de colaboradores (8), departamentos (3), projetos (6) e folha salarial (281.000);
- Barras: colaboradores por departamento e por gerente;
- Colunas: salário médio por departamento;
- Tabela: colaborador, departamento, gerente, salário e cidade;
- Botão de navegação para a Página 2.

**Página 2 — Projetos e anomalias**
- Barras: total de horas por projeto;
- Barras: horas por colaborador, com formatação condicional (destaque para quem tem menos de 40 horas);
- Colunas: dependentes por colaborador;
- Tabela: colaboradores sem gerente;
- Textos com as anomalias encontradas e botão de navegação de volta à Página 1.

# Anomalias Encontradas
- **James Borg (Headquarters)** não tem gerente e tem **0 hora** alocada, no projeto Reorganization;
- **Jennifer Wallace** soma **35 horas**; os demais colaboradores com horas alocadas somam 40;
- Nenhum departamento está sem gerente;
- O endereço do Ramesh Narayan tinha um hífen no nome da rua, tratado na transformação.

# Mesclar x Anexar
Neste caso foi usado **Mesclar**, e não **Anexar**. O **Anexar** empilha linhas de tabelas com a mesma estrutura de colunas. Aqui, as tabelas têm colunas diferentes e se relacionam por uma chave (`Dnumber`), e o objetivo é **acrescentar colunas** (o local a cada departamento, o nome do departamento a cada colaborador). Por isso a operação correta é a **mescla**, que une as tabelas lado a lado pela chave.

# Arquivos

| Arquivo | Descrição |
|---|---|
| `Power BI - Transformação de Dados.pbix` | Projeto completo do Power BI Desktop, com as 2 páginas e as transformações |
| `Power BI - Transformação de Dados.pdf` | Exportação em PDF das 2 páginas do relatório |
| `Power BI - Transformação de Dados.pptx` | Apresentação com as 2 páginas do relatório (um slide por página) |

# Autor
- Kelwin Paschoal
