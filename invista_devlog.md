
# Devlog - Invista

## 12-03-2024

**Rota**: https://fy4ijt03nj.execute-api.us-east-1.amazonaws.com/qa/webservice/webservice

Repositórios H  : cpf  | imóveis    | investigador | tribunais
Repositórios PH : cnpj | precatorio | jucesp       | pgfn

*[12:00 - 15:00]* Aprendi como vamos fazer para usar EventBridge para chamar uma lambda.
*[15:30 - 16:30]* Refatorei os repositórios cpf & imóveis.
*[16:30 - 17:00]* Refatorei os repositórios investigador.
*[17:00 - 17:30]* Refatorei os repositórios tribunais.

## 19-03-2024

*[12:00 - 16:00]* Ajustes realizados em tribunais para migração da AWS. Especificamente,
adequei o repositorio ao mypy e passei a adequá-lo também ao pylint.
*[16:00 - 20:00]* Realizei ajustes em imóveis para nova migração da AWS com alembic. 
Criei um script de migração do primeiro esquema de banco de dados e adaptei o segundo
script de migração. Também fiz adequações para conformidade com pylint e mypy. Empurrei
as mudanças para o GitHub e testei os endpoints. Os endpoints parecem funcionar, mas não 
consegui acessar a base de dados. Criei funções de logging para entender como o código recebe
nomes de pessoas pela url, para tentar acessar a base de dados por meio da url.


## 21-03-2024
*[13:30 - ]* Reunião do time de PPA para decidir direções do time.
Hoje foi falado sobre construir planilhas para melhorar a análise. 

Feature: Pipeline Inteligente de Processamento de Processo
    - Baixar os processos automaticamente dos tribunais
    - Enviar para o Leitor
Feature: ter a possibilidade de filtrar a lista por credores e enviar diretamente para a análise
Feature: ter a análise listada na lista de processos, ter um botão na lista de processos que envia para análise


