# Smart Stock 📦

Sistema inteligente de gerenciamento de estoque com reposição automática baseado em análise de relatórios CSV.  A aplicação monitora níveis de estoque, identifica itens abaixo do limite de reposição e automaticamente envia solicitações de compra para fornecedores.

## 📋 Sobre o Projeto

Smart Stock é uma aplicação Spring Boot que automatiza o processo de controle de estoque e reposição de produtos.  O sistema lê relatórios de estoque em formato CSV, calcula automaticamente as quantidades necessárias para reposição (com margem de segurança de 20%) e se comunica com APIs externas do setor de compras para processar as solicitações. 

## ✨ Funcionalidades

- 📊 **Leitura de Relatórios CSV** - Processamento automático de arquivos de estoque
- 🔍 **Detecção Inteligente** - Identifica itens abaixo do threshold de reposição
- 🧮 **Cálculo Automático** - Calcula quantidade de reposição com 20% de margem
- 🔐 **Autenticação OAuth2** - Integração segura com API de compras
- 📡 **Integração HTTP** - Comunicação com setor de compras via Feign Client
- 💾 **Persistência MongoDB** - Armazena histórico de solicitações de compra
- ⚡ **Processamento Assíncrono** - Execução em background via CompletableFuture
- 📝 **Rastreamento Completo** - Registra status de sucesso/falha de cada compra
- 🎯 **Mock Server** - Ambiente local de desenvolvimento com Mockoon

## 🛠️ Tecnologias Utilizadas

- **Java 21** - Linguagem de programação
- **Spring Boot 3.3.3** - Framework principal
- **Spring Data MongoDB** - Persistência NoSQL
- **Spring Cloud OpenFeign** - Cliente HTTP declarativo
- **OpenCSV 5.9** - Leitura de arquivos CSV
- **MongoDB** - Banco de dados NoSQL
- **Maven** - Gerenciador de dependências
- **Docker** - Containerização do banco de dados
- **Bruno** - Cliente HTTP para testes de API (5.4%)
- **Mockoon** - Mock server para ambiente local

## 🗄️ Modelo de Dados

### PurchaseRequestEntity (MongoDB)

```
Collection: col_purchase_requests

{
  "_id": "uuid",
  "item_id": "UUID do item",
  "item_name": "Nome do produto",
  "quantity_on_stock": 5,
  "reorder_threshold": 10,
  "supplier_name": "Nome do fornecedor",
  "supplier_email": "email@fornecedor.com",
  "last_stock_update_time": "2024-09-08T14:30:00",
  "purchase_quantity": 12,
  "purchased_with_success": true,
  "purchase_date_time": "2024-09-10T15:45:00"
}
```

### Campos

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `item_id` | String | Identificador único do item (UUID) |
| `item_name` | String | Nome do produto |
| `quantity_on_stock` | Integer | Quantidade atual em estoque |
| `reorder_threshold` | Integer | Limite mínimo para reposição |
| `supplier_name` | String | Nome do fornecedor |
| `supplier_email` | String | Email do fornecedor |
| `last_stock_update_time` | LocalDateTime | Última atualização do estoque |
| `purchase_quantity` | Integer | Quantidade solicitada para compra |
| `purchased_with_success` | Boolean | Status da solicitação |
| `purchase_date_time` | LocalDateTime | Data/hora da solicitação |

## 📦 Estrutura do Projeto

```
smart-stock/
├── src/
│   └── main/
│       ├── java/
│       │   └── tech/
│       │       └── buildrun/
│       │           └── smartstock/
│       │               ├── client/
│       │               │   ├── dto/
│       │               │   ├── AuthClient.java
│       │               │   └── PurchaseSectorClient.java
│       │               ├── controller/
│       │               │   ├── dto/
│       │               │   └── StartController.java
│       │               ├── domain/
│       │               │   └── CsvStockItem.java
│       │               ├── entity/
│       │               │   └── PurchaseRequestEntity.java
│       │               ├── repository/
│       │               │   └── PurchaseRequestRepository.java
│       │               ├── service/
│       │               │   ├── AuthService.java
│       │               │   ├── PurchaseSectorService.java
│       │               │   ├── ReportService.java
│       │               │   └── SmartStockService.java
│       │               └── SmartstockApplication.java
│       └── resources/
│           └── application.properties
├── docker/
│   └── docker-compose.yml
├── local-env/
│   └── purchase-sector-apis.json
├── reports/
│   └── stock. csv
├── collection/
│   └── [Bruno API Collection]
├── pom.xml
└── README. md
```

