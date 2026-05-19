#  RabbitMQ + Spring Boot

## PASSO 0: Instalar RabbitMQ via Docker


### Verificar instalação
```bash
docker --version
```

---

## PASSO 1: Rodar RabbitMQ em Docker

### Comando para iniciar RabbitMQ
```bash
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3.12-management
```

**O que significa cada parte:**
- `docker run` = executar um container
- `-d` = rodar em background (não travar o terminal)
- `--name rabbitmq` = dar um nome ao container
- `-p 5672:5672` = porta para comunicação com RabbitMQ (produtores/consumidores)
- `-p 15672:15672` = porta do painel web de administração
- `rabbitmq:3.12-management` = imagem oficial do RabbitMQ com painel

### Acessar o Painel do RabbitMQ
- URL: `http://localhost:15672`
- Usuário: `guest`
- Senha: `guest`

### Verificar se RabbitMQ está rodando
```bash
docker ps
```
Procure por uma linha com `rabbitmq` nela.

### Parar RabbitMQ (quando quiser desligar)
```bash
docker stop rabbitmq
```

### Reiniciar RabbitMQ (se parou)
```bash
docker start rabbitmq
```

---

## PASSO 2: Preparar o Ambiente do VS Code

### 2.1 Extensões Necessárias
No VS Code, vá em Extensions (Ctrl+Shift+X) e instale:

1. **Extension Pack for Java**
   - Autor: Microsoft
   - Inclui: Java suporte, Maven, debugger

2. **Spring Boot Extension Pack**
   - Autor: Pivotal Software
   - Inclui: Spring Boot tools, properties file editing

3. **REST Client** (opcional, mas útil)
   - Autor: Huachao Mao
   - Para testar APIs REST sem Postman

### 2.2 Verificar Java e Maven
```bash
java -version
mvn -version
```

