# Bikcraft
# DOC INTEGRAÇÃO COM A API-LIGGA 

## Descrição
Integração com a API-LIGGA para consultas de disponibilidade de cobertura com fibra óptica


### Tecnologias
 * NodeJs

### Deendências
 * Libs
   * axios
   * express
   * msnodesqlv8
   * mssql
   * node-schedule
 * SO
   * Windows 11
 * IDE
   * Visual Studio Code



### Instalação
* Baixe o projeto .zip na pasta que desejar.
* descompacte o arquivo.
* abra o cmd dentro da pasta raiz e digite o comando abaixo.

   ```
   npm install package.json
   npm install
   ```

* Criação de tabelas obrigatórias na base SQL
```
   create table EnriqLiggaCopel (
      id_comCobertura int NOT NULL PRIMARY KEY IDENTITY,
      cep varchar(60),
      endereco varchar(120),
      numero varchar(100),
      cidade varchar(100),
      Phone varchar(100),
      nome varchar(150),
      planos nvarchar(max),
      updated datetime
   )
```
     create table SemCobLiggaCopel (
        id_semCobertura int NOT NULL PRIMARY KEY IDENTITY,
        cep varchar(60),
        endereco varchar(120),
        numero varchar(100),
        cidade varchar(100),
        Phone varchar(100),
        nome varchar(150),
        updated datetime
     )

```
   create table LiggaCopelHistorico (
     id int NOT NULL PRIMARY KEY IDENTITY,
     cep varchar(60),
     numero varchar(100),
     possui_cobertura varchar, 
     created_at datetime
   )
```
 

## Estrutura do projeto
* API
  *  getAvailability
      * Requisita a api através do endpoint https://lvt.apiliggatelecom.com.br/address/availability enviado uma requisição POST com o body.
      
        ```
            {
              "cep": cep,
              "number": numero casa
            }
        ```
        verifica a variavel hasAvailability, caso seja true retorna true, caso contrario retorna false.
  * getCityByCep
      * Resqusita a api atraves do endpoint https://lvt.apiliggatelecom.com.br/address/cep/{cep} enviando uma requisicao GET com um parametro obrigatorio {Cep) sem caracteres especiais. 
      exemplo: 
      
          ```
              https://lvt.apiliggatelecom.com.br/address/cep/81110112
          ```
          retorna a cidade ou um array vazio.
          
  * getPlans
      * Requisita a api atraves do endpoint 'https://lvt.apiliggatelecom.com.br/products/{tipoPessoa}/{cidade} enviado uma requisicao do tipo GET aceitando 2 parametros obrigatorios {tipoPessoa} e {Cidade}. 
      exemplo:
      
          ```
             https://lvt.apiliggatelecom.com.br/products/f/CURITIBA
          ```
          retorna um array com planos de internet ou retorna um array vazio
          
* Controller
   * CheckSituationController
        * checkSituationComCob func 
            * faz um Select na tabela EnricLiggaCopel pegando um dados a partir de uma chave composta de cep e numero e retorna se houve ou nao um dados encontraro atraves do length da pesquisa.
        * checkSituationSemCob func
            * faz um Select na tabela SemCobLiggaCopel pegando um dados a partir de uma chave composta de cep e numero e retorna se houve ou nao um dados encontraro atraves. do length da pesquisa
        * getIdIfSituationComCob func 
            * faz um Select na tabela EnricLiggaCopel pegando um dados a partir de uma chave composta de cep e numero e retorna o ID do registro.
        * getIdIfSituationSemCob func
            * faz um Select na tabela SemCobLiggaCopel pegando um dados a partir de uma chave composta de cep e numero e retorna o ID do registro.
            
   * ComCobController
        * insertIntoComCob func 
            * Insere dados na tabela EnriqLiggaCopel caso o registro pesquisado na api possua cobertura.
        * updateComCob func 
            * Atualiza dados na tabela EnriqLiggaCopel caso o registro pesquisado na api volte a ser consultado e ainda possua cobertura.
        * deleteFromComCob func 
            * Deleta dados na tabela EnriqLiggaCopel caso o registro pesquisado na api não possua mais cobertura.
       
   * LiggaHistoricoController
        * insertIntoComCob func 
            * Insere dados na tabela LiggaCopelHistorico para todo registro consultado na api mesmo tendo ou não tendo cobertura.
      
   * RelSemCobController
        * getSemCob func 
            * Faz um select de todos os dados da tabela RelSemCoberturaCopel.
            
   * SemCobController
        * insertIntoSemCob func 
            * Faz um insert na tabela SemCobLiggaCopel para todos os registros que foram pesquisado na api e não possuiam cobertura.
        * updateSemCob func 
            * Faz um update na tabela SemCobLiggaCopel para todos os registros que foram reconsultados na api e ainda não possuem cobertura.
        * deleteFromSemCob func 
            * Faz um delete na tabela SemCobLiggaCopel para todos os registros que foram reconsultados na api e agora possuem cobertura.
            
 * Database
   * dbConfig
        * Usado para cria a configuração do banco de dados especificamente para o MSSQL/SQL
        
            ```
              const dbConfig =  {
                  server: "seu server aqui",
                  port:1433,
                  user: 'seu user aqui',
                  password: 'sua senha aqui',
                  database: "sua database aqui",
                  options: {
                      enableArithAbort: true,
                  },
                  connectionTimeout:150000,
                  pool: {
                      max: 10,
                      min: 0,
                      idleTimeoutMillis:30000,
                  },
              };
            ```
          
          
          
      
 * Util
   * Sleep 
       * sleep func
           * recebe um parametro chamado ms onde podemos definir um valor em milisegundos e o mesmo criar um promisse onde usa a func setTimeout, passamos o parâmetro ms para a func setTimeout para termos o tempo que ela ira demorar para resolver a promisse.
           
           
 * Raiz 
   * api_ligga 
       * Possui uma cron para ficar executando o programa em loop infinito onde o mesmo só é ativado quando chega no horario determinado pelos parametros passados nas       variáveis como mostrado abaixo.
       
        
        ```
          rule.dayOfWeek = [new schedule.Range(0, 6)]; -- range dos dias que deve ficar ativo iniciando em 0 = Sunday e terminando em 6 = saturday
          rule.hour = 11; -- define que hrs deve executar
          rule.minute = 15; -- define os minutos em que deve começar
        
        ```
  * app  
    *  enriqApiLigga async func
        * Realiza uma consulta no banco de dados puxando uma array de contatos que precisam ser enriquecidos com a informação de possui ou não cobertura.
        * Cria um loop para trabalhar cada dado individualmente. 
        * Consulta na API e verifica se o registro possui cobertura ou não. 
        * Caso o registro possua cobertura ele vai entrar no primeiro if dentro do loop. 
            * Bloco de registros que possuem cobertura
                * verifica a cidade do usuario pelo cep informado.
                * traz os planos do usuario pela cidade obtida.
                * se o usuário ja possuia algum registro na base de não possui cobertura o mesmo é retirado de la e adicionado a base de possui cobertura.
                * se o registro ja estava na base de possui cobertura e é reconsultado novamente o mesmo sofre uma atualização.
        * Se o registro não possuir cobertura ele cai no else. 
            * Bloco de registros que possuem cobertura
                * verifica se o registro existe na tabela possui cobertura e se sim o mesmo é retirado e adicionado a tabela de não possui cobertura.
                * se o registro foi reconsultado e ainda não possui cobertura o mesmo é atualizado.
                * se o registro é novo ele é inserido na tabela não possui cobertura.
      







      
