[readme_md.md](https://github.com/user-attachments/files/34745814/readme_md.md)
# 🏥 Sistema de Gestão Hospitalar (`hospital_db`)

Projeto de modelagem relacional, carga de dados e extração de indicadores gerenciais (BI/Relatórios) desenvolvido em **MySQL 8.0**.

## 📁 Estrutura do Repositório

```
hospital-db/
├── 📂 database/                  # Dumps das tabelas e dados iniciais
│   ├── hospital_db_pacientes.sql
│   ├── hospital_db_medicos.sql
│   ├── hospital_db_quartos.sql
│   ├── hospital_db_internacoes.sql
│   ├── hospital_db_consultas.sql
│   └── hospital_db_prescricoes.sql
├── 📂 queries/                   # Scripts de relatórios e análises
│   └── relatorios.sql
└── 📄 README.md                  # Documentação do projeto

```

## 🗄️ Parte 1: Estrutura do Banco de Dados (`hospital_db`)

### 1.1 Arquitetura das Tabelas

O banco utiliza o motor **InnoDB** com codificação `utf8mb4` para garantir a integridade referencial entre os registros clínicos e financeiros:

| Tabela | Descrição | Chave Primária (PK) | Chaves Estrangeiras (FK) | 
| ----- | ----- | ----- | ----- | 
| **`pacientes`** | Dados cadastrais, CPF, contato e convênio médico. | `id_paciente` | — | 
| **`medicos`** | Corpo clínico, registro CRM e especialidade. | `id_medico` | — | 
| **`quartos`** | Controle de leitos (UTI, Enfermaria, Apartamento, Suíte VIP) e disponibilidade. | `id_quarto` | — | 
| **`internacoes`** | Histórico de admissões, altas e valor diário cobrado. | `id_internacao` | `id_paciente`, `id_quarto` | 
| **`consultas`** | Atendimentos ambulatoriais, status e faturamento. | `id_consulta` | `id_paciente`, `id_medico` | 
| **`prescricoes`** | Receituário médico, posologia e quantidade de itens. | `id_prescricao` | `id_consulta` | 

### 1.2 Como Importar no MySQL Workbench

1. Crie e selecione o banco de dados:

   ```
   CREATE DATABASE IF NOT EXISTS hospital_db CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
   USE hospital_db;
   
   ```

2. Execute os arquivos da pasta `/database` **nesta ordem** (respeitando as chaves estrangeiras):

   1. `hospital_db_pacientes.sql`

   2. `hospital_db_medicos.sql`

   3. `hospital_db_quartos.sql`

   4. `hospital_db_internacoes.sql`

   5. `hospital_db_consultas.sql`

   6. `hospital_db_prescricoes.sql`

## 📊 Parte 2: Consultas SQL e Resultados Esperados

### 1. Faturamento Total de Consultas por Convênio (`Saúde`)

**Objetivo:** Levantamento do valor total arrecadado em consultas para convênios que possuem `"Saúde"` no nome.

```
SELECT 
    pacientes.convenio,
    SUM(consultas.valor_consulta) AS faturamento_total
FROM consultas
JOIN pacientes ON consultas.id_paciente = pacientes.id_paciente
WHERE pacientes.convenio LIKE '%Saúde%'
GROUP BY pacientes.convenio;

```

**Resultado:**

| convenio | faturamento_total | 
| ----- | ----- | 
| Bradesco Saúde | `400.00` | 

### 2. Média do Valor de Consulta por Especialidade (`Cardiologia`)

**Objetivo:** Média do valor das consultas realizadas apenas nas especialidades ligadas à Cardiologia.

```
SELECT 
    medicos.especialidade,
    AVG(consultas.valor_consulta) AS media_valor_consulta
FROM consultas
JOIN medicos ON consultas.id_medico = medicos.id_medico
WHERE medicos.especialidade LIKE '%Cardiologia%'
GROUP BY medicos.especialidade;

```

**Resultado:**

| especialidade | media_valor_consulta | 
| ----- | ----- | 
| Cardiologia | `350.000000` | 
| Cardiologia Pediatrica | `400.000000` | 

### 3. Valor Médio das Diárias por Tipo de Quarto (`Apartamento` ou `UTI`)

**Objetivo:** Análise do valor médio da diária praticado nas internações em acomodações do tipo `"Apartamento"` ou `"UTI"`.

```
SELECT 
    quartos.tipo_quarto,
    AVG(internacoes.valor_diaria) AS media_valor_diaria
FROM internacoes
JOIN quartos ON internacoes.id_quarto = quartos.id_quarto
WHERE quartos.tipo_quarto IN ('Apartamento', 'UTI')
GROUP BY quartos.tipo_quarto;

```

**Resultado:**

| tipo_quarto | media_valor_diaria | 
| ----- | ----- | 
| UTI | `466.666667` | 
| Apartamento | `323.333333` | 

### 4. Total de Consultas Realizadas por Médico

**Objetivo:** Lista o nome dos médicos e a soma total dos valores de suas consultas com status `"Realizada"` ou `"Finalizada"`.

```
SELECT 
    medicos.nome_medico,
    SUM(consultas.valor_consulta) AS total_faturado
FROM consultas
JOIN medicos ON consultas.id_medico = medicos.id_medico
WHERE consultas.status_consulta IN ('Realizada', 'Finalizada')
GROUP BY medicos.id_medico, medicos.nome_medico;

```

**Resultado:**

| nome_medico | total_faturado | 
| ----- | ----- | 
| Dr. Marcos Andrade | `700.00` | 
| Dra. Patricia Lima | `280.00` | 
| Dr. Roberto Silva | `400.00` | 
| Dra. Camila Rocha | `1000.00` | 
| Dr. Gustavo Santos | `320.00` | 
| Dra. Vanessa Martins | `550.00` | 
| Dra. Renata Mello | `300.00` | 
| Dra. Claudia Castro | `600.00` | 

### 5. Soma de Diárias por Paciente com Sobrenome `"Silva"`

**Objetivo:** Soma acumulada do valor das diárias de internação para pacientes com sobrenome `"Silva"`.

```
SELECT 
    pacientes.nome_paciente,
    SUM(internacoes.valor_diaria) AS soma_valor_diarias
FROM internacoes
JOIN pacientes ON internacoes.id_paciente = pacientes.id_paciente
WHERE pacientes.nome_paciente LIKE '%Silva%'
GROUP BY pacientes.id_paciente, pacientes.nome_paciente;

```

**Resultado:**

| nome_paciente | soma_valor_diarias | 
| ----- | ----- | 
| Ana Silva | `450.00` | 
| João Pedro Silva | `200.00` | 

### 6. Contagem de Prescrições por Status da Consulta

**Objetivo:** Quantidade total de medicamentos prescritos agrupada pelo status da consulta, considerando `"Dipirona"` ou `"Paracetamol"`.

```
SELECT 
    consultas.status_consulta,
    SUM(prescricoes.quantidade) AS total_quantidade_medicamentos,
    COUNT(prescricoes.id_prescricao) AS total_prescricoes
FROM prescricoes
JOIN consultas ON prescricoes.id_consulta = consultas.id_consulta
WHERE prescricoes.medicamento LIKE '%Dipirona%' 
   OR prescricoes.medicamento LIKE '%Paracetamol%'
GROUP BY consultas.status_consulta;

```

**Resultado:**

| status_consulta | total_quantidade_medicamentos | total_prescricoes | 
| ----- | ----- | ----- | 
| Realizada | `35` | `2` | 

### 7. Média de Valor de Consultas por Convênio (`Unimed`) e Especialidade

**Objetivo:** Valor médio pago por consulta cruzando os convênios que contêm `"Unimed"` com a especialidade do médico.

```
SELECT 
    pacientes.convenio,
    medicos.especialidade,
    AVG(consultas.valor_consulta) AS media_valor_consulta
FROM consultas
JOIN pacientes ON consultas.id_paciente = pacientes.id_paciente
JOIN medicos ON consultas.id_medico = medicos.id_medico
WHERE pacientes.convenio LIKE '%Unimed%'
GROUP BY pacientes.convenio, medicos.especialidade;

```

**Resultado:**

| convenio | especialidade | media_valor_consulta | 
| ----- | ----- | ----- | 
| Unimed | Cardiologia | `350.000000` | 
| Unimed | Pediatria | `280.000000` | 
| Unimed | Cirurgia Geral | `500.000000` | 
| Unimed Pleno | Cirurgia Plastica | `600.000000` | 

### 8. Média de Diária de Internação por Convênio em Quartos de UTI

**Objetivo:** Valor médio gasto em diárias de internação agrupado por convênio, considerando apenas quartos classificados como `"UTI"`.

```
SELECT 
    pacientes.convenio,
    AVG(internacoes.valor_diaria) AS media_valor_diaria
FROM internacoes
JOIN pacientes ON internacoes.id_paciente = pacientes.id_paciente
JOIN quartos ON internacoes.id_quarto = quartos.id_quarto
WHERE quartos.tipo_quarto = 'UTI'
GROUP BY pacientes.convenio;

```

**Resultado:**

| convenio | media_valor_diaria | 
| ----- | ----- | 
| Unimed | `475.000000` | 
| Bradesco Saúde | `450.000000` | 

### 9. Medicamentos Prescritos por Médico em Consultas Realizadas

**Objetivo:** Quantidade total de itens prescritos por médico nas consultas marcadas como `"Realizada"`.

```
SELECT 
    medicos.nome_medico,
    SUM(prescricoes.quantidade) AS total_itens_prescritos
FROM prescricoes
JOIN consultas ON prescricoes.id_consulta = consultas.id_consulta
JOIN medicos ON consultas.id_medico = medicos.id_medico
WHERE consultas.status_consulta = 'Realizada'
GROUP BY medicos.id_medico, medicos.nome_medico;

```

**Resultado:**

| nome_medico | total_itens_prescritos | 
| ----- | ----- | 
| Dr. Marcos Andrade | `71` | 
| Dra. Patricia Lima | `30` | 
| Dr. Roberto Silva | `15` | 
| Dra. Camila Rocha | `20` | 
| Dr. Gustavo Santos | `12` | 
| Dra. Vanessa Martins | `5` | 
| Dra. Claudia Castro | `28` | 

### 10. Valor Total e Média de Consultas por Paciente (Inicial `"M"`)

**Objetivo:** Soma e média dos valores gastos em consultas por pacientes cujo nome começa com a letra `"M"`.

```
SELECT 
    pacientes.nome_paciente,
    SUM(consultas.valor_consulta) AS valor_total_consultas,
    AVG(consultas.valor_consulta) AS media_valor_consultas
FROM consultas
JOIN pacientes ON consultas.id_paciente = pacientes.id_paciente
WHERE pacientes.nome_paciente LIKE 'M%'
GROUP BY pacientes.id_paciente, pacientes.nome_paciente;

```

**Resultado:**

| nome_paciente | valor_total_consultas | media_valor_consultas | 
| ----- | ----- | ----- | 
| Maria Santos | `280.00` | `280.000000` | 
