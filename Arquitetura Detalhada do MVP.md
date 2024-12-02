### **Arquitetura Detalhada do MVP**

---

#### **Objetivo do MVP**
Incorporar os dispositivos Raspberry Pi como pontos remotos de coleta de dados, enviando informações via Webhook para o backend hospedado na nuvem. O backend processa os dados e os envia para o Apache Kafka, mantendo a robustez e escalabilidade da arquitetura.

---

### **1. Componentes Principais**

1. **Dispositivos de Coleta**
   - **Raspberry Pi** conectados aos sensores de pressão.
   - Os dispositivos enviam dados para o backend via **Webhooks HTTP REST**.
   - Comunicação segura via autenticação e criptografia TLS.

2. **Backend na Nuvem**
   - API REST (em **Spring Boot**) para receber dados dos dispositivos.
   - Dados são validados e publicados no Kafka.
   - Registro de leituras no banco de dados relacional (PostgreSQL).

3. **Mensageria**
   - **Apache Kafka** para gestão de eventos:
     - Tópico `pressure-readings`: Leituras recebidas dos sensores.
     - Tópico `leakage-alerts`: Alertas gerados após detecção de vazamentos.

4. **Detecção de Vazamentos**
   - Serviço dedicado para:
     - Consumir eventos do tópico `pressure-readings`.
     - Realizar comparações de leituras com base no cache (Redis).
     - Identificar vazamentos e gerar eventos de alerta.

5. **Notificação**
   - Serviço de notificação consome eventos do tópico `leakage-alerts` e notifica os responsáveis.
   - Canais:
     - **E-mail** (Java Mail).

6. **Banco de Dados**
   - **PostgreSQL**:
     - Persistência de leituras, alertas e histórico de notificações.
   - **Redis**:
     - Armazenamento temporário para comparação rápida de leituras.

---

### **2. Fluxo de Dados**

1. **Leitura de Sensores (Raspberry Pi)**
   - O Raspberry Pi lê os dados dos sensores conectados e os envia para o backend via **HTTP POST**.

2. **Ingestão e Validação no Backend**
   - O backend valida o payload e publica os dados no tópico Kafka `pressure-readings`.
   - Leituras também são registradas no PostgreSQL para auditoria.

3. **Processamento de Vazamentos**
   - Serviço de detecção consome os dados do tópico Kafka.
   - Recupera a última leitura armazenada no Redis para comparação.
   - Identifica vazamentos (variação maior que 5 mca) e publica alertas no tópico `leakage-alerts`.

4. **Notificação**
   - Serviço de notificação consome eventos do tópico de alertas e envia notificações via e-mail.

---

### **3. Diagrama de Arquitetura**

```plaintext
[Raspberry Pi com Sensores]
       |
       |--> [API REST - Backend na Nuvem]
                    |
                    |--> [Apache Kafka]
                              |
        ---------------------------------------------
        |                                           |
[Detecção de Vazamentos]                 [Serviço de Notificação]
        |                                           
   [PostgreSQL]
        |
     [Redis]
```

---

### **4. Tecnologias Atualizadas**
|------------------------------|-------------------------------|----------------------------------------------|
| **Componente**               | **Tecnologia**                | **Justificativa**                            |
|------------------------------|-------------------------------|----------------------------------------------|
| **Dispositivo IoT**          | Raspberry Pi                  | Coleta de dados em campo de forma flexível.  |
| **Comunicação IoT->Backend** | HTTP REST (Webhooks)          | Leve e compatível com a maioria dos sistemas.|
| **Backend**                  | Java + Spring Boot            | Framework robusto para APIs e processamento. |
| **Mensageria**               | Apache Kafka                  | Alta performance e escalabilidade.           |
| **Banco de Dados Relacional**| PostgreSQL                    | Consistência e suporte a consultas SQL.      |
| **Banco de Dados em Cache**  | Redis                         | Processamento rápido para leituras recentes. |
| **Notificação por E-mail**   | Java Mail                     | Serviços confiáveis para envio de e-mails.   |
|------------------------------|-------------------------------|----------------------------------------------|
---

### **5. Escalabilidade e Resiliência**

1. **Escalabilidade**
   - Adicionar novos dispositivos Raspberry Pi não impacta os serviços na nuvem.
   - Kafka escala horizontalmente com partições.
   - Redis armazena leituras recentes para processamentos rápidos.

2. **Resiliência**
   - Raspberry Pi armazena leituras localmente caso a comunicação falhe.
   - Backend implementa retry para envio ao Kafka.
   - Dados persistidos no PostgreSQL garantem integridade em caso de falhas.

---

### **6. Segurança Adicional**

