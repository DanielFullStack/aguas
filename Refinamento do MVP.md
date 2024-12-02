### Desenvolvimento do MVP para Detecção de Vazamentos

O desenvolvimento será feito de forma incremental, começando com os serviços essenciais que integram os componentes descritos na arquitetura. Seguem as etapas detalhadas:

---

### **1. Configuração do Projeto**

1. **Criação do Projeto com Spring Boot**
   - Ferramenta: **Spring Initializr**.
   - Dependências principais:
     - Spring Web (para API REST).
     - Spring Data JPA (para integração com banco de dados).
     - PostgreSQL Driver.
     - Spring Kafka (para mensageria).
     - Spring Boot Actuator (para monitoramento).

2. **Estrutura de Diretórios**

```plaintext
src/main/java/com/aguasfluentes
├── config          # Configurações de Kafka, banco de dados, etc.
├── controller      # Endpoints REST.
├── dto             # Objetos de transferência de dados.
├── entity          # Entidades do banco de dados.
├── repository      # Interfaces para acesso ao banco de dados.
├── service         # Lógica de negócios (detecção de vazamentos, notificações).
├── utils           # Funções utilitárias.
└── listener        # Consumidores de eventos do Kafka.
```

---

### **2. Implementação do Serviço de Ingestão de Dados**

#### **Endpoint para Receber Leituras de Pressão**

- **Descrição**: Recebe dados de sensores e publica no tópico `pressure-readings` do Kafka.

```java
@RestController
@RequestMapping("/api/sensors")
public class SensorController {

    private final KafkaTemplate<String, PressureReading> kafkaTemplate;

    public SensorController(KafkaTemplate<String, PressureReading> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    @PostMapping("/pressure")
    public ResponseEntity<String> sendPressureReading(@RequestBody PressureReadingDto dto) {
        PressureReading reading = new PressureReading(dto.getSensorId(), dto.getPressure(), dto.getTimestamp());
        kafkaTemplate.send("pressure-readings", reading.getSensorId(), reading);
        return ResponseEntity.ok("Reading sent to Kafka successfully.");
    }
}
```

#### **Modelo de Dados**

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class PressureReading {
    private String sensorId;
    private double pressure;
    private long timestamp;
}
```

---

### **3. Detecção de Vazamentos**

#### **Consumidor do Tópico `pressure-readings`**

```java
@Component
public class PressureReadingConsumer {

    private final LeakDetectionService leakDetectionService;

    public PressureReadingConsumer(LeakDetectionService leakDetectionService) {
        this.leakDetectionService = leakDetectionService;
    }

    @KafkaListener(topics = "pressure-readings", groupId = "leak-detection-group")
    public void consume(PressureReading reading) {
        leakDetectionService.detectLeak(reading);
    }
}
```

#### **Serviço de Detecção de Vazamentos**

```java
@Service
public class LeakDetectionService {

    private final RedisTemplate<String, Double> redisTemplate;
    private final KafkaTemplate<String, LeakAlert> kafkaTemplate;

    public LeakDetectionService(RedisTemplate<String, Double> redisTemplate, KafkaTemplate<String, LeakAlert> kafkaTemplate) {
        this.redisTemplate = redisTemplate;
        this.kafkaTemplate = kafkaTemplate;
    }

    public void detectLeak(PressureReading reading) {
        String sensorKey = "sensor:" + reading.getSensorId();
        Double lastPressure = redisTemplate.opsForValue().get(sensorKey);

        if (lastPressure != null && Math.abs(reading.getPressure() - lastPressure) > 5) {
            LeakAlert alert = new LeakAlert(reading.getSensorId(), reading.getPressure(), lastPressure, reading.getTimestamp());
            kafkaTemplate.send("leakage-alerts", alert.getSensorId(), alert);
        }

        // Atualizar a pressão no cache
        redisTemplate.opsForValue().set(sensorKey, reading.getPressure());
    }
}
```

#### **Modelo de Alerta de Vazamento**

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class LeakAlert {
    private String sensorId;
    private double currentPressure;
    private double previousPressure;
    private long timestamp;
}
```

---

### **4. Serviço de Notificação**

#### **Consumidor do Tópico `leakage-alerts`**

```java
@Component
public class LeakAlertConsumer {

    private final NotificationService notificationService;

    public LeakAlertConsumer(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    @KafkaListener(topics = "leakage-alerts", groupId = "notification-group")
    public void consume(LeakAlert alert) {
        notificationService.sendAlert(alert);
    }
}
```

#### **Serviço de Notificação**

```java
@Service
public class NotificationService {

    public void sendAlert(LeakAlert alert) {
        String message = String.format(
            "Leak detected on sensor %s: Current pressure = %.2f, Previous pressure = %.2f.",
            alert.getSensorId(), alert.getCurrentPressure(), alert.getPreviousPressure()
        );

        // Enviar email (exemplo simplificado)
        System.out.println("Email Sent: " + message);

        // Enviar SMS (exemplo simplificado)
        System.out.println("SMS Sent: " + message);
    }
}
```

---

### **5. Configuração de Kafka**

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: default-group
      auto-offset-reset: earliest
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"
```

---

### **6. Banco de Dados PostgreSQL**

#### **Tabela para Persistir Leituras**

```sql
CREATE TABLE pressure_readings (
    id SERIAL PRIMARY KEY,
    sensor_id VARCHAR(50),
    pressure DOUBLE PRECISION,
    timestamp BIGINT
);

CREATE TABLE leak_alerts (
    id SERIAL PRIMARY KEY,
    sensor_id VARCHAR(50),
    current_pressure DOUBLE PRECISION,
    previous_pressure DOUBLE PRECISION,
    timestamp BIGINT
);
```

---

### **7. Testes Locais**

- Testar:
  1. API de ingestão (`/api/sensors/pressure`) com dados simulados.
  2. Detecção de vazamento verificando o comportamento com variações de pressão.
  3. Mensageria entre os tópicos `pressure-readings` e `leakage-alerts`.

---