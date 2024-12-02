### Proposta de Solução para Águas Fluentes S/A

---

#### **Projeto Documental**

##### **Objetivo**
Desenvolver um sistema automatizado para:
1. **Medição automatizada** dos hidrômetros.
2. **Processamento ágil** de faturas, reduzindo o tempo atual de 7+5 dias para um processo quase em tempo real.
3. **Identificação e notificação de vazamentos** com base na análise de variação de pressão, permitindo ações preventivas.

##### **Arquitetura Proposta**

1. **Arquitetura Base**
   - **Microservices**: Componentes independentes para medição, processamento e notificações.
   - **Event-driven Architecture**: Baseada em eventos capturados por sensores de pressão e hidrômetros, utilizando **Apache Kafka** para ingestão e disseminação de dados.

2. **Componentes Principais**
   - **Medição Automática**:
     - Serviço para receber leituras periódicas de hidrômetros conectados.
     - Utilização de APIs RESTful para ingestão de dados dos sensores.
   - **Processamento de Faturas**:
     - Serviço para cálculo automatizado com base em leitura recente.
     - Atualização de histórico de consumo em banco de dados.
   - **Detecção de Vazamentos**:
     - Algoritmo baseado em análise da variação de pressão (threshold de 5 mca).
     - Notificação em tempo real via e-mail.
   - **Interface do Usuário**:
     - Portal web para visualização de consumo, faturas e alertas de vazamento.

3. **Infraestrutura**
   - **Backend**: Java com Spring Boot.
   - **Mensageria**: Apache Kafka para ingestão e processamento em tempo real.
   - **Banco de Dados**:
     - Relacional (PostgreSQL) para persistência de leituras, faturas e histórico.
     - Não Relacional (Redis) para cache e notificações rápidas.
   - **Plataforma na Nuvem**: AWS para escalabilidade e alta disponibilidade.

##### **Fluxo de Dados**

1. Leituras de sensores e hidrômetros são capturadas periodicamente.
2. Dados são enviados para o sistema via APIs e processados no serviço de medição.
3. Eventos de variação de pressão são registrados no Kafka e consumidos pelo serviço de detecção de vazamentos.
4. Processamento de faturas é realizado automaticamente no final de cada ciclo, com geração de relatórios.
5. Notificações são enviadas aos consumidores em caso de vazamentos detectados.

---

#### **Divisão de Entregas e Sprints**
| **Sprint** | **Tarefas**                                                                          | **Duração (semanas)**  |
|------------|--------------------------------------------------------------------------------------|------------------------|
| 1          | - Configuração de ambiente (Infraestrutura na nuvem, Kafka, banco de dados)          | 2                      |
|            | - Criação do serviço de ingestão de dados para sensores de pressão e hidrômetros.    |                        |
|------------|--------------------------------------------------------------------------------------|------------------------|
| 2          | - Implementação do serviço de detecção de vazamentos.                                | 2                      |
|            | - Configuração de alertas por e-mail.                                                |                        |
|------------|--------------------------------------------------------------------------------------|------------------------|
| 3          | - Desenvolvimento do serviço de processamento de faturas.                            | 2                      |
|            | - Integração com banco de dados para cálculo e armazenamento do histórico.           |                        |
|------------|--------------------------------------------------------------------------------------|------------------------|
| 4          | - Desenvolvimento da interface do usuário (portal web).                              | 2                      |
|            | - Testes de integração e ajuste de fluxos.                                           |                        |
|------------|--------------------------------------------------------------------------------------|------------------------|
| 5          | - Testes finais de desempenho.                                                       | 1                      |
|            | - Deploy do MVP e entrega.                                                           |                        |

---

#### **MVP**

O MVP irá focar na **detecção e notificação de vazamentos**. Para isso:
1. Configuração de sensores para capturar variação de pressão.
2. Implementação do serviço de detecção utilizando eventos do Kafka.
3. Integração com APIs de notificação por e-mail.

##### **Tecnologias do MVP**
- Java Spring Boot (backend).
- Apache Kafka (mensageria).
- PostgreSQL (banco de dados para persistência).
- Redis (cache para alertas).

---

#### **Estimativa de Esforço**
|----------------------------------------|----------------------|
| **Atividade**                          | **Horas Estimadas**  |
|----------------------------------------|----------------------|
| Configuração inicial                   | 40                   |
| Desenvolvimento do backend             | 120                  |
| Criação da interface web               | 80                   |
| Integração com sistemas externos       | 60                   |
| Testes e ajustes finais                | 40                   |
| **Total Estimado (em horas)**          | **340 horas**        |
|----------------------------------------|----------------------|

Com base no total de horas, a estimativa para orçamento pode ser ajustada considerando valores por hora aplicados no mercado local.

---

#### **Considerações Finais**
Esta solução está alinhada com os objetivos da Águas Fluentes S/A de modernizar processos e aumentar a eficiência operacional. O MVP garante um impacto inicial significativo com a detecção de vazamentos, enquanto as sprints subsequentes completam o escopo do projeto.