1. **Autenticação e Autorização**
   - Cada Raspberry Pi recebe uma chave de API ou token JWT exclusivo.
   - O backend valida o token antes de aceitar requisições.

2. **Criptografia**
   - Comunicação Raspberry Pi -> Backend via HTTPS.
   - Dados sensíveis no banco de dados protegidos com criptografia.

3. **Monitoramento de Dispositivos**
   - Health check periódico dos dispositivos para garantir operação contínua.
   - Alertas gerados em caso de inatividade.

---

### **7. Conclusão**

Esta arquitetura incrementada integra perfeitamente os dispositivos Raspberry Pi como sensores remotos, mantendo:
- **Simplicidade nos dispositivos (webhook)**.
- **Robustez e escalabilidade na nuvem (Kafka + Spring Boot)**.
- **Resiliência contra falhas em redes ou dispositivos**.

Essa abordagem é ideal para um MVP escalável que combina IoT e processamento em tempo real.

### **Componentes Principais**

---

#### **1. Coleta e Ingestão de Dados de Sensores**
- **Raspberry Pi com Sensores de Pressão**:
  - Equipado com APIs em **Spring Boot** para gerenciar e enviar leituras periodicamente.
  - Dados são transmitidos via **API REST** para o backend na nuvem em intervalos configuráveis (ex.: a cada 30 segundos).

- **Backend de Ingestão (API REST)**:
  - Desenvolvido em **Spring Boot**, é o ponto de entrada para receber os dados dos sensores.
  - Realiza validações iniciais, como sanitização de payloads.
  - Publica os dados no tópico **Kafka** `pressure-readings` para processamento posterior.

---

#### **2. Apache Kafka como Backbone de Mensageria**
- Gerencia a comunicação assíncrona e escalável entre os serviços.
- Tópicos principais:
  1. **`pressure-readings`**:
     - Centraliza todas as leituras de sensores.
     - Permite o consumo por múltiplos serviços, incluindo detecção de vazamentos e armazenamento.
  2. **`leakage-alerts`**:
     - Contém eventos de vazamentos detectados.
     - É consumido por serviços de notificação e outros sistemas de integração.

---

#### **3. Detecção de Vazamentos**
- Serviço dedicado em **Spring Boot**, implementado com foco em alta performance:
  1. **Consumo de Dados**:
     - Lê eventos do tópico Kafka `pressure-readings`.
     - Utiliza **Redis** para acessar rapidamente o estado mais recente de cada sensor (leituras anteriores).
  2. **Análise e Detecção**:
     - Calcula variações de pressão entre leituras consecutivas.
     - Se a variação exceder ±5 mca, um vazamento é identificado.
  3. **Publicação de Alertas**:
     - Gera eventos no tópico Kafka `leakage-alerts` contendo detalhes do vazamento, como:
       - Localização do sensor.
       - Horário do evento.
       - Valores da pressão anterior e atual.

---

#### **4. Notificação**
- Serviço especializado para enviar alertas em tempo real, consumindo eventos do tópico `leakage-alerts`.
- **Canais de Notificação**:
  1. **E-mail**:
     - Integrado com **Java Mail** para envio confiável e escalável.  

---

#### **5. Persistência de Dados com PostgreSQL**
- Banco de dados relacional utilizado para armazenamento seguro e estruturado:
  1. **Leituras de Pressão**:
     - Todas as leituras são armazenadas para auditoria e análises históricas.
  2. **Eventos de Vazamento**:
     - Registros detalhados dos alertas gerados pelo sistema.
  3. **Histórico de Notificações**:
     - Armazena logs de mensagens enviadas, garantindo rastreabilidade.

---

#### **6. Cache para Processamento Rápido com Redis**
- Redis é usado para:
  1. **Cache de Leituras Recentes**:
     - Armazena as leituras mais recentes de cada sensor para processamento em tempo real pelo serviço de detecção.
     - Reduz consultas ao banco de dados e acelera cálculos de variação.
  2. **Notificações em Tempo Real**:
     - Gerencia temporariamente mensagens e status de envio para garantir entregas rápidas.

---

### **Diferenciais Aprimorados**
- **Segurança e Eficiência**:
  - Comunicação entre Raspberry Pi e backend é segura (HTTPS).
  - Kafka e Redis garantem alta disponibilidade e processamento em tempo real.
- **Resiliência**:
  - Dados são persistidos no PostgreSQL, enquanto Redis oferece suporte para recuperação rápida em caso de falhas.
- **Escalabilidade**:
  - Sensores adicionais podem ser facilmente integrados sem impactar o sistema, graças ao uso de Kafka e serviços desacoplados.