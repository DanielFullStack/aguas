## README.md

# Aplicação de Detecção de Vazamentos

Este repositório contém os serviços necessários para rodar o sistema de detecção de vazamentos. Os serviços utilizam **Docker Compose** para facilitar a configuração e execução. Abaixo estão as instruções para rodar a aplicação localmente.

---

[Desenho Técnico.pdf](https://github.com/user-attachments/files/17972237/Desenho.Tecnico.pdf)
![Desenho Técnico](https://github.com/user-attachments/assets/3b932ae1-da9f-4f05-938f-ed57bc25975b)

## Serviços Principais

1. **`[srv_ingestion](https://github.com/DanielFullStack/srv_ingestion)`**: Responsável por ingerir dados de sensores.
2. **`[srv_leakdetection](https://github.com/DanielFullStack/srv_leakdetection)`**: Processa leituras de pressão e detecta vazamentos.
3. **`[srv_notification](https://github.com/DanielFullStack/srv_notification)`**: Envia notificações sobre vazamentos detectados.
4. **`[srv_sensor](https://github.com/DanielFullStack/srv_sensor)`**: Simula os sensores que geram leituras de pressão.

---

## Pré-requisitos

Certifique-se de ter instalado os seguintes softwares:

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)

---

## Passo a Passo para Rodar a Aplicação

1. **Clone o repositório:**

   ```bash
   git clone https://github.com/seu-usuario/seu-repositorio.git
   cd seu-repositorio
   ```

2. **Inicie os serviços na ordem correta:**

   Cada serviço possui seu próprio diretório com um arquivo `docker-compose.yml`. Siga a ordem abaixo para iniciar os serviços:

   ### Iniciar o `srv_ingestion`

   ```bash
   cd srv_ingestion
   docker-compose up --build -d
   ```

   ### Iniciar o `srv_leakdetection`

   ```bash
   cd srv_leakdetection
   docker-compose up --build -d
   ```

   ### Iniciar o `srv_notification`

   ```bash
   cd srv_notification
   docker-compose up --build -d
   ```

   ### Iniciar o `srv_sensor`

   ```bash
   cd srv_sensor
   docker-compose up --build -d
   ```

3. **Verificar os contêineres em execução:**

   Execute o comando abaixo para verificar se todos os contêineres estão rodando:

   ```bash
   docker ps
   ```

   Você deverá ver os contêineres dos quatro serviços listados.

4. **Logs (opcional):**

   Para visualizar os logs de um serviço específico, use o comando:

   ```bash
   docker-compose logs -f
   ```

---

## Encerrando os Serviços

Para parar todos os serviços, utilize o comando `docker-compose down` em cada diretório:

```bash
cd srv_sensor
docker-compose down
cd ../srv_notification
docker-compose down
cd ../srv_leakdetection
docker-compose down
cd ../srv_ingestion
docker-compose down
```