Se não tiver, instale:
- **Java**: [openjdk.java.net](https://openjdk.java.net/)
- **Maven**: [maven.apache.org](https://maven.apache.org/download.cgi)

---

## PASSO 3: Criar Estrutura de Pastas

Abra um terminal e execute:

```bash
# Criar pasta principal
mkdir rabbitmq-demo
cd rabbitmq-demo

# Criar pastas dos projetos
mkdir rabbitmq-producer
mkdir rabbitmq-consumer

echo "Pastas criadas com sucesso!"
```

Agora você tem:
```
rabbitmq-demo/
├── rabbitmq-producer/
└── rabbitmq-consumer/
```

---

## PASSO 4: Criar Projeto PRODUTOR

### 4.1 Criar estrutura de pastas do Maven

No terminal, dentro de `rabbitmq-producer`:

```bash
cd rabbitmq-producer

# Criar estrutura Maven padrão
mkdir -p src/main/java/com/example/producer/config
mkdir -p src/main/java/com/example/producer/controller
mkdir -p src/main/java/com/example/producer/service
mkdir -p src/main/resources

echo "Estrutura do Produtor criada!"
```

### 4.2 Arquivo application.properties

Criar arquivo: `src/main/resources/application.properties`

```properties
# Configurações da aplicação
spring.application.name=rabbitmq-producer
server.port=8080

# Configurações RabbitMQ
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG
```

**O que é cada linha:**
- `server.port=8080` = aplicação roda em http://localhost:8080
- `spring.rabbitmq.host=localhost` = conecta ao RabbitMQ local
- `spring.rabbitmq.port=5672` = porta padrão do RabbitMQ
- `spring.rabbitmq.username=guest` = usuário padrão
- `spring.rabbitmq.password=guest` = senha padrão

---

## PASSO 5: Criar Arquivos do PRODUTOR

### 5.1 arquivo `pom.xml`

Criar arquivo: `pom.xml` (raiz do projeto)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>rabbitmq-producer</artifactId>
    <version>1.0.0</version>
    <name>RabbitMQ Producer</name>
    <description>API REST que envia mensagens para RabbitMQ</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Spring Web: para criar API REST -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring AMQP: para comunicar com RabbitMQ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**O que é pom.xml?**
- É o arquivo de configuração do Maven
- Define a versão do projeto
- Define as bibliotecas necessárias (dependências)
- Maven automaticamente baixa tudo

---

## PASSO 6: Criar Classes Java do PRODUTOR

### 6.1 Classe Principal: ProducerApplication.java

Criar arquivo: `src/main/java/com/example/producer/ProducerApplication.java`

```java
package com.example.producer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class, args);
    }
}
```

**O que faz:**
- `@SpringBootApplication` = ativa o Spring Magic
- `SpringApplication.run()` = inicia tudo
- Seu servidor web vai estar em `http://localhost:8080`

### 6.2 Configuração RabbitMQ: RabbitMQConfig.java

Criar arquivo: `src/main/java/com/example/producer/config/RabbitMQConfig.java`

```java
package com.example.producer.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    // Nomes que NUNCA devem mudar (produtor e consumidor usam os mesmos)
    public static final String EXCHANGE_NAME = "mensagens-exchange";
    public static final String QUEUE_NAME = "mensagens-fila";
    public static final String ROUTING_KEY = "mensagens.texto";

    // Criar a FILA (Queue)
    @Bean
    public Queue mensagensQueue() {
        return new Queue(QUEUE_NAME, true);
    }

    // Criar o EXCHANGE (roteador)
    @Bean
    public DirectExchange mensagensExchange() {
        return new DirectExchange(EXCHANGE_NAME, true, false);
    }

    // Conectar fila ao exchange
    @Bean
    public Binding binding(Queue mensagensQueue, DirectExchange mensagensExchange) {
        return BindingBuilder
                .bind(mensagensQueue)
                .to(mensagensExchange)
                .with(ROUTING_KEY);
    }
}
```

**Fluxo Visual:**
```
[Seu código envia "Olá"]
        ↓
[Vai para o EXCHANGE "mensagens-exchange"]
        ↓
[Exchange procura binding com routing_key "mensagens.texto"]
        ↓
[Encontra! Envia para QUEUE "mensagens-fila"]
        ↓
[Mensagem fica esperando no RabbitMQ]
```

### 6.3 Service: MensagemProducerService.java

Criar arquivo: `src/main/java/com/example/producer/service/MensagemProducerService.java`

```java
package com.example.producer.service;

import com.example.producer.config.RabbitMQConfig;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Service;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class MensagemProducerService {

    private static final Logger logger = LoggerFactory.getLogger(MensagemProducerService.class);
    private final RabbitTemplate rabbitTemplate;

    // Spring vai fornecer automaticamente um RabbitTemplate
    public MensagemProducerService(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void enviarMensagem(String mensagem) {
        try {
            logger.info("▶️  ENVIANDO: {}", mensagem);
            
            // Envia a mensagem para o RabbitMQ
            rabbitTemplate.convertAndSend(
                    RabbitMQConfig.EXCHANGE_NAME,
                    RabbitMQConfig.ROUTING_KEY,
                    mensagem
            );
            
            logger.info("✅ SUCESSO: Mensagem enfileirada!");
            
        } catch (Exception e) {
            logger.error("❌ ERRO: {}", e.getMessage());
            throw new RuntimeException("Falha ao enviar", e);
        }
    }
}
```

**O que faz cada parte:**
- `@Service` = diz ao Spring que isto é um serviço
- `RabbitTemplate` = ferramenta para enviar mensagens
- `enviarMensagem()` = envia a mensagem para o RabbitMQ

### 6.4 Controller: MensagemController.java

Criar arquivo: `src/main/java/com/example/producer/controller/MensagemController.java`

```java
package com.example.producer.controller;

import com.example.producer.service.MensagemProducerService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/mensagens")
@CrossOrigin(origins = "*")
public class MensagemController {

    private static final Logger logger = LoggerFactory.getLogger(MensagemController.class);
    private final MensagemProducerService producerService;

    public MensagemController(MensagemProducerService producerService) {
        this.producerService = producerService;
    }

    // Endpoint principal: enviar mensagem
    @PostMapping("/enviar")
    public ResponseEntity<Map<String, String>> enviarMensagem(@RequestBody String mensagem) {
        Map<String, String> response = new HashMap<>();

        try {
            // Validação
            if (mensagem == null || mensagem.trim().isEmpty()) {
                response.put("status", "erro");
                response.put("mensagem", "Mensagem não pode ser vazia!");
                return ResponseEntity.badRequest().body(response);
            }

            logger.info("📨 Recebida: {}", mensagem);
            
            // Enviar para RabbitMQ
            producerService.enviarMensagem(mensagem);

            // Resposta
            response.put("status", "sucesso");
            response.put("mensagem", "Enfileirada com sucesso!");
            response.put("conteudo", mensagem);
            
            return ResponseEntity.ok(response);

        } catch (Exception e) {
            response.put("status", "erro");
            response.put("mensagem", e.getMessage());
            return ResponseEntity.status(500).body(response);
        }
    }

    // Health check
    @GetMapping("/health")
    public ResponseEntity<Map<String, String>> health() {
        Map<String, String> response = new HashMap<>();
        response.put("status", "ok");
        response.put("servico", "Produtor RabbitMQ");
        return ResponseEntity.ok(response);
    }
}
```

**Endpoints criados:**
- `GET http://localhost:8080/api/mensagens/health` = verifica se está rodando
- `POST http://localhost:8080/api/mensagens/enviar` = envia mensagem

---

## PASSO 7: Criar Projeto CONSUMIDOR

### 7.1 Estrutura de pastas

No terminal, volte para `rabbitmq-demo` e entre em `rabbitmq-consumer`:

```bash
cd ../rabbitmq-consumer

# Criar estrutura
mkdir -p src/main/java/com/example/consumer/config
mkdir -p src/main/java/com/example/consumer/service
mkdir -p src/main/resources

echo "Estrutura do Consumidor criada!"
```

### 7.2 application.properties

Criar arquivo: `src/main/resources/application.properties`

```properties
# Configurações da aplicação
spring.application.name=rabbitmq-consumer

# Configurações RabbitMQ
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

# Logging
logging.level.root=INFO
logging.level.com.example=DEBUG
```

### 7.3 pom.xml

Criar arquivo: `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>rabbitmq-consumer</artifactId>
    <version>1.0.0</version>
    <name>RabbitMQ Consumer</name>
    <description>Consumidor de mensagens RabbitMQ</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Spring AMQP: para RabbitMQ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## PASSO 8: Criar Classes Java do CONSUMIDOR

### 8.1 Classe Principal: ConsumerApplication.java

Criar arquivo: `src/main/java/com/example/consumer/ConsumerApplication.java`

```java
package com.example.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

### 8.2 Configuração: RabbitMQConfig.java

Criar arquivo: `src/main/java/com/example/consumer/config/RabbitMQConfig.java`

```java
package com.example.consumer.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    // ⚠️ DEVE SER IDÊNTICO AO PRODUTOR!
    public static final String EXCHANGE_NAME = "mensagens-exchange";
    public static final String QUEUE_NAME = "mensagens-fila";
    public static final String ROUTING_KEY = "mensagens.texto";

    @Bean
    public Queue mensagensQueue() {
        return new Queue(QUEUE_NAME, true);
    }

    @Bean
    public DirectExchange mensagensExchange() {
        return new DirectExchange(EXCHANGE_NAME, true, false);
    }

    @Bean
    public Binding binding(Queue mensagensQueue, DirectExchange mensagensExchange) {
        return BindingBuilder
                .bind(mensagensQueue)
                .to(mensagensExchange)
                .with(ROUTING_KEY);
    }
}
```

### 8.3 Service: MensagemConsumerService.java

Criar arquivo: `src/main/java/com/example/consumer/service/MensagemConsumerService.java`

```java
package com.example.consumer.service;

import com.example.consumer.config.RabbitMQConfig;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Service
public class MensagemConsumerService {

    private static final Logger logger = LoggerFactory.getLogger(MensagemConsumerService.class);

    /**
     * ⭐ ESTE É O MÉTODO MÁGICO!
     * 
     * @RabbitListener(queues = QUEUE_NAME):
     * "Spring, escute a fila 'mensagens-fila'"
     * "Quando uma mensagem chegar, chame este método"
     * 
     * O Spring automaticamente:
     * 1. Conecta ao RabbitMQ
     * 2. Fica ouvindo a fila
     * 3. Quando mensagem chega, chama este método
     * 4. Remove a mensagem da fila
     * 5. Volta a ouvir
     */
    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME)
    public void consumirMensagem(String mensagem) {
        try {
            String dataHora = LocalDateTime.now()
                    .format(DateTimeFormatter.ofPattern("HH:mm:ss.SSS"));
            
            // Exibir no console
            logger.info("");
            logger.info("╔═══════════════════════════════════════════════════════╗");
            logger.info("║ 📬 MENSAGEM RECEBIDA DO RABBITMQ!                     ║");
            logger.info("╠═══════════════════════════════════════════════════════╣");
            logger.info("║ ⏰ Hora: {}                                            ║", dataHora);
            logger.info("║ 📝 Texto: \"{}\"", mensagem);
            logger.info("║ ✅ Status: Processada com sucesso!                    ║");
            logger.info("╚═══════════════════════════════════════════════════════╝");
            logger.info("");
            
        } catch (Exception e) {
            logger.error("❌ ERRO ao processar: {}", e.getMessage());
        }
    }
}
```

---

## PASSO 9: Executar os Projetos

### 9.1 Terminal 1: Rodar o PRODUTOR

```bash
cd rabbitmq-demo/rabbitmq-producer
mvn spring-boot:run
```

Quando ver:
```
Tomcat started on port(s): 8080
```

✅ Produtor está rodando em `http://localhost:8080`

### 9.2 Terminal 2: Rodar o CONSUMIDOR

```bash
cd rabbitmq-demo/rabbitmq-consumer
mvn spring-boot:run
```

Quando ver:
```
Started ConsumerApplication in X seconds
```

✅ Consumidor está rodando e AGUARDANDO MENSAGENS

---

## PASSO 10: Testar com Postman

### 10.1 Instalar Postman
Baixar em: [postman.com](https://www.postman.com/downloads/)

### 10.2 Fazer requisição

1. Abrir Postman
2. Clique em "+" para nova requisição
3. Método: **POST**
4. URL: `http://localhost:8080/api/mensagens/enviar`
5. Aba "Body" → selecione "raw" → texto puro (não JSON)
6. Digite: `Olá RabbitMQ!`
7. Clique **Send**

### 10.3 Ver a mensagem no Consumidor

No terminal do Consumidor, você verá:

```
╔═══════════════════════════════════════════════════════╗
║ 📬 MENSAGEM RECEBIDA DO RABBITMQ!                     ║
╠═══════════════════════════════════════════════════════╣
║ ⏰ Hora: 10:30:45.123                                 ║
║ 📝 Texto: "Olá RabbitMQ!"
║ ✅ Status: Processada com sucesso!                    ║
╚═══════════════════════════════════════════════════════╝
```

### 🎉 FUNCIONOU!

---

## RESUMO DO QUE APRENDEMOS

### Fluxo Completo:
```
1. Você faz POST em http://localhost:8080/api/mensagens/enviar
                                    ↓
2. Controller recebe a mensagem
                                    ↓
3. Service envia para RabbitMQ
                                    ↓
4. RabbitMQ envia para EXCHANGE
                                    ↓
5. EXCHANGE envia para QUEUE
                                    ↓
6. Consumidor recebe da QUEUE
                                    ↓
7. Exibe no console
```

### Conceitos:
- **Queue (Fila)**: armazena mensagens
- **Exchange**: roteia mensagens
- **Binding**: conecta Queue ao Exchange
- **Produtor**: envia mensagens
- **Consumidor**: recebe mensagens

### Tecnologias:
- **Docker**: rodando RabbitMQ
- **Spring Boot**: framework Java
- **Spring AMQP**: comunicação com RabbitMQ
- **Maven**: gerenciador de dependências
- **Postman**: teste de APIs

---

## 🐛 Problemas Comuns

### "Connection refused"
RabbitMQ não está rodando!
```bash
docker start rabbitmq
```

### "Could not find org.springframework.boot"
Maven não baixou dependências. Execute:
```bash
mvn clean install
```

### Porta 5672 já está em uso
Outro RabbitMQ está rodando:
```bash
docker ps  # veja containers
docker stop nome-do-container
```

### "Cannot create AMQP connection"
RabbitMQ credenciais erradas. Verifique `application.properties`

---