## 🚀 Como Executar

### Pré-requisitos

- Java 21 ou superior
- Maven 3.6+
- Docker e Docker Compose
- Mockoon (para ambiente local)

### Passo 1: Clone o repositório

```bash
git clone https://github.com/matalvesdev/smart-stock.git
cd smart-stock
```

### Passo 2: Inicie o MongoDB com Docker

```bash
cd docker
docker-compose up -d
```

Isso irá criar um container MongoDB com as seguintes configurações:
- **Port**: 27017
- **Database**: smartstockdb
- **Username**: admin
- **Password**: 123

### Passo 3: Configure o Mock Server (Ambiente Local)

1. Instale o [Mockoon](https://mockoon.com/)
2. Importe o arquivo `local-env/purchase-sector-apis.json`
3. Inicie o mock server na porta 3000

O mock server simula duas APIs:
- `POST /api/token` - Autenticação OAuth2
- `POST /api/purchases` - Envio de solicitações de compra

### Passo 4: Configure as credenciais

Defina as variáveis de ambiente ou edite `application.properties`:

```bash
export APP_CLIENT_ID=ABC
export APP_CLIENT_SECRET=DEF
```

### Passo 5: Execute a aplicação

```bash
# Volte para o diretório raiz
cd ..

# Execute com Maven Wrapper
./mvnw spring-boot:run

# Ou compile e execute o JAR
./mvnw clean package
java -jar target/smartstock-0.0.1-SNAPSHOT. jar
```

A aplicação estará disponível em:  `http://localhost:8080`

## 📡 API Endpoints

### Iniciar Processamento de Estoque

```http
POST /start
Content-Type: application/json

{
  "reportPath": "/caminho/absoluto/para/reports/stock.csv"
}
```

**Resposta**:  `202 Accepted`

**Descrição**: Inicia o processamento assíncrono do relatório de estoque.

**Exemplo**:
```json
{
  "reportPath": "/home/user/smart-stock/reports/stock.csv"
}
```

## 📊 Formato do CSV

O arquivo CSV deve seguir o seguinte formato:

```csv
item_id,item_name,quantity,reorder_threshold,supplier_name,supplier_email,last_stock_update_time
1f3b5d6a-4bce-4b90-8c0c-dc71a9f81e65,Apple AirPods Pro,4,10,Apple Inc.,support@apple.com,2024-09-08T14:30:00
```

### Campos Obrigatórios

| Campo | Descrição | Exemplo |
|-------|-----------|---------|
| `item_id` | UUID do item | `1f3b5d6a-4bce-4b90-8c0c-dc71a9f81e65` |
| `item_name` | Nome do produto | `Apple AirPods Pro` |
| `quantity` | Quantidade em estoque | `4` |
| `reorder_threshold` | Limite para reposição | `10` |
| `supplier_name` | Nome do fornecedor | `Apple Inc.` |
| `supplier_email` | Email do fornecedor | `support@apple.com` |
| `last_stock_update_time` | Última atualização | `2024-09-08T14:30:00` |

## 🔄 Fluxo de Processamento

1. **Leitura do CSV**: Sistema lê o arquivo de relatório
2. **Análise de Estoque**: Para cada item, verifica se `quantity < reorder_threshold`
3. **Cálculo de Reposição**: `reorder_quantity = threshold + (threshold * 0.20)`
4. **Autenticação**: Obtém token OAuth2 da API de compras
5. **Envio de Solicitação**: Envia requisição de compra com token
6. **Persistência**: Salva registro no MongoDB com status de sucesso/falha

### Exemplo de Cálculo

```
Item: Apple AirPods Pro
Quantidade em estoque: 4
Threshold: 10
Status:  ABAIXO DO LIMITE

Cálculo:
reorder_quantity = 10 + (10 * 0.20)
reorder_quantity = 10 + 2
reorder_quantity = 12

Ação: Solicitar compra de 12 unidades
```

## ⚙️ Configuração

### application.properties

```properties
spring.application.name=smartstock

# OAuth2 Credentials
app.client-id=${APP_CLIENT_ID: ABC}
app.client-secret=${APP_CLIENT_SECRET:DEF}

# External APIs
api.auth-url=http://localhost:3000
api.purchase-sector-url=http://localhost:3000

# MongoDB Configuration
spring.data.mongodb.authentication-database=admin
spring.data.mongodb.auto-index-creation=true
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring. data.mongodb.database=smartstockdb
spring.data.mongodb.username=admin
spring.data.mongodb.password=123
```

### Docker Compose

```yaml
services:
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=123
```

## 🔑 Características Técnicas

### OpenFeign Client

A aplicação utiliza Spring Cloud OpenFeign para comunicação com APIs externas:

```java
@FeignClient(name = "PurchaseSectorClient", url = "${api.purchase-sector-url}")
public interface PurchaseSectorClient {
    @PostMapping(path = "/api/purchases")
    ResponseEntity<PurchaseResponse> sendPurchaseRequest(
        @RequestHeader("Authorization") String token,
        @RequestBody PurchaseRequest request
    );
}
```

### OpenCSV Binding

Leitura declarativa de CSV com anotações: 

```java
@CsvBindByName(column = "item_id")
private String itemId;

@CsvBindByName(column = "quantity")
private Integer quantity;
```

### Processamento Assíncrono

Execução em background para não bloquear a API:

```java
@PostMapping(path = "/start")
public ResponseEntity<Void> start(@RequestBody StartDto dto) {
    CompletableFuture.runAsync(() -> {
        smartStockService.start(dto.reportPath());
    });
    return ResponseEntity.accepted().build();
}
```

### MongoDB com Spring Data

Mapeamento de documentos com anotações:

```java
@Document(collection = "col_purchase_requests")
public class PurchaseRequestEntity {
    @MongoId
    @Field(name = "item_id")
    private String itemId;
    
    @Field(name = "purchased_with_success")
    private boolean purchasedWithSuccess;
}
```

## 📝 Exemplo de Uso Completo

### 1. Preparar o relatório CSV

Adicione o arquivo `stock.csv` na pasta `reports/` com itens abaixo do threshold. 

### 2. Iniciar o processamento

```bash
curl -X POST http://localhost:8080/start \
  -H "Content-Type: application/json" \
  -d '{
    "reportPath": "/absolute/path/to/reports/stock.csv"
  }'
```

### 3. Verificar no MongoDB

```javascript
// Conecte ao MongoDB
use smartstockdb

// Consulte as solicitações de compra
db.col_purchase_requests.find().pretty()
```

### 4. Resultado Esperado

```json
{
  "_id": "1f3b5d6a-4bce-4b90-8c0c-dc71a9f81e65",
  "item_name": "Apple AirPods Pro",
  "quantity_on_stock": 4,
  "reorder_threshold": 10,
  "supplier_name": "Apple Inc.",
  "supplier_email": "support@apple.com",
  "purchase_quantity": 12,
  "purchased_with_success": true,
  "purchase_date_time": "2024-09-10T15:45:32. 123"
}
```

## 🧪 Testes

```bash
./mvnw test
```

## 🎯 Collection Bruno

O projeto inclui uma coleção Bruno (5.4% do código) para testar os endpoints da API.  Importe a pasta `collection/` no Bruno para ter acesso a requisições pré-configuradas.

## 🔧 Troubleshooting

### Erro de autenticação
- Verifique se o Mockoon está rodando na porta 3000
- Confirme as credenciais `APP_CLIENT_ID=ABC` e `APP_CLIENT_SECRET=DEF`

### MongoDB Connection Error
- Certifique-se de que o container Docker está rodando
- Verifique as credenciais no `application.properties`

### Arquivo CSV não encontrado
- Use caminho absoluto no campo `reportPath`
- Verifique permissões de leitura do arquivo

## 👨‍💻 Autor

Desenvolvido por [matalvesdev](https://github.com/matalvesdev)

## 📄 Licença

Este projeto está sob a licença MIT.  Veja o arquivo LICENSE para mais detalhes.

---

📦 *"Estoque inteligente, negócios eficientes!"* 📦

⭐ Se este projeto foi útil para você, considere dar uma estrela! 
